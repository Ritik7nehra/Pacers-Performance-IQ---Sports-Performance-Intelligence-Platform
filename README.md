Pacers Performance IQ

An end-to-end basketball sports-performance intelligence platform for daily readiness review, athlete testing, markerless movement screening, game-performance context, and staff communication.

Unofficial portfolio project by Ritik Nehra. It is not affiliated with the Indiana Pacers, Pacers Sports & Entertainment, or the NBA. All athlete-monitoring, testing, wellness, and movement data included in the repository are synthetic and de-identified.

Why this project is stronger than a typical basketball analytics notebook

Most public basketball projects stop at shot charts, player rankings, or game predictions. This project models the work of a sports performance intelligence function:

Integrate daily workload, wellness, force-plate-style testing, movement screening, and game data.

Compare each athlete with their own recent baseline rather than a universal cutoff.

Estimate whether a test change is larger than normal measurement noise.

Surface an explainable staff action queue with data-confidence labels.

Connect pregame context to game outcomes without making causal claims.

Track data quality, privacy classification, and model limitations.

It demonstrates the combination of sports science reasoning, data engineering, statistical analysis, computer vision, SQL, R, Python, and dashboard design expected in a performance environment.

Staff question the system answers

Who needs attention today, what changed, how confident are we, and what should staff review next?

The output is decision support—not a medical diagnosis, injury prediction, participation decision, or return-to-play clearance.

Portfolio evidence by skill area

Target skill

Evidence in this repository

Sports performance analytics

Individualized readiness deviations, internal/external load, rolling exposure context, game-performance bridge

Athlete testing and evaluation

CMJ, 10 m sprint, 505 change of direction, IMTP-style metrics, trial CV, typical error, ICC workflow in R, measurement-aware change signals

Biomechanics and movement assessment

MediaPipe/OpenCV video pipeline, joint-angle time series, landing asymmetry, trunk lean, confidence filtering, annotated video output

Data visualization and dashboards

Interactive Streamlit application plus a self-contained HTML executive demo

SQL

Star-style SQLite warehouse, indexed fact/mart tables, reusable views, staff queries, data catalog

Python

Synthetic data generation, ETL, robust baseline features, testing analytics, visualization, computer vision, automated tests

R

Repeated-trial reliability analysis with ICC(2,1), CV, typical error, SEM, and smallest worthwhile change demonstration

Communication and governance

Staff action notes, reason codes, confidence levels, data-quality audit, explicit limitations and ethical controls

System architecture

flowchart LR
    A[Session load / tracking] --> E[Python ingestion and validation]
    B[Wellness questionnaires] --> E
    C[Athlete testing trials] --> E
    D[Standardized movement video] --> V[MediaPipe pose pipeline]
    V --> E
    G[Public game data or demo box scores] --> E

    E --> W[(SQLite analytics warehouse)]
    W --> F[Individual baselines and rolling features]
    W --> T[Test reliability and meaningful-change layer]
    W --> M[Movement-screening mart]
    F --> Q[Explainable staff action queue]
    T --> Q
    M --> Q
    Q --> S[Streamlit dashboard]
    Q --> H[Standalone HTML dashboard]
    W --> P[Power BI-ready model]

Core analytical design

1. Individualized daily monitoring

For each athlete, the pipeline calculates leakage-safe robust z-scores from the previous 28 observations only. It evaluates:

Sleep relative to the athlete's recent pattern

Fatigue and soreness deviations

External-load deviations

Latest CMJ jump height and RSI-mod, carried forward for no more than seven days

Seven-day and 28-day exposure context

A transparent score summarizes the components, but the dashboard always shows the primary contributing factor, source values, and data confidence. The first ten observations are labeled Building baseline rather than receiving a misleading score.

2. Athlete testing and evaluation

The synthetic testing system includes repeated trials for:

Countermovement jump: jump height, RSI-mod, peak power per kg, countermovement depth, peak braking force per kg

10 m sprint time

505 change-of-direction time by side

Isometric mid-thigh pull-style peak force per kg

The analytical layer calculates:

Session mean, standard deviation, and coefficient of variation

Recent 28-day mean and between-session variation

Typical-error estimate from trial variation

Smallest worthwhile change demonstration

Improved / Stable / Declined signals only when change exceeds the larger measurement-aware threshold

The R workflow adds ICC(2,1), SEM, pooled typical error, athlete-level CV summaries, and a reliability visualization.

3. Biomechanics and movement screening

src/biomechanics/pose_assessment.py uses the MediaPipe Tasks Pose Landmarker interface to process a standardized video and export:

Frame-by-frame left/right knee and hip angles

Trunk lean from image vertical

Knee-separation ratio

Inter-limb knee-angle asymmetry

Pose confidence

Transparent movement-screening score

Annotated video and trial summary

The module is deliberately labeled screening only. Camera standardization, visual review, metric reliability, and validation against a trusted reference system are required before operational use.

4. Game-performance bridge

The platform joins pregame readiness context with a synthetic per-36 game-performance index, eFG%, minutes, usage, and other box-score-style features. The dashboard reports a descriptive association and states that it is not causal. Opponent, role, tactics, score state, and other confounders remain.

5. Data quality and governance

The pipeline tracks:

Primary-key completeness and duplicates

Overall missingness

Data-source labels

Synthetic/de-identified classification

Warehouse layer, source file, row count, and column count

Data-confidence status on staff-facing flags

Quick start

Run the complete demo

python -m pip install -r requirements.txt
python -m src.run_pipeline

This command:

Generates a deterministic synthetic season.

Builds individualized features and analytical marts.

Creates warehouse.sqlite and SQL views.

Exports the standalone dashboard and latest staff action queue.

Open the interactive dashboard

streamlit run dashboard/app.py

Open the no-server dashboard

Open:

outputs/pacers_performance_iq_demo.html

The HTML file is self-contained and includes its visualization library.

Run tests

python -m pytest

Run the R reliability workflow

Rscript r/test_reliability.R \
  data/raw/testing_long.csv \
  outputs/r_reliability \
  jump_height_cm \
  CMJ

Analyze a movement video

Install the optional computer-vision dependencies, obtain an official compatible MediaPipe task model, and follow models/README.md.

python -m pip install -r requirements-biomechanics.txt
python -m src.biomechanics.pose_assessment \
  --video data/raw/drop_jump.mp4 \
  --model models/pose_landmarker_full.task \
  --athlete-id P01 \
  --date 2026-04-15

Warehouse model

Dimensions

dim_athlete

dim_game

Source facts

fact_session

fact_wellness

fact_test_trial

fact_movement_trial

fact_game_performance

Analytical marts

mart_daily_status

mart_team_daily

mart_test_session

mart_test_reliability

mart_test_change

mart_movement_session

mart_game_readiness

audit_data_quality

Operational views

vw_latest_team_status

vw_staff_action_queue

vw_latest_test_signals

vw_latest_movement

vw_game_performance_bridge

See sql/03_analysis_queries.sql for interview-ready examples.

Repository map

pacers-performance-intelligence/
├── dashboard/app.py                  # Interactive staff dashboard
├── data/raw/                         # Synthetic source tables
├── data/processed/                   # Analytical marts
├── docs/                             # Project, validation, Power BI, interview material
├── models/README.md                  # MediaPipe model and capture setup
├── outputs/                          # Standalone dashboard and staff queue
├── r/test_reliability.R              # R measurement-quality workflow
├── sql/                              # Schema, views, and analysis queries
├── src/generate_demo_data.py         # Synthetic season generator
├── src/feature_engineering.py        # Baselines, testing, movement, game bridge
├── src/build_warehouse.py            # SQLite build and metadata catalog
├── src/export_static_dashboard.py    # Self-contained HTML output
├── src/biomechanics/pose_assessment.py
└── tests/                            # Unit and integration checks

Suggested live demonstration

Open Morning board and explain the action queue.

Select the lowest-scoring athlete and show the factor-level evidence.

Move to Testing lab and show why a change must exceed measurement noise.

Open Movement and explain confidence, camera protocol, and validation limits.

Show the SQL view behind the action queue.

End with Data quality to demonstrate production awareness.

Interview summary

I built a synthetic but production-shaped basketball performance intelligence platform. It integrates daily load, wellness, repeated athlete testing, movement-video features, and game data into a SQL warehouse. Python creates individualized baseline deviations and an explainable staff queue; R evaluates trial reliability; MediaPipe extracts movement features; and Streamlit presents the results with confidence and governance controls. I intentionally avoided injury prediction and designed the system to support—not replace—performance and medical staff judgment.

Important limitations

No real Pacers athlete, health, medical, tracking, or practice data are used.

The scoring weights and thresholds are demonstration choices requiring stakeholder calibration and prospective validation.

Synthetic game outcomes are useful for software demonstration, not basketball conclusions.

Markerless 2D measures depend on camera plane, occlusion, lighting, frame rate, and model error.

Asymmetry is a descriptive feature; its reliability and interpretation are metric- and protocol-specific.

Association between monitoring variables and game performance is not evidence of causation.

Any operational system would require privacy review, role-based access, secure storage, audit logging, and approved data-retention policies.

Research and implementation foundations

Official Pacers analytics role descriptions emphasize multi-source data integration, internal tools, analytical infrastructure, reporting, Python, SQL, Power BI, and communication with basketball staff.

MediaPipe Pose Landmarker supports image, video, and live-stream modes and returns image-coordinate and 3D world landmarks.

Recent validation work supports the promise of low-cost markerless movement capture while showing stronger agreement in sagittal-plane hip/knee measures than frontal or transverse measures.

CMJ monitoring literature reinforces the importance of test reliability, CV/ICC, and caution around less reliable asymmetry variables.

Detailed sources and their design implications are listed in docs/VALIDATION_AND_ETHICS.md.
