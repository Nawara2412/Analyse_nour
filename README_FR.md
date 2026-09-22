# Doctoome – Stage Data Engineer / Data Analyst
## Étude de cas technique

Cet exercice fait partie du processus de recrutement pour un **stage de 6 mois**.

### Contexte

Doctoome a lancé une page dédiée à un questionnaire de sensibilisation au diabète de type 2.
Les visiteurs arrivent via différents canaux d’acquisition et peuvent quitter la page, commencer le
questionnaire de cinq questions ou le terminer. Les questionnaires complétés sont classés dans l’une
des trois catégories analytiques suivantes :

- `no_current_indication` : aucune indication actuelle selon ce questionnaire ;
- `possible_risk` : catégorie correspondant à un risque potentiel selon ce questionnaire ;
- `declared_diagnosed` : diagnostic antérieur déclaré dans le questionnaire.

Toutes les données fournies sont synthétiques et ne représentent aucun patient réel ni aucune personne identifiable.
**Le questionnaire et ses résultats sont fictifs et simplifiés pour les besoins de cet exercice de recrutement.
Ils ne doivent pas être interprétés comme des diagnostics médicaux ou des recommandations cliniques.**
Ce questionnaire n’est pas un outil de diagnostic médical validé.

### Votre mission

Votre objectif est d’explorer les données fournies et de présenter les conclusions que vous jugez
les plus utiles pour Doctoome. L’exercice est volontairement ouvert : à vous de choisir vos priorités
et de les justifier. Vous pouvez, par exemple, vous intéresser au funnel du questionnaire, à l’acquisition,
aux données démographiques, aux réponses, au comportement des utilisateurs, à la qualité des données
ou encore à leur évolution dans le temps. Il s’agit d’exemples et non d’une liste obligatoire d’analyses.

### Tables disponibles

Les six fichiers présents dans `data/` sont au format Parquet. Les horodatages sont en UTC. La période
d’observation couvre juin à août 2026. Les identifiants permettent de relier les différentes tables :
`visitor_id` relie les visiteurs aux sessions et à l’activité ; `session_id` relie les sessions
à l’activité et aux résultats ; `question_id` relie les réponses au catalogue des questions.

| Fichier | Granularité et colonnes |
|---|---|
| `visitors.parquet` | Une ligne par visiteur. `visitor_id` : identifiant stable ; `birth_year` : année de naissance déclarée ; `gender` : catégorie de genre déclarée ; `region` : région française déclarée ; `first_seen_at` : première visite observée. |
| `sessions.parquet` | Une ligne par session sur la page d’atterrissage. `session_id` : identifiant ; `visitor_id` : visiteur ; `session_started_at` : heure d’arrivée ; `acquisition_source` : source d’acquisition enregistrée ; `campaign_name` : attribution à une campagne, lorsqu’elle est disponible ; `device_type` : mobile, desktop ou tablette ; `landing_page` : chemin de l’URL ; `is_returning_visitor` : indique si une session antérieure existe pour ce visiteur durant la période d’observation fournie. |
| `questionnaire_questions.parquet` | Une ligne par question. `question_id` : identifiant ; `question_number` : position de 1 à 5 ; `question_text` : formulation ; `response_type` : type de saisie ; `allowed_answers` : chaîne JSON contenant les réponses possibles. Les questions sont fictives et simplifiées pour les besoins de cet exercice de recrutement. |
| `questionnaire_events.parquet` | Une ligne par événement enregistré. `event_id` : identifiant ; `session_id` et `visitor_id` : session et visiteur associés ; `event_timestamp` : date et heure de l’événement ; `event_type` : action enregistrée ; `question_number` : position de la question concernée, ou valeur nulle pour les événements non liés à une question. Les types d’événements sont `landing_view`, `questionnaire_start`, `question_view`, `question_answer` et `questionnaire_complete`. |
| `questionnaire_answers.parquet` | Une ligne par réponse enregistrée. `answer_id` : identifiant ; `session_id` et `visitor_id` : session et visiteur associés ; `question_id` et `question_number` : question concernée ; `answer_value` : réponse enregistrée ; `answered_at` : date et heure d’enregistrement. |
| `questionnaire_outcomes.parquet` | Une ligne par questionnaire complété. `session_id` et `visitor_id` : session et visiteur associés ; `outcome_category` : l’une des trois catégories mentionnées ci-dessus ; `completed_at` : date et heure de complétion. |

Certaines informations descriptives ou d’attribution peuvent être indisponibles. Documentez vos propres
hypothèses concernant les données ainsi que les transformations que vous effectuez.

### Attentes techniques

Vous devez démontrer votre maîtrise de **Python et SQL**, du nettoyage et de la transformation des données,
du raisonnement analytique, de la reproductibilité et de la communication des résultats.
Utilisez des visualisations lorsque cela est pertinent.

Vous pouvez utiliser pandas, Polars, DuckDB, SQLite, PySpark, Matplotlib, Plotly ou toute autre
bibliothèque raisonnable. L’utilisation d’une stack technique complexe ne constitue pas un avantage en soi.

### Livrables

Remettez votre code source, les requêtes SQL utilisées, les instructions permettant de reproduire votre travail
(y compris les dépendances), votre analyse et les visualisations associées, ainsi qu’une courte présentation.

La création d’un dashboard est facultative. Votre projet doit expliquer clairement comment reproduire votre travail
à partir des fichiers fournis et quelles hypothèses peuvent avoir un impact sur vos conclusions.

À la fin de votre analyse, répondez explicitement aux questions suivantes :

1. Quels sont les résultats les plus importants ?
2. Quelles questions restent sans réponse ?
3. Quelles données supplémentaires collecteriez-vous ?
4. Que recommanderiez-vous de suivre si le questionnaire restait actif pendant 12 mois ?

### Délai et entretien

Vous disposez de **72 heures à compter de la réception du cas pour remettre votre travail**. Il s’agit d’un cas
ouvert : nous ne nous attendons pas à ce que vous exploriez toutes les pistes possibles. Votre capacité à
prioriser fait partie de l’évaluation.

L’entretien technique d’une heure qui suivra comprendra :

- 15 minutes de présentation à partir de votre support **C-level en slides** ;
- 15 minutes de questions-réponses sur le travail remis ;
- 30 minutes de live coding à partir de jeux de données distincts de ceux fournis dans ce dossier.

### IA et outils externes

Vous pouvez utiliser de la documentation et des outils de développement. Si vous utilisez de l’IA générative,
vous restez responsable de votre solution et devez être capable d’expliquer et de modifier l’ensemble du travail
remis pendant l’entretien technique.
