# Doctoome – Data Engineer / Data Analyst Internship
## Technical Case Study

This exercise is part of the recruitment process for a **6-month internship**.

### Context

Doctoome has launched a landing page for a Type 2 Diabetes awareness questionnaire.
Visitors arrive through several acquisition channels and may leave, start the
five-question flow, or complete it. Completed questionnaires receive one of three
analytical categories:

- `no_current_indication`: no current indication according to this questionnaire;
- `possible_risk`: a possible-risk category according to this questionnaire;
- `declared_diagnosed`: a reported prior diagnosis in the questionnaire.

All supplied data is synthetic and represents no real patients or identifiable people.
**The questionnaire and its outcomes are fictional and simplified for this recruitment
exercise. They must not be interpreted as medical diagnoses or clinical recommendations.**
The questionnaire is not a validated medical diagnostic tool.

### Your mission

Your objective is to explore the supplied data and present the findings you believe
are most useful to Doctoome. The exercise is deliberately open-ended: choose and
justify your priorities. You might investigate the questionnaire funnel, acquisition,
demographics, responses, user behaviour, data quality, or changes over time. These
are examples, not requirements or a mandatory analysis checklist.

### Available tables

The six files in `data/` use Parquet format. Timestamps are in UTC. The observation
period is June–August 2026. Identifiers can be used to join the tables:
`visitor_id` links visitors to sessions and activity; `session_id` links sessions
to activity and outcomes; `question_id` links answers to the question catalog.

| File | Grain and columns |
|---|---|
| `visitors.parquet` | One row per visitor. `visitor_id`: stable identifier; `birth_year`: reported birth year; `gender`: reported category; `region`: reported French region; `first_seen_at`: first observed visit. |
| `sessions.parquet` | One row per landing-page session. `session_id`: identifier; `visitor_id`: visitor; `session_started_at`: arrival time; `acquisition_source`: recorded source; `campaign_name`: campaign attribution, when available; `device_type`: mobile, desktop or tablet; `landing_page`: URL path; `is_returning_visitor`: whether an earlier session for this visitor exists in the supplied observation period. |
| `questionnaire_questions.parquet` | One row per question. `question_id`: identifier; `question_number`: position 1–5; `question_text`: wording; `response_type`: input type; `allowed_answers`: JSON string containing response options. The questions are fictional and simplified for this recruitment exercise. |
| `questionnaire_events.parquet` | One row per recorded event. `event_id`: identifier; `session_id` and `visitor_id`: associated session and visitor; `event_timestamp`: event time; `event_type`: recorded action; `question_number`: related position, or null for non-question events. Event types are `landing_view`, `questionnaire_start`, `question_view`, `question_answer`, and `questionnaire_complete`. |
| `questionnaire_answers.parquet` | One row per recorded answer. `answer_id`: identifier; `session_id` and `visitor_id`: associated session and visitor; `question_id` and `question_number`: related question; `answer_value`: recorded response; `answered_at`: recording time. |
| `questionnaire_outcomes.parquet` | One row per completed questionnaire. `session_id` and `visitor_id`: associated session and visitor; `outcome_category`: one of the three categories above; `completed_at`: completion time. |

Some descriptive or attribution fields may be unavailable. Document your own
assumptions about the data and any transformations you make.

### Technical expectations

Demonstrate Python **and SQL**, data cleaning and transformation, analytical reasoning,
reproducibility, and clear communication. Use visualisations where useful.
You may use pandas, Polars, DuckDB, SQLite, PySpark, Matplotlib, Plotly, or other
reasonable libraries. A complex stack is not an advantage by itself.

### Deliverables

Submit your source code, SQL queries used, reproduction instructions (including
dependencies), analysis and supporting visualisations, and a short presentation.
A dashboard is optional. Make it clear how to reproduce your work from the supplied
files and which assumptions affect your conclusions.

At the end, explicitly answer:

1. What are the most important findings?
2. What questions remain unanswered?
3. What additional data would you collect?
4. What would you recommend monitoring if the questionnaire continued for 12 months?

### Timing and interview

You have **72 hours from receipt to submission**. This is an open-ended case: you
are not expected to investigate every possible direction. Prioritisation is part
of the assessment.

The subsequent one-hour interview includes:

- 15 minutes for your presentation;
- 15 minutes of Q&A about your submitted work;
- 30 minutes of live coding using separate datasets, not the files in this package.

### AI and external tools

You may use documentation and development tools. If you use generative AI, you
remain responsible for your solution and must be able to explain and modify all
submitted work during the technical interview.
