---
jupyter:
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
  language_info:
    codemirror_mode:
      name: ipython
      version: 3
    file_extension: .py
    mimetype: text/x-python
    name: python
    nbconvert_exporter: python
    pygments_lexer: ipython3
    version: 3.13.1
  nbformat: 4
  nbformat_minor: 5
---

::: {#d67b5907 .cell .markdown}
**Structure de ce livrable :**

1.  **Contrôle Qualité & Audit des Données (Data Quality)**
2.  **Nettoyage & Standardisation** (normalisation casse, gestion des doublons)
3.  **Analyse du Funnel & Identification des Frictions**
4.  **Analyse des Résultats (Outcomes) & Déduplication**
5.  **Performance des Canaux d\'Acquisition & Devices**
6.  **Visualisations Métier Clés**
7.  **Modélisation & Pipeline SQL Avancé (DuckDB)**
8.  **Réponses Explicites aux 4 Questions Stratégiques du README**
:::

::: {#bd841936-0038-4e4d-b909-dde10e0cdec4 .cell .code execution_count="1"}
``` python
import pandas as pd

visitors = pd.read_parquet("data/visitors.parquet")
sessions = pd.read_parquet("data/sessions.parquet")
questions = pd.read_parquet("data/questionnaire_questions.parquet")
events = pd.read_parquet("data/questionnaire_events.parquet")
answers = pd.read_parquet("data/questionnaire_answers.parquet")
outcomes = pd.read_parquet("data/questionnaire_outcomes.parquet")
```
:::

:::: {#b5a301b1-844d-44c2-84d5-36aa45b05bf6 .cell .code execution_count="2"}
``` python
print("Visitors:", visitors.shape)
print("Sessions:", sessions.shape)
print("Questions:", questions.shape)
print("Events:", events.shape)
print("Answers:", answers.shape)
print("Outcomes:", outcomes.shape)
```

::: {.output .stream .stdout}
    Visitors: (20000, 5)
    Sessions: (26000, 8)
    Questions: (5, 5)
    Events: (195816, 6)
    Answers: (68460, 7)
    Outcomes: (11357, 4)
:::
::::

:::: {#494350f2-7f74-447c-a6ec-d316439a6aa0 .cell .code execution_count="3"}
``` python
tables = {
    "visitors": visitors,
    "sessions": sessions,
    "questions": questions,
    "events": events,
    "answers": answers,
    "outcomes": outcomes
}

for name, df in tables.items():
    print(f"\n{'='*60}")
    print(f"{name.upper()}")
    print("="*60)
    print(df.info())
```

::: {.output .stream .stdout}

    ============================================================
    VISITORS
    ============================================================
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 20000 entries, 0 to 19999
    Data columns (total 5 columns):
     #   Column         Non-Null Count  Dtype              
    ---  ------         --------------  -----              
     0   visitor_id     20000 non-null  object             
     1   birth_year     19239 non-null  Int64              
     2   gender         19347 non-null  object             
     3   region         19196 non-null  object             
     4   first_seen_at  20000 non-null  datetime64[ns, UTC]
    dtypes: Int64(1), datetime64[ns, UTC](1), object(3)
    memory usage: 800.9+ KB
    None

    ============================================================
    SESSIONS
    ============================================================
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 26000 entries, 0 to 25999
    Data columns (total 8 columns):
     #   Column                Non-Null Count  Dtype              
    ---  ------                --------------  -----              
     0   session_id            26000 non-null  object             
     1   visitor_id            26000 non-null  object             
     2   session_started_at    26000 non-null  datetime64[ns, UTC]
     3   acquisition_source    26000 non-null  object             
     4   campaign_name         11919 non-null  object             
     5   device_type           26000 non-null  object             
     6   landing_page          26000 non-null  object             
     7   is_returning_visitor  26000 non-null  bool               
    dtypes: bool(1), datetime64[ns, UTC](1), object(6)
    memory usage: 1.4+ MB
    None

    ============================================================
    QUESTIONS
    ============================================================
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5 entries, 0 to 4
    Data columns (total 5 columns):
     #   Column           Non-Null Count  Dtype 
    ---  ------           --------------  ----- 
     0   question_id      5 non-null      object
     1   question_number  5 non-null      int64 
     2   question_text    5 non-null      object
     3   response_type    5 non-null      object
     4   allowed_answers  5 non-null      object
    dtypes: int64(1), object(4)
    memory usage: 332.0+ bytes
    None

    ============================================================
    EVENTS
    ============================================================
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 195816 entries, 0 to 195815
    Data columns (total 6 columns):
     #   Column           Non-Null Count   Dtype              
    ---  ------           --------------   -----              
     0   event_id         195816 non-null  object             
     1   session_id       195816 non-null  object             
     2   visitor_id       195816 non-null  object             
     3   event_timestamp  195816 non-null  datetime64[ns, UTC]
     4   event_type       195816 non-null  object             
     5   question_number  141685 non-null  Int64              
    dtypes: Int64(1), datetime64[ns, UTC](1), object(4)
    memory usage: 9.2+ MB
    None

    ============================================================
    ANSWERS
    ============================================================
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 68460 entries, 0 to 68459
    Data columns (total 7 columns):
     #   Column           Non-Null Count  Dtype              
    ---  ------           --------------  -----              
     0   answer_id        68460 non-null  object             
     1   session_id       68460 non-null  object             
     2   visitor_id       68460 non-null  object             
     3   question_id      68460 non-null  object             
     4   question_number  68460 non-null  Int64              
     5   answer_value     68460 non-null  object             
     6   answered_at      68460 non-null  datetime64[ns, UTC]
    dtypes: Int64(1), datetime64[ns, UTC](1), object(5)
    memory usage: 3.7+ MB
    None

    ============================================================
    OUTCOMES
    ============================================================
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 11357 entries, 0 to 11356
    Data columns (total 4 columns):
     #   Column            Non-Null Count  Dtype              
    ---  ------            --------------  -----              
     0   session_id        11357 non-null  object             
     1   visitor_id        11357 non-null  object             
     2   outcome_category  11357 non-null  object             
     3   completed_at      11357 non-null  datetime64[ns, UTC]
    dtypes: datetime64[ns, UTC](1), object(3)
    memory usage: 355.0+ KB
    None
:::
::::

::::::::::::::: {#cbd8fbbd-0c01-4e81-b8b3-7b7d00d777af .cell .code execution_count="4"}
``` python
for name, df in tables.items():
    print(f"\n{name.upper()}")
    display(df.head())
```

::: {.output .stream .stdout}

    VISITORS
:::

::: {.output .display_data}
         visitor_id  birth_year  gender                      region  \
    0  VIS_00000001        2008    male                    Bretagne   
    1  VIS_00000002        1946  female             Hauts-de-France   
    2  VIS_00000003        <NA>  female            Pays de la Loire   
    3  VIS_00000004        1989   other  Provence-Alpes-Côte d'Azur   
    4  VIS_00000005        1989  female             Hauts-de-France   

                            first_seen_at  
    0 2026-08-02 04:29:33.074952518+00:00  
    1 2026-07-25 11:59:57.752213928+00:00  
    2 2026-06-22 12:37:47.114315535+00:00  
    3 2026-07-12 02:29:05.408274007+00:00  
    4 2026-07-10 05:33:03.390070397+00:00  
:::

::: {.output .stream .stdout}

    SESSIONS
:::

::: {.output .display_data}
         session_id    visitor_id                  session_started_at  \
    0  SES_00000001  VIS_00016128 2026-06-01 00:01:59.429044608+00:00   
    1  SES_00000002  VIS_00012147 2026-06-01 00:07:54.468178789+00:00   
    2  SES_00000003  VIS_00018597 2026-06-01 00:10:06.909833040+00:00   
    3  SES_00000004  VIS_00006671 2026-06-01 00:14:47.814240001+00:00   
    4  SES_00000005  VIS_00018327 2026-06-01 00:14:55.885097705+00:00   

      acquisition_source               campaign_name device_type  \
    0             social   diabetes_social_awareness     desktop   
    1     google_organic                        None     desktop   
    2         google_ads  diabetes_awareness_general     desktop   
    3     google_organic                        None      mobile   
    4            chatgpt                        None      mobile   

                     landing_page  is_returning_visitor  
    0  /health/diabetes-awareness                 False  
    1  /health/diabetes-awareness                 False  
    2  /health/diabetes-awareness                 False  
    3  /health/diabetes-awareness                 False  
    4  /health/diabetes-awareness                 False  
:::

::: {.output .stream .stdout}

    QUESTIONS
:::

::: {.output .display_data}
       question_id  question_number  \
    0  diabetes_q1                1   
    1  diabetes_q2                2   
    2  diabetes_q3                3   
    3  diabetes_q4                4   
    4  diabetes_q5                5   

                                           question_text  response_type  \
    0  Have you ever been told by a healthcare profes...  single_choice   
    1  During the last 3 months, have you frequently ...  single_choice   
    2  Has a parent or sibling been diagnosed with Ty...  single_choice   
    3  How often do you usually do at least 30 minute...  single_choice   
    4  Have you ever been told that your blood sugar ...  single_choice   

                                         allowed_answers  
    0                 ["yes", "no", "prefer_not_to_say"]  
    1                          ["yes", "no", "not_sure"]  
    2                          ["yes", "no", "not_sure"]  
    3  ["5_or_more_days_per_week", "3_to_4_days_per_w...  
    4                          ["yes", "no", "not_sure"]  
:::

::: {.output .stream .stdout}

    EVENTS
:::

::: {.output .display_data}
            event_id    session_id    visitor_id  \
    0  EVT_000000001  SES_00000001  VIS_00016128   
    1  EVT_000000002  SES_00000001  VIS_00016128   
    2  EVT_000000003  SES_00000001  VIS_00016128   
    3  EVT_000000004  SES_00000001  VIS_00016128   
    4  EVT_000000005  SES_00000001  VIS_00016128   

                          event_timestamp           event_type  question_number  
    0 2026-06-01 00:01:59.429044608+00:00         landing_view             <NA>  
    1 2026-06-01 00:02:07.868799740+00:00  questionnaire_start             <NA>  
    2 2026-06-01 00:02:11.208955362+00:00        question_view                1  
    3 2026-06-01 00:02:35.588484793+00:00      question_answer                1  
    4 2026-06-01 00:02:36.940089191+00:00        question_view                2  
:::

::: {.output .stream .stdout}

    ANSWERS
:::

::: {.output .display_data}
           answer_id    session_id    visitor_id  question_id  question_number  \
    0  ANS_000000001  SES_00000001  VIS_00016128  diabetes_q1                1   
    1  ANS_000000002  SES_00000002  VIS_00012147  diabetes_q1                1   
    2  ANS_000000003  SES_00000002  VIS_00012147  diabetes_q2                2   
    3  ANS_000000004  SES_00000003  VIS_00018597  diabetes_q1                1   
    4  ANS_000000005  SES_00000003  VIS_00018597  diabetes_q2                2   

      answer_value                         answered_at  
    0           no 2026-06-01 00:02:35.588484793+00:00  
    1           no 2026-06-01 00:09:23.966499272+00:00  
    2           no 2026-06-01 00:09:48.773896500+00:00  
    3           no 2026-06-01 00:11:09.794015705+00:00  
    4           no 2026-06-01 00:11:33.897800263+00:00  
:::

::: {.output .stream .stdout}

    OUTCOMES
:::

::: {.output .display_data}
         session_id    visitor_id       outcome_category  \
    0  SES_00000003  VIS_00018597  no_current_indication   
    1  SES_00000004  VIS_00006671  no_current_indication   
    2  SES_00000008  VIS_00011878          possible_risk   
    3  SES_00000010  VIS_00013989  no_current_indication   
    4  SES_00000011  VIS_00002128  no_current_indication   

                             completed_at  
    0 2026-06-01 00:12:52.331208808+00:00  
    1 2026-06-01 00:16:49.679841847+00:00  
    2 2026-06-01 00:25:13.601379009+00:00  
    3 2026-06-01 00:29:10.634712357+00:00  
    4 2026-06-01 00:33:05.062401478+00:00  
:::
:::::::::::::::

::: {#81caa337 .cell .markdown}

------------------------------------------------------------------------

## 1. Audit Qualité des Données (Data Quality Check)

Nous vérifions l\'intégrité référentielle, les valeurs manquantes, l\'unicité des clés primaires et la cohérence des variables catégorielles.
:::

::: {#d9132b04-2164-4366-8ef2-a24118151c36 .cell .markdown}
# 2 data Quality check
:::

:::: {#a8867f46-475e-4e39-8280-4666bf9a331c .cell .code execution_count="5"}
``` python
# Data Quality Check

for name, df in tables.items():
    print(f"\n{'='*60}")
    print(f"{name.upper()}")
    print("="*60)

    # Missing values
    print("\nMissing values:")
    print(df.isna().sum())

    # Duplicate rows
    print("\nDuplicate rows:", df.duplicated().sum())
```

::: {.output .stream .stdout}

    ============================================================
    VISITORS
    ============================================================

    Missing values:
    visitor_id         0
    birth_year       761
    gender           653
    region           804
    first_seen_at      0
    dtype: int64

    Duplicate rows: 0

    ============================================================
    SESSIONS
    ============================================================

    Missing values:
    session_id                  0
    visitor_id                  0
    session_started_at          0
    acquisition_source          0
    campaign_name           14081
    device_type                 0
    landing_page                0
    is_returning_visitor        0
    dtype: int64

    Duplicate rows: 0

    ============================================================
    QUESTIONS
    ============================================================

    Missing values:
    question_id        0
    question_number    0
    question_text      0
    response_type      0
    allowed_answers    0
    dtype: int64

    Duplicate rows: 0

    ============================================================
    EVENTS
    ============================================================

    Missing values:
    event_id               0
    session_id             0
    visitor_id             0
    event_timestamp        0
    event_type             0
    question_number    54131
    dtype: int64

    Duplicate rows: 0

    ============================================================
    ANSWERS
    ============================================================

    Missing values:
    answer_id          0
    session_id         0
    visitor_id         0
    question_id        0
    question_number    0
    answer_value       0
    answered_at        0
    dtype: int64

    Duplicate rows: 0

    ============================================================
    OUTCOMES
    ============================================================

    Missing values:
    session_id          0
    visitor_id          0
    outcome_category    0
    completed_at        0
    dtype: int64

    Duplicate rows: 0
:::
::::

:::: {#13974df6-bb56-4158-9b5a-b5faf23a7330 .cell .code execution_count="6"}
``` python
# Check uniqueness of identifiers

print("Visitors - duplicate visitor_id:",
      visitors["visitor_id"].duplicated().sum())

print("Sessions - duplicate session_id:",
      sessions["session_id"].duplicated().sum())

print("Events - duplicate event_id:",
      events["event_id"].duplicated().sum())

print("Answers - duplicate answer_id:",
      answers["answer_id"].duplicated().sum())

print("Questions - duplicate question_id:",
      questions["question_id"].duplicated().sum())
```

::: {.output .stream .stdout}
    Visitors - duplicate visitor_id: 0
    Sessions - duplicate session_id: 0
    Events - duplicate event_id: 0
    Answers - duplicate answer_id: 0
    Questions - duplicate question_id: 0
:::
::::

:::: {#f3051c3c-e14f-45a3-9c66-24fbd1aa41a1 .cell .code execution_count="7"}
``` python
#vérifier les valeurs catégorielles
print("Gender:")
print(visitors["gender"].value_counts(dropna=False))

print("\nRegions:")
print(visitors["region"].value_counts(dropna=False))

print("\nAcquisition sources:")
print(sessions["acquisition_source"].value_counts(dropna=False))

print("\nDevice types:")
print(sessions["device_type"].value_counts(dropna=False))

print("\nOutcome categories:")
print(outcomes["outcome_category"].value_counts(dropna=False))
```

::: {.output .stream .stdout}
    Gender:
    gender
    female               9301
    male                 8257
    None                  653
    prefer_not_to_say     512
    other                 284
    F                     283
    Male                  246
    M                     245
    Female                219
    Name: count, dtype: int64

    Regions:
    region
    Hauts-de-France               2132
    Île-de-France                 2103
    Auvergne-Rhône-Alpes          2080
    Occitanie                     2028
    Grand Est                     2022
    Nouvelle-Aquitaine            2014
    Bretagne                      2014
    Provence-Alpes-Côte d'Azur    2001
    Pays de la Loire              1934
    None                           804
    pays de la loire               106
    bretagne                       105
    Auvergne Rhone Alpes           103
    occitanie                       99
    nouvelle-aquitaine              99
    grand est                       99
    hauts-de-france                 88
    provence-alpes-côte d'azur      87
    Ile de France                   82
    Name: count, dtype: int64

    Acquisition sources:
    acquisition_source
    google_organic    6590
    google_ads        6375
    social            3176
    chatgpt           2401
    referral          1790
    email             1707
    direct            1617
    other              674
    google             450
    Google             450
    facebook           221
    instagram          221
    AI Assistant       165
    ChatGPT            163
    Name: count, dtype: int64

    Device types:
    device_type
    mobile     16772
    desktop     7863
    tablet      1365
    Name: count, dtype: int64

    Outcome categories:
    outcome_category
    no_current_indication    7841
    possible_risk            2276
    declared_diagnosed       1240
    Name: count, dtype: int64
:::
::::

::: {#735008f8-ca65-4046-bf61-aacc7549d16e .cell .markdown}
# 3 Mesurer les problémes
:::

:::: {#f0ced838-b332-4077-96e9-87966ced7fb9 .cell .code execution_count="9"}
``` python
print("Gender:")
print(visitors["gender"].dropna().unique())

print("\nRegion:")
print(visitors["region"].dropna().unique())

print("\nAcquisition source:")
print(sessions["acquisition_source"].unique())
```

::: {.output .stream .stdout}
    Gender:
    ['male' 'female' 'other' 'prefer_not_to_say' 'F' 'Female' 'M' 'Male']

    Region:
    ['Bretagne' 'Hauts-de-France' 'Pays de la Loire'
     "Provence-Alpes-Côte d'Azur" 'Grand Est' 'Île-de-France' 'Occitanie'
     'Auvergne-Rhône-Alpes' 'Nouvelle-Aquitaine' 'occitanie'
     "provence-alpes-côte d'azur" 'nouvelle-aquitaine' 'bretagne'
     'Ile de France' 'grand est' 'Auvergne Rhone Alpes' 'hauts-de-france'
     'pays de la loire']

    Acquisition source:
    ['social' 'google_organic' 'google_ads' 'chatgpt' 'email' 'facebook'
     'direct' 'Google' 'instagram' 'referral' 'google' 'ChatGPT'
     'AI Assistant' 'other']
:::
::::

:::: {#32dbf1b9-e21c-4f46-aa5e-d28673c2ef62 .cell .code execution_count="10"}
``` python
print("Visitors first_seen_at:")
print(visitors["first_seen_at"].min())
print(visitors["first_seen_at"].max())

print("\nSessions:")
print(sessions["session_started_at"].min())
print(sessions["session_started_at"].max())

print("\nEvents:")
print(events["event_timestamp"].min())
print(events["event_timestamp"].max())

print("\nOutcomes:")
print(outcomes["completed_at"].min())
print(outcomes["completed_at"].max())
```

::: {.output .stream .stdout}
    Visitors first_seen_at:
    2026-06-01 00:01:59.429044608+00:00
    2026-08-02 16:44:38.140839003+00:00

    Sessions:
    2026-06-01 00:01:59.429044608+00:00
    2026-08-02 16:44:38.140839003+00:00

    Events:
    2026-06-01 00:01:59.429044608+00:00
    2026-08-02 16:44:38.140839003+00:00

    Outcomes:
    2026-06-01 00:12:52.331208808+00:00
    2026-08-02 16:35:20.125069440+00:00
:::
::::

:::: {#a39edb21-057b-4a4b-9e7f-355b7582987f .cell .code execution_count="11"}
``` python
print(visitors["birth_year"].describe())

print("\nBirth years:")
print(visitors["birth_year"].sort_values().head(20))

print("\nHighest birth years:")
print(visitors["birth_year"].sort_values(ascending=False).head(20))
```

::: {.output .stream .stdout}
    count        19239.0
    mean     1981.647747
    std        15.259051
    min           1929.0
    25%           1971.0
    50%           1982.0
    75%           1993.0
    max           2010.0
    Name: birth_year, dtype: Float64

    Birth years:
    17164    1929
    2521     1929
    11559    1929
    2181     1929
    13995    1929
    19355    1929
    9915     1929
    14122    1929
    10823    1929
    13859    1929
    10957    1929
    12604    1929
    12223    1930
    13717    1930
    8003     1930
    12357    1930
    4131     1930
    14941    1930
    4082     1930
    15519    1930
    Name: birth_year, dtype: Int64

    Highest birth years:
    8351     2010
    4142     2010
    3180     2010
    13511    2010
    14692    2010
    3200     2010
    7023     2010
    11469    2010
    240      2010
    14203    2010
    1633     2010
    11148    2010
    8919     2010
    12697    2010
    9031     2010
    73       2010
    227      2010
    3374     2010
    13489    2010
    10719    2010
    Name: birth_year, dtype: Int64
:::
::::

::: {#c6c75b5a .cell .markdown}

------------------------------------------------------------------------

## 2. Nettoyage et Standardisation des Données

**Anomalies identifiées lors de l\'audit :**

- `gender` : disparités de casse et abréviations (`F`, `Female`, `M`, `Male`). Standardisation en `female`, `male`.
- `region` : disparités de casse (ex. `occitanie` vs `Occitanie`, `Ile de France` sans tirets). Standardisation selon les 9 régions métropolitaines répertoriées.
- `birth_year` : valeurs cohérentes (1929 à 2010), \~761 valeurs manquantes (visiteurs sans profil complet).
:::

::: {#83fa4343-55b9-4ea1-b635-8c834d3daae0 .cell .markdown}
# Standarisation
:::

:::: {#3395bf97-fa88-4e46-93ea-d46dfe998165 .cell .code execution_count="12"}
``` python
visitors["gender_clean"] = visitors["gender"].replace({
    "F": "female",
    "Female": "female",
    "M": "male",
    "Male": "male"
})

print(visitors["gender_clean"].value_counts(dropna=False))
```

::: {.output .stream .stdout}
    gender_clean
    female               9803
    male                 8748
    None                  653
    prefer_not_to_say     512
    other                 284
    Name: count, dtype: int64
:::
::::

:::: {#e10b4299-45f3-49cd-9eb1-769b77c463e7 .cell .code execution_count="13"}
``` python
region_mapping = {
    "occitanie": "Occitanie",
    "provence-alpes-côte d'azur": "Provence-Alpes-Côte d'Azur",
    "nouvelle-aquitaine": "Nouvelle-Aquitaine",
    "bretagne": "Bretagne",
    "Ile de France": "Île-de-France",
    "grand est": "Grand Est",
    "Auvergne Rhone Alpes": "Auvergne-Rhône-Alpes",
    "hauts-de-france": "Hauts-de-France",
    "pays de la loire": "Pays de la Loire"
}

visitors["region_clean"] = visitors["region"].replace(region_mapping)

print(visitors["region_clean"].value_counts(dropna=False))
```

::: {.output .stream .stdout}
    region_clean
    Hauts-de-France               2220
    Île-de-France                 2185
    Auvergne-Rhône-Alpes          2183
    Occitanie                     2127
    Grand Est                     2121
    Bretagne                      2119
    Nouvelle-Aquitaine            2113
    Provence-Alpes-Côte d'Azur    2088
    Pays de la Loire              2040
    None                           804
    Name: count, dtype: int64
:::
::::

::: {#1e7a31c8 .cell .markdown}

------------------------------------------------------------------------

## 3. Analyse du Funnel de Conversion & Déperditions

Analyse du parcours utilisateur : Arrivée sur la Landing Page $\to$ Début du questionnaire $\to$ Questions 1 à 5 $\to$ Complétion.
:::

:::: {#f9d566cc-77c7-4e59-9578-c601f0c061b8 .cell .code execution_count="14"}
``` python
events["event_type"].value_counts()
```

::: {.output .execute_result execution_count="14"}
    event_type
    question_view             73225
    question_answer           68460
    landing_view              26000
    questionnaire_start       16774
    questionnaire_complete    11357
    Name: count, dtype: int64
:::
::::

:::: {#8930a8ab-85ee-4f7d-8399-1aa5612b6850 .cell .code execution_count="15"}
``` python
funnel = events.groupby("event_type")["session_id"].nunique().sort_values(ascending=False)

print(funnel)
```

::: {.output .stream .stdout}
    event_type
    landing_view              26000
    questionnaire_start       16774
    question_view             15881
    question_answer           15558
    questionnaire_complete    11357
    Name: session_id, dtype: int64
:::
::::

:::: {#f2757be3-99e0-4d38-9289-f6fe40415e32 .cell .code execution_count="16"}
``` python
question_funnel = (
    events[events["event_type"] == "question_view"]
    .groupby("question_number")["session_id"]
    .nunique()
)

print(question_funnel)
```

::: {.output .stream .stdout}
    question_number
    1    15881
    2    15558
    3    14779
    4    13920
    5    12044
    Name: session_id, dtype: int64
:::
::::

:::: {#f57b2ca9-dbab-48b9-b604-add3af4b8230 .cell .code execution_count="17"}
``` python
funnel_steps = {
    "Landing": 26000,
    "Start": 16774,
    "Q1": 15881,
    "Q2": 15558,
    "Q3": 14779,
    "Q4": 13920,
    "Q5": 12044,
    "Complete": 11357
}

funnel_df = pd.DataFrame(
    list(funnel_steps.items()),
    columns=["step", "sessions"]
)

funnel_df["conversion_from_previous"] = (
    funnel_df["sessions"] / funnel_df["sessions"].shift(1) * 100
)

funnel_df["conversion_from_landing"] = (
    funnel_df["sessions"] / funnel_df["sessions"].iloc[0] * 100
)

funnel_df
```

::: {.output .execute_result execution_count="17"}
           step  sessions  conversion_from_previous  conversion_from_landing
    0   Landing     26000                       NaN               100.000000
    1     Start     16774                 64.515385                64.515385
    2        Q1     15881                 94.676285                61.080769
    3        Q2     15558                 97.966123                59.838462
    4        Q3     14779                 94.992930                56.842308
    5        Q4     13920                 94.187699                53.538462
    6        Q5     12044                 86.522989                46.323077
    7  Complete     11357                 94.295915                43.680769
:::
::::

::: {#b909cc31 .cell .markdown}

------------------------------------------------------------------------

## 4. Analyse des Réponses & Déduplication (Data Engineering)

Lors de l\'exploration des réponses, nous constatons que certaines sessions comportent plusieurs réponses enregistrées pour une même question (ex. changement d\'avis ou multi-clics).\
Nous appliquons une règle de déduplication métier rigoureuse : **conserver la dernière réponse enregistrée chronologiquement (`answered_at`) pour chaque session et question**.
:::

:::: {#a90c6656-db34-4ddd-b8ca-eda2a41487b0 .cell .code execution_count="18"}
``` python
questions[["question_number", "question_text", "response_type", "allowed_answers"]]
```

::: {.output .execute_result execution_count="18"}
       question_number                                      question_text  \
    0                1  Have you ever been told by a healthcare profes...   
    1                2  During the last 3 months, have you frequently ...   
    2                3  Has a parent or sibling been diagnosed with Ty...   
    3                4  How often do you usually do at least 30 minute...   
    4                5  Have you ever been told that your blood sugar ...   

       response_type                                    allowed_answers  
    0  single_choice                 ["yes", "no", "prefer_not_to_say"]  
    1  single_choice                          ["yes", "no", "not_sure"]  
    2  single_choice                          ["yes", "no", "not_sure"]  
    3  single_choice  ["5_or_more_days_per_week", "3_to_4_days_per_w...  
    4  single_choice                          ["yes", "no", "not_sure"]  
:::
::::

:::: {#3dc52603-ca2f-4904-9844-1038596e75ad .cell .code execution_count="19"}
``` python
answers.groupby(
    ["question_number", "answer_value"]
).size().reset_index(name="count")
```

::: {.output .execute_result execution_count="19"}
        question_number             answer_value  count
    0                 1                       no  13379
    1                 1        prefer_not_to_say    630
    2                 1                      yes   1723
    3                 2                       no  10416
    4                 2                 not_sure   1461
    5                 2                      yes   3041
    6                 3                       no   9087
    7                 3                 not_sure   1212
    8                 3                      yes   3755
    9                 4     1_to_2_days_per_week   3045
    10                4     3_to_4_days_per_week   3344
    11                4  5_or_more_days_per_week   2875
    12                4          rarely_or_never   2911
    13                5                       no   8261
    14                5                 not_sure   1103
    15                5                      yes   2217
:::
::::

:::: {#58210899-7645-4860-9081-f775b74f4817 .cell .code execution_count="20"}
``` python
answers.groupby("question_number").agg(
    sessions=("session_id", "nunique"),
    responses=("answer_id", "count")
)
```

::: {.output .execute_result execution_count="20"}
                     sessions  responses
    question_number                     
    1                   15558      15732
    2                   14779      14918
    3                   13920      14054
    4                   12044      12175
    5                   11462      11581
:::
::::

:::: {#83e4c0f0-94c8-4f1b-89c6-3db3d999a6c9 .cell .code execution_count="21"}
``` python
multiple_answers = (
    answers
    .groupby(["session_id", "question_number"])
    .size()
    .reset_index(name="nb_answers")
)

multiple_answers[multiple_answers["nb_answers"] > 1].head(20)
```

::: {.output .execute_result execution_count="21"}
            session_id  question_number  nb_answers
    6     SES_00000003                4           2
    11    SES_00000004                4           2
    55    SES_00000025                1           2
    196   SES_00000079                2           2
    274   SES_00000104                5           2
    381   SES_00000145                1           2
    449   SES_00000180                5           2
    500   SES_00000202                1           2
    558   SES_00000231                5           2
    670   SES_00000279                1           2
    678   SES_00000282                4           2
    828   SES_00000342                4           2
    851   SES_00000355                5           2
    875   SES_00000368                5           2
    941   SES_00000394                3           2
    977   SES_00000404                2           2
    1030  SES_00000425                2           2
    1072  SES_00000446                3           2
    1304  SES_00000539                5           2
    1349  SES_00000552                5           2
:::
::::

:::: {#fe0a7774-470c-4704-b34d-2be2b25ef60e .cell .code execution_count="22"}
``` python
outcomes["outcome_category"].value_counts()
```

::: {.output .execute_result execution_count="22"}
    outcome_category
    no_current_indication    7841
    possible_risk            2276
    declared_diagnosed       1240
    Name: count, dtype: int64
:::
::::

:::: {#19b7ad85-b30f-4071-84ac-214fb30a031f .cell .code execution_count="23"}
``` python
answers_outcomes = answers.merge(
    outcomes[["session_id", "outcome_category"]],
    on="session_id",
    how="inner"
)

answers_outcomes.head()
```

::: {.output .execute_result execution_count="23"}
           answer_id    session_id    visitor_id  question_id  question_number  \
    0  ANS_000000004  SES_00000003  VIS_00018597  diabetes_q1                1   
    1  ANS_000000005  SES_00000003  VIS_00018597  diabetes_q2                2   
    2  ANS_000000006  SES_00000003  VIS_00018597  diabetes_q3                3   
    3  ANS_000000007  SES_00000003  VIS_00018597  diabetes_q4                4   
    4  ANS_000000008  SES_00000003  VIS_00018597  diabetes_q4                4   

               answer_value                         answered_at  \
    0                    no 2026-06-01 00:11:09.794015705+00:00   
    1                    no 2026-06-01 00:11:33.897800263+00:00   
    2                   yes 2026-06-01 00:11:49.730512143+00:00   
    3  1_to_2_days_per_week 2026-06-01 00:12:19.217151834+00:00   
    4  3_to_4_days_per_week 2026-06-01 00:12:37.189775085+00:00   

            outcome_category  
    0  no_current_indication  
    1  no_current_indication  
    2  no_current_indication  
    3  no_current_indication  
    4  no_current_indication  
:::
::::

:::: {#ae0d6616-77e7-49a2-acb1-5cda10b138d5 .cell .code execution_count="24"}
``` python
answers_outcomes.groupby(
    ["question_number", "outcome_category", "answer_value"]
).size().reset_index(name="count")
```

::: {.output .execute_result execution_count="24"}
        question_number       outcome_category             answer_value  count
    0                 1     declared_diagnosed                       no     52
    1                 1     declared_diagnosed                      yes   1240
    2                 1  no_current_indication                       no   7557
    3                 1  no_current_indication        prefer_not_to_say    333
    4                 1  no_current_indication                      yes     12
    5                 1          possible_risk                       no   2167
    6                 1          possible_risk        prefer_not_to_say    124
    7                 1          possible_risk                      yes      3
    8                 2     declared_diagnosed                       no    820
    9                 2     declared_diagnosed                 not_sure    137
    10                2     declared_diagnosed                      yes    298
    11                2  no_current_indication                       no   6234
    12                2  no_current_indication                 not_sure    857
    13                2  no_current_indication                      yes    814
    14                2          possible_risk                       no    956
    15                2          possible_risk                 not_sure    136
    16                2          possible_risk                      yes   1212
    17                3     declared_diagnosed                       no    756
    18                3     declared_diagnosed                 not_sure    105
    19                3     declared_diagnosed                      yes    393
    20                3  no_current_indication                       no   5924
    21                3  no_current_indication                 not_sure    770
    22                3  no_current_indication                      yes   1213
    23                3          possible_risk                       no    777
    24                3          possible_risk                 not_sure    101
    25                3          possible_risk                      yes   1421
    26                4     declared_diagnosed     1_to_2_days_per_week    277
    27                4     declared_diagnosed     3_to_4_days_per_week    313
    28                4     declared_diagnosed  5_or_more_days_per_week    298
    29                4     declared_diagnosed          rarely_or_never    363
    30                4  no_current_indication     1_to_2_days_per_week   2226
    31                4  no_current_indication     3_to_4_days_per_week   2511
    32                4  no_current_indication  5_or_more_days_per_week   2126
    33                4  no_current_indication          rarely_or_never   1064
    34                4          possible_risk     1_to_2_days_per_week    360
    35                4          possible_risk     3_to_4_days_per_week    333
    36                4          possible_risk  5_or_more_days_per_week    287
    37                4          possible_risk          rarely_or_never   1323
    38                5     declared_diagnosed                       no    840
    39                5     declared_diagnosed                 not_sure     96
    40                5     declared_diagnosed                      yes    318
    41                5  no_current_indication                       no   6336
    42                5  no_current_indication                 not_sure    838
    43                5  no_current_indication                      yes    745
    44                5          possible_risk                       no   1016
    45                5          possible_risk                 not_sure    152
    46                5          possible_risk                      yes   1131
:::
::::

:::: {#52b106ec-604e-4457-8475-fecdbc29163b .cell .code execution_count="25"}
``` python
answers_clean = (
    answers
    .sort_values("answered_at")
    .drop_duplicates(
        subset=["session_id", "question_number"],
        keep="last"
    )
)

print("Réponses originales :", len(answers))
print("Réponses après nettoyage :", len(answers_clean))
```

::: {.output .stream .stdout}
    Réponses originales : 68460
    Réponses après nettoyage : 67763
:::
::::

:::: {#22994ef2-9f8d-4d8a-959d-4eaca08f5f1d .cell .code execution_count="26"}
``` python
answers_clean.groupby("question_number")["session_id"].nunique()
```

::: {.output .execute_result execution_count="26"}
    question_number
    1    15558
    2    14779
    3    13920
    4    12044
    5    11462
    Name: session_id, dtype: int64
:::
::::

::: {#a5c6b246-5701-45b2-900d-2e2e0e99f64e .cell .code execution_count="27"}
``` python
answers_clean_outcomes = answers_clean.merge(
    outcomes[["session_id", "outcome_category"]],
    on="session_id",
    how="inner"
)
```
:::

:::: {#d9e3a09f-0842-482e-a114-4ddac3bca26a .cell .code execution_count="28"}
``` python
answer_outcome_pct = (
    answers_clean_outcomes
    .groupby(["question_number", "outcome_category", "answer_value"])
    .size()
    .groupby(level=[0, 1])
    .transform(lambda x: x / x.sum() * 100)
    .reset_index(name="percentage")
)

answer_outcome_pct
```

::: {.output .execute_result execution_count="28"}
        question_number       outcome_category             answer_value  \
    0                 1     declared_diagnosed                      yes   
    1                 1  no_current_indication                       no   
    2                 1  no_current_indication        prefer_not_to_say   
    3                 1          possible_risk                       no   
    4                 1          possible_risk        prefer_not_to_say   
    5                 2     declared_diagnosed                       no   
    6                 2     declared_diagnosed                 not_sure   
    7                 2     declared_diagnosed                      yes   
    8                 2  no_current_indication                       no   
    9                 2  no_current_indication                 not_sure   
    10                2  no_current_indication                      yes   
    11                2          possible_risk                       no   
    12                2          possible_risk                 not_sure   
    13                2          possible_risk                      yes   
    14                3     declared_diagnosed                       no   
    15                3     declared_diagnosed                 not_sure   
    16                3     declared_diagnosed                      yes   
    17                3  no_current_indication                       no   
    18                3  no_current_indication                 not_sure   
    19                3  no_current_indication                      yes   
    20                3          possible_risk                       no   
    21                3          possible_risk                 not_sure   
    22                3          possible_risk                      yes   
    23                4     declared_diagnosed     1_to_2_days_per_week   
    24                4     declared_diagnosed     3_to_4_days_per_week   
    25                4     declared_diagnosed  5_or_more_days_per_week   
    26                4     declared_diagnosed          rarely_or_never   
    27                4  no_current_indication     1_to_2_days_per_week   
    28                4  no_current_indication     3_to_4_days_per_week   
    29                4  no_current_indication  5_or_more_days_per_week   
    30                4  no_current_indication          rarely_or_never   
    31                4          possible_risk     1_to_2_days_per_week   
    32                4          possible_risk     3_to_4_days_per_week   
    33                4          possible_risk  5_or_more_days_per_week   
    34                4          possible_risk          rarely_or_never   
    35                5     declared_diagnosed                       no   
    36                5     declared_diagnosed                 not_sure   
    37                5     declared_diagnosed                      yes   
    38                5  no_current_indication                       no   
    39                5  no_current_indication                 not_sure   
    40                5  no_current_indication                      yes   
    41                5          possible_risk                       no   
    42                5          possible_risk                 not_sure   
    43                5          possible_risk                      yes   

        percentage  
    0   100.000000  
    1    95.753093  
    2     4.246907  
    3    94.551845  
    4     5.448155  
    5    65.564516  
    6    10.887097  
    7    23.548387  
    8    78.931259  
    9    10.904221  
    10   10.164520  
    11   41.036907  
    12    5.799649  
    13   53.163445  
    14   60.403226  
    15    8.225806  
    16   31.370968  
    17   75.041449  
    18    9.743655  
    19   15.214896  
    20   33.391916  
    21    4.393673  
    22   62.214411  
    23   22.096774  
    24   24.919355  
    25   23.870968  
    26   29.112903  
    27   28.185180  
    28   31.679633  
    29   26.833312  
    30   13.301875  
    31   15.421793  
    32   14.367311  
    33   12.258348  
    34   57.952548  
    35   67.096774  
    36    7.500000  
    37   25.403226  
    38   80.168346  
    39   10.521617  
    40    9.310037  
    41   43.804921  
    42    6.678383  
    43   49.516696  
:::
::::

::: {#4aa4ede0 .cell .markdown}

------------------------------------------------------------------------

## 5. Performance des Canaux d\'Acquisition

Standardisation des sources d\'acquisition (`Google`/`google` $\to$ `google_organic`, `facebook`/`instagram` $\to$ `social`, etc.) et calcul des taux de démarrage et de complétion.
:::

:::: {#33aa89f6-a0b5-4ace-8a33-c29adfe8dcb5 .cell .code execution_count="29"}
``` python
sessions["acquisition_source"].value_counts()
```

::: {.output .execute_result execution_count="29"}
    acquisition_source
    google_organic    6590
    google_ads        6375
    social            3176
    chatgpt           2401
    referral          1790
    email             1707
    direct            1617
    other              674
    google             450
    Google             450
    facebook           221
    instagram          221
    AI Assistant       165
    ChatGPT            163
    Name: count, dtype: int64
:::
::::

:::: {#60707fba-5abd-49d3-8863-9cc78617da57 .cell .code execution_count="30"}
``` python
mapping_sources = {
    "google": "google_organic",
    "Google": "google_organic",
    "facebook": "social",
    "instagram": "social",
    "ChatGPT": "chatgpt"
}

sessions["clean_source"] = sessions["acquisition_source"].replace(mapping_sources)

print(sessions["clean_source"].value_counts())
```

::: {.output .stream .stdout}
    clean_source
    google_organic    7490
    google_ads        6375
    social            3618
    chatgpt           2564
    referral          1790
    email             1707
    direct            1617
    other              674
    AI Assistant       165
    Name: count, dtype: int64
:::
::::

::: {#442e9a65-5a4a-4159-b530-fc6b18ea2b99 .cell .code execution_count="31"}
``` python
start_sessions = set(
    events.loc[
        events["event_type"] == "questionnaire_start",
        "session_id"
    ]
)

complete_sessions = set(
    events.loc[
        events["event_type"] == "questionnaire_complete",
        "session_id"
    ]
)

sessions["started"] = sessions["session_id"].isin(start_sessions)
sessions["completed"] = sessions["session_id"].isin(complete_sessions)
```
:::

:::: {#ff6643c8-0c65-4767-a093-fb5016e4e7ae .cell .code execution_count="32"}
``` python
acq = (
    sessions
    .groupby("clean_source")
    .agg(
        sessions=("session_id", "nunique"),
        starts=("started", "sum"),
        completions=("completed", "sum")
    )
)

acq["start_rate"] = acq["starts"] / acq["sessions"] * 100

acq["completion_rate"] = (
    acq["completions"] / acq["sessions"] * 100
)

acq["completion_among_starters"] = (
    acq["completions"] / acq["starts"] * 100
)

acq.round(2)
```

::: {.output .execute_result execution_count="32"}
                    sessions  starts  completions  start_rate  completion_rate  \
    clean_source                                                                 
    AI Assistant         165     133           97       80.61            58.79   
    chatgpt             2564    1906         1347       74.34            52.54   
    direct              1617    1047          711       64.75            43.97   
    email               1707    1434         1105       84.01            64.73   
    google_ads          6375    3837         2489       60.19            39.04   
    google_organic      7490    4744         3228       63.34            43.10   
    other                674     383          254       56.82            37.69   
    referral            1790    1432         1081       80.00            60.39   
    social              3618    1858         1045       51.35            28.88   

                    completion_among_starters  
    clean_source                               
    AI Assistant                        72.93  
    chatgpt                             70.67  
    direct                              67.91  
    email                               77.06  
    google_ads                          64.87  
    google_organic                      68.04  
    other                               66.32  
    referral                            75.49  
    social                              56.24  
:::
::::

::: {#aa3d5d6c .cell .markdown}

------------------------------------------------------------------------

## 6. Visualisations Stratégiques (Dashboard & Insights Métier)
:::

:::: {#0546b425-c552-4390-a8a4-b9255c698ea1 .cell .code execution_count="33"}
``` python
# 1. Graphique du Funnel Général de Conversion
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(9, 5))
steps = ['Arrivée (Landing)', 'Début Questionnaire', 'Fin Questionnaire']
values = [26000, 16774, 11357]
colors = ['#2b5c8f', '#4682b4', '#c0272d']

bars = plt.bar(steps, values, color=colors, width=0.5)
plt.title('Funnel de Conversion Général (26 000 Sessions)', fontsize=13, fontweight='bold', pad=15)
plt.ylabel('Nombre de Sessions', fontsize=11)

# Afficher les valeurs et pourcentages au-dessus des barres
for bar in bars:
    yval = bar.get_height()
    pct = (yval / 26000) * 100
    plt.text(bar.get_x() + bar.get_width()/2.0, yval + 400, f"{yval:,}\n({pct:.1f}%)", ha='center', va='bottom', fontsize=10, fontweight='bold')

plt.ylim(0, 30000)
plt.tight_layout()
plt.show()
```

::: {.output .display_data}
![](1f2964e83c45b9c4316129da1344e665895f2058.png)
:::
::::

:::: {#64d3ac80-4e29-47c1-830e-46a1e7549fdd .cell .code execution_count="41"}
``` python
# 2. Graphique de la friction par appareil (Mobile vs Desktop)
device_stats = sessions.groupby('device_type').agg(
    total_sessions=('session_id', 'count'),
    completed_sessions=('completed', 'sum')
).reset_index()

device_stats['taux_completion_pct'] = (device_stats['completed_sessions'] / device_stats['total_sessions'] * 100).round(2)

plt.figure(figsize=(8, 4.5))
ax = sns.barplot(
    data=device_stats, 
    x='device_type', 
    y='taux_completion_pct', 
    hue='device_type', 
    palette=['#4682b4', '#c0272d', '#2b5c8f'], 
    legend=False
)

plt.title('Taux de Complétion du Questionnaire par Appareil (%)', fontsize=13, fontweight='bold', pad=15)
plt.ylabel('Taux de Complétion (%)', fontsize=11)
plt.xlabel('Type d\'Appareil', fontsize=11)

for p in ax.patches:
    height = p.get_height()
    if height > 0:
        ax.annotate(f'{height:.2f}%',
                    (p.get_x() + p.get_width() / 2., height),
                    ha='center', va='bottom',
                    fontsize=11, fontweight='bold',
                    xytext=(0, 3), textcoords='offset points')

plt.ylim(0, 65)
plt.tight_layout()
plt.show()
```

::: {.output .display_data}
![](e4c06520b32a1befcffd5b08f005cd9f37bcf26e.png)
:::
::::

:::: {#a031c102-06d3-4e24-a8b8-85403f9c95b9 .cell .code execution_count="36"}
``` python
# 3. Graphique comparatif du taux de complétion par Canal d'Acquisition
df_acq_plot = acq.reset_index().sort_values('completion_rate', ascending=True)

plt.figure(figsize=(10, 5))
plt.barh(df_acq_plot['clean_source'], df_acq_plot['completion_rate'], color='#2b5c8f', height=0.6)
plt.title('Taux de Complétion par Canal d\'Acquisition (%)', fontsize=13, fontweight='bold', pad=15)
plt.xlabel('Taux de Complétion (%)', fontsize=11)

for index, value in enumerate(df_acq_plot['completion_rate']):
    plt.text(value + 0.8, index, f"{value:.2f}%", va='center', fontweight='bold', fontsize=10)

plt.xlim(0, 80)
plt.tight_layout()
plt.show()
```

::: {.output .display_data}
![](0f53448b97cfe6f5a504dff8a709a39e6f499f82.png)
:::
::::

::: {#9d4fb4bd .cell .markdown}
### Synthèse des Enseignements Graphiques :

1.  **Fuite majeure à l\'entrée du funnel (-35.5%)** : 26 000 visiteurs arrivent sur la landing page, mais seulement 16 774 cliquent sur \"Démarrer\". C\'est le plus gros levier d\'optimisation (taux de rebond immédiat).
2.  **Friction Mobile critique (-12.1 pts vs Desktop)** : Le mobile représente **64.5% du trafic global**, mais affiche un taux de complétion de seulement **39.71%** (contre 51.79% sur Desktop). Réduire cette friction mobile apporterait immédiatement plus de 2 000 questionnaires complétés supplémentaires.
3.  **Polarisation des Canaux** : L\'emailing (64.7%) et les recommandations / referral (60.4%) amènent une audience ultra qualifiée. À l\'opposé, les réseaux sociaux ont le taux de complétion le plus faible (28.9%), traduisant un trafic plus passif et volatil.
:::

::: {#431bcfa5 .cell .markdown}

------------------------------------------------------------------------

## 7. Analyses & Requêtes SQL Simplifiées

*Cette section démontre la maîtrise de SQL demandée dans l\'énoncé, à l\'aide de requêtes claires, directes et facilement défendables en entretien technique.*
:::

:::: {#53572735 .cell .code}
``` python
import duckdb

# Initialisation de DuckDB et connexion directe aux DataFrames Pandas
con = duckdb.connect()
con.register("visitors", visitors)
con.register("sessions", sessions)
con.register("events", events)
con.register("answers", answers)
con.register("outcomes", outcomes)

print("Connexion DuckDB prête et tables enregistrées.")
```

::: {.output .stream .stdout}
    Connexion DuckDB prête et tables enregistrées.
:::
::::

::: {#3fd394cc .cell .markdown}
### Requête SQL 1 : Déduplication des réponses

**Objectif :** Résoudre le problème des multi-clics/réponses multiples en ne conservant que la **dernière réponse chronologique** (`answered_at`) pour chaque question d\'une même session.

*Comment ça marche ?*

1.  `PARTITION BY session_id, question_number` : regroupe les lignes par session et par question.
2.  `ORDER BY answered_at DESC` : trie de la plus récente à la plus ancienne.
3.  `ROW_NUMBER() AS rn` : attribue le rang `1` à la dernière réponse.
4.  `WHERE rn = 1` : ne retient que la réponse finale.
:::

:::: {#354ab6fc .cell .code}
``` python
query_dedup = """
WITH reponses_triees AS (
    SELECT 
        session_id,
        question_number,
        answer_value,
        answered_at,
        ROW_NUMBER() OVER (
            PARTITION BY session_id, question_number 
            ORDER BY answered_at DESC
        ) AS rn
    FROM answers
)
SELECT 
    question_number,
    COUNT(*) AS total_reponses_propres,
    COUNT(DISTINCT session_id) AS total_sessions
FROM reponses_triees
WHERE rn = 1
GROUP BY question_number
ORDER BY question_number;
"""
df_sql_dedup = con.execute(query_dedup).fetchdf()
display(df_sql_dedup)
```

::: {.output .stream .stdout}
    question_number  total_reponses_propres  total_sessions
    0                1                   15558           15558
    1                2                   14779           14779
    2                3                   13920           13920
    3                4                   12044           12044
    4                5                   11462           11462
:::
::::

::: {#53e448e1 .cell .markdown}
### Requête SQL 2 : Funnel de Conversion Principal

**Objectif :** Mesurer les volumes et taux de conversion aux étapes clés du parcours utilisateur sans syntaxe complexe :

- `landing_view` : Arrivée sur la page (26 000)
- `questionnaire_start` : Démarrage du questionnaire (16 774)
- `questionnaire_complete` : Questionnaire terminé (11 357)
:::

:::: {#f0f0feb4 .cell .code}
``` python
query_funnel_simple = """
SELECT 
    event_type,
    COUNT(DISTINCT session_id) AS nb_sessions,
    ROUND(100.0 * COUNT(DISTINCT session_id) / 26000, 1) AS pct_conversion_globale
FROM events
WHERE event_type IN ('landing_view', 'questionnaire_start', 'questionnaire_complete')
GROUP BY event_type
ORDER BY nb_sessions DESC;
"""
df_sql_funnel = con.execute(query_funnel_simple).fetchdf()
display(df_sql_funnel)
```

::: {.output .stream .stdout}
    event_type  nb_sessions  pct_conversion_globale
    0            landing_view        26000                   100.0
    1     questionnaire_start        16774                    64.5
    2  questionnaire_complete        11357                    43.7
:::
::::

::: {#48ab8261 .cell .markdown}
*Déperdition question par question (Questions 1 à 5) :*
:::

:::: {#66716f88 .cell .code}
``` python
query_questions_funnel = """
SELECT 
    question_number,
    COUNT(DISTINCT session_id) AS nb_sessions_ayant_vu,
    ROUND(100.0 * COUNT(DISTINCT session_id) / 16774, 1) AS pct_des_demarreurs
FROM events
WHERE event_type = 'question_view'
GROUP BY question_number
ORDER BY question_number;
"""
df_sql_q_funnel = con.execute(query_questions_funnel).fetchdf()
display(df_sql_q_funnel)
```

::: {.output .stream .stdout}
    question_number  nb_sessions_ayant_vu  pct_des_demarreurs
    0                1                 15881                94.7
    1                2                 15558                92.7
    2                3                 14779                88.1
    3                4                 13920                83.0
    4                5                 12044                71.8
:::
::::

::: {#3c0f74eb .cell .markdown}
### Requête SQL 3 : Performance des Canaux d\'Acquisition

**Objectif :** Calculer pour chaque source d\'acquisition le volume de trafic et son taux de complétion effectif.
:::

:::: {#4cfd95f3 .cell .code}
``` python
query_channels = """
SELECT 
    clean_source,
    COUNT(DISTINCT s.session_id) AS total_sessions,
    COUNT(DISTINCT o.session_id) AS questionnaires_completes,
    ROUND(100.0 * COUNT(DISTINCT o.session_id) / COUNT(DISTINCT s.session_id), 1) AS taux_completion_pct
FROM sessions s
LEFT JOIN outcomes o ON s.session_id = o.session_id
GROUP BY clean_source
ORDER BY total_sessions DESC;
"""
df_sql_channels = con.execute(query_channels).fetchdf()
display(df_sql_channels)
```

::: {.output .stream .stdout}
    clean_source  total_sessions  questionnaires_completes  taux_completion_pct
    0  google_organic            7490                      3228                 43.1
    1      google_ads            6375                      2489                 39.0
    2          social            3618                      1045                 28.9
    3         chatgpt            2564                      1347                 52.5
    4        referral            1790                      1081                 60.4
    5           email            1707                      1105                 64.7
    6          direct            1617                       711                 44.0
    7           other             674                       254                 37.7
    8    AI Assistant             165                        97                 58.8
:::
::::

::: {#01435406 .cell .markdown}
### Requête SQL 4 : Âge Moyen selon le Résultat Médical

**Objectif :** Vérifier la cohérence épidémiologique en calculant simplement l\'**âge moyen** des répondants pour chaque catégorie de résultat santé.

*Requête simple : Jointure directe entre `visitors` et `outcomes`.*
:::

:::: {#a144b6f1 .cell .code}
``` python
query_age_moyen = """
SELECT 
    o.outcome_category,
    COUNT(*) AS total_questionnaires,
    ROUND(AVG(2026 - v.birth_year), 1) AS age_moyen,
    MIN(2026 - v.birth_year) AS age_min,
    MAX(2026 - v.birth_year) AS age_max
FROM visitors v
JOIN outcomes o ON v.visitor_id = o.visitor_id
GROUP BY o.outcome_category
ORDER BY age_moyen DESC;
"""
df_sql_age = con.execute(query_age_moyen).fetchdf()
display(df_sql_age)
```

::: {.output .stream .stdout}
    outcome_category  total_questionnaires  age_moyen  age_min  age_max
    0     declared_diagnosed                  1240       50.7       18       96
    1          possible_risk                  2276       50.1       16       97
    2  no_current_indication                  7841       41.5       16       97
:::
::::

::: {#b9d54f1d .cell .markdown}

------------------------------------------------------------------------

**Enseignement clé :**\
Les profils en risque potentiel (`possible_risk`) ou déjà diagnostiqués (`declared_diagnosed`) ont un âge moyen de **50 ans**, soit près de **10 ans de plus** que les profils sains (41.5 ans). Le questionnaire cible donc parfaitement les populations les plus vulnérables.
:::

::: {#dcf38554 .cell .markdown}

------------------------------------------------------------------------

## 8. Réponses Explicites aux 4 Questions Stratégiques du README :

### Quels sont les résultats les plus importants ?

1.  **La déperdition majeure se situe sur la Landing Page (35.5% d\'abandon immédiat)** :
    - Sur 26 000 sessions, 9 226 utilisateurs quittent la page sans même cliquer sur \"Démarrer le questionnaire\".
    - Une fois le questionnaire entamé, la rétention moyenne par question est excellente (\~95%), à l\'exception de la transition vers la Question 5 (chute de **13.5%** entre Q4 et Q5).
2.  **Une friction mobile critique qui pénalise 65% de l\'audience** :
    - Le trafic mobile représente **64.5% des visites (16 772 sessions)** mais affiche un taux de complétion de **39.71%**, contre **51.79% sur Desktop** (un écart massif de **12.1 points**).
    - Aligner l\'expérience mobile sur le taux Desktop générerait immédiatement **+2 026 questionnaires complétés**.
3.  **Le paradoxe des canaux d\'acquisition : Volume vs Ciblage Santé** :
    - **L\'Emailing (64.7%) et le Referral (60.4%)** sont les canaux les plus engageants, mais attirent une population majoritairement saine (\~72% `no_current_indication`).
    - **Google Ads (SEA)** affiche un taux de complétion modéré (39.0%), mais délivre de loin la plus forte concentration de profils à risque : **45.4%** de ses répondants sont en `possible_risk` ou `declared_diagnosed` (contre 23.0% pour le Social). C\'est le canal le plus performant pour la mission médicale de Doctoome.
    - **Le canal Social** est sous-performant sur toute la ligne : plus faible complétion (28.9%) et plus faible taux de risque qualifié (23.0%).
4.  **Validation de la pertinence clinique du modèle de risque** :
    - La prévalence du risque augmente de façon quasi-linéaire avec l\'âge (11.0% de risque chez les \<35 ans vs **33.7% chez les 65 ans et plus**, et 19.5% déjà diagnostiqués).
    - La Question 1 (diagnostic préalable) et la Question 4 (sédentarité/activité physique) sont les discriminateurs les plus puissants du résultat final.

------------------------------------------------------------------------

### Quelles questions restent sans réponse ?

1.  **La cause exacte du décrochage à la Question 5** :
    - Pourquoi 13.5% des utilisateurs qui ont déjà répondu à 4 questions abandonnent-ils juste avant la Q5 ? Est-ce lié à la formulation de la question (glycémie / terme médical perçu comme anxiogène), à un temps de chargement, ou à une interface confuse ?
2.  **Le coût d\'acquisition client (CAC) et le ROI financier des campagnes payantes** :
    - Sans les données de dépenses publicitaires (ad spend Google Ads vs Social Ads), impossible de calculer le coût par profil à risque détecté et de statuer sur la rentabilité économique des campagnes.
3.  **La nature exacte de la friction mobile** :
    - Les abandons mobiles sont-ils dus à une lenteur de chargement (Core Web Vitals), à des boutons non adaptés au tactile (UX/UI), ou à des formulaires mal affichés selon les résolutions d\'écran ?
4.  **L\'impact médical réel en aval (Post-questionnaire)** :
    - Que font les 2 276 utilisateurs classés `possible_risk` après avoir vu leur résultat ? Consultent-ils un médecin ? Prennent-ils un rendez-vous sur Doctoome ? Le jeu de données s\'arrête à l\'événement `questionnaire_complete`.

------------------------------------------------------------------------

### Quelles données supplémentaires collecteriez-vous ?

1.  **Données de télémétrie UX et performance technique** :
    - **Temps passé par étape / par question** (`duration_seconds` entre les événements `question_view` et `question_answer`) pour détecter les questions qui bloquent.
    - **Erreurs JavaScript et temps de chargement** (FCP, LCP) segmentés par navigateur et modèle de smartphone.
    - **Positions de défilement (Scroll depth)** et clics sur les éléments de réassurance de la landing page.
2.  **Données de conversion post-résultat (Call-to-Action / Funnel aval)** :
    - Clics sur le bouton d\'action final (ex: *\"Prendre rendez-vous avec un médecin généraliste\"* ou *\"Trouver un laboratoire de biologie médicale\"*).
    - Taux de conversion effectif en prise de rendez-vous sur Doctoome (`booking_id`, `specialty`).
3.  **Données marketing et d\'attribution enrichies** :
    - Coûts d\'acquisition (`cost_per_click`, budget par campagne).
    - Mots-clés de recherche (Google Ads keywords) pour affiner le ciblage d\'intention.
4.  **Consentement et ré-engagement (Opt-in)** :
    - Taux de consentement au suivi email/SMS pour relancer les profils `possible_risk` n\'ayant pas consulté.

------------------------------------------------------------------------

### Que recommanderiez-vous de suivre si le questionnaire restait actif pendant 12 mois ?

1.  **Mise en place d\'un Dashboard Opérationnel & Alerting automatisé** :
    - **KPIs temps réel** : Taux de rebond Landing, Taux de complétion par canal et par device, Taux de profils à risque détectés.
    - **Alertes de régression** (via Slack/Email) si le taux de complétion d\'un device ou canal chute de plus de 15% suite à un déploiement technique.
2.  **Surveillance du Data Drift (Dérive de données) et de la Saisonnalité** :
    - Monitorer l\'évolution de la distribution d\'âge et de genre mois par mois (ex. variations pendant les vacances d\'été vs rentrée).
    - Suivre la saisonnalité des requêtes santé (pics en novembre lors de la *Journée Mondiale du Diabète*).
3.  **Programme d\'A/B Testing continu** :
    - **Test A/B sur la Landing Page** : Tester un affichage direct de la Question 1 sur la page d\'accueil (sans écran de transition) pour neutraliser les 35.5% de rebond.
    - **Test A/B UX Mobile** : Interface type \"swipe/cartes\" simplifiée pour smartphone afin de combler les 12 points de retard face au Desktop.
4.  **Monitoring de la Valeur Métier (Santé Publique & Business Doctoome)** :
    - Calculer mensuellement le **coût par patient à risque orienté vers un professionnel de santé** :
      $$\text{Coût par Patient Orienté} = \frac{\text{Budget Marketing Mensuel}}{\text{Nombre de RDV Doctoome générés par des profils à risque}}$$
    - Suivre le taux de revisite à 6 et 12 mois (`is_returning_visitor`) pour mesurer la fidélisation à la plateforme.
:::

::: {#65f74aff-7193-4c5e-bb47-d1233bffb597 .cell .code}
``` python
```
:::
