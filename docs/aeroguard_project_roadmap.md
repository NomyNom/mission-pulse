# AeroGuard: Mission Control Telemetry + Predictive Maintenance System

## Project Goal

Build a mission-control-style system that replays aerospace engine telemetry, monitors sensor health, detects anomalies, predicts remaining useful life, and displays alerts in Open MCT.

The project should demonstrate an end-to-end aerospace ML/software engineering workflow:

```text
C-MAPSS data
    ↓
Telemetry replay
    ↓
FastAPI backend
    ↓
Anomaly detection + RUL model
    ↓
Open MCT dashboard
    ↓
Operator sees health status and alerts
```

---

## Big Picture

This project simulates the type of system used to monitor aircraft, spacecraft, satellites, drones, rovers, or engines during operation.

A real mission system receives telemetry from a vehicle or subsystem, processes it, checks whether the system is healthy, detects abnormal behavior, and alerts operators when action may be needed.

The simplified workflow is:

```text
Vehicle/system is operating
        ↓
Sensors produce telemetry
        ↓
Ground/backend system receives telemetry
        ↓
Software checks health and detects issues
        ↓
Mission operators see alerts and make decisions
```

For this project, C-MAPSS turbofan data acts as the simulated aerospace engine telemetry.

---

## MVP Statement

Use NASA C-MAPSS turbofan data to simulate live engine telemetry, serve it through FastAPI, visualize it in Open MCT, and show basic anomaly alerts and equipment health predictions.

---

## Core System Architecture

```text
C-MAPSS Dataset
    ↓
Data Preprocessing Pipeline
    ↓
Telemetry Replay Service
    ↓
FastAPI Backend
    ↓
ML Models
    ├── RUL Prediction
    └── Anomaly Detection
    ↓
Open MCT Dashboard
    ↓
Alerts + Health Monitoring
```

---

## Main Technologies

### Backend

- Python
- FastAPI
- pandas
- NumPy
- scikit-learn
- Optional later: PyTorch

### Frontend / Telemetry Visualization

- Open MCT
- JavaScript telemetry provider

### Machine Learning

- Baseline RUL model: Random Forest, XGBoost, or CatBoost
- Baseline anomaly model: rolling z-score or Isolation Forest
- Advanced later: LSTM, GRU, 1D CNN, autoencoder, LSTM autoencoder

### Deployment / Packaging

- Docker
- Docker Compose
- README documentation
- Architecture diagram
- Demo screenshots or video

---

# Project Roadmap

## Phase 0 — Define Project Scope

### Goal

Decide exactly what the first version should do.

### MVP Scope

```text
Input: C-MAPSS turbofan sensor data
Backend: FastAPI
Frontend: Open MCT
ML: anomaly detection + RUL prediction
Output: telemetry dashboard, health score, alerts
```

### Do Not Include Yet

Avoid these early because they can become bloat:

- user accounts
- complex authentication
- cloud deployment
- Kubernetes
- many datasets
- complex custom frontend outside Open MCT
- multiple model architectures at the start
- over-engineered microservices

### Output of This Phase

A short project spec describing:

- project goal
- dataset
- backend
- dashboard
- ML tasks
- final deliverable

---

## Phase 1 — Understand and Prepare the Data

### Goal

Get comfortable with the C-MAPSS dataset and reshape it into a telemetry-friendly format.

### Tasks

1. Download/load C-MAPSS data.
2. Understand the columns:
   - engine/unit id
   - cycle number
   - operating settings
   - sensor readings
3. Add readable column names.
4. Create a processed dataframe.
5. Create the RUL target.

```python
RUL = max_cycle_for_engine - current_cycle
```

6. Convert the data into a telemetry-friendly format if needed.

Example long telemetry format:

```text
unit_id | cycle | timestamp | sensor_name | value
```

### Why This Matters

Open MCT works better when telemetry can be requested by object/channel, such as:

```text
engine_1.sensor_7
engine_1.health_score
engine_1.anomaly_score
```

### Output

Processed data files:

```text
data/processed/processed_telemetry.csv
data/processed/processed_rul_training.csv
```

---

## Phase 2 — Build a Baseline RUL Model

### Goal

Create the first useful health prediction model.

### Model Task

Predict remaining useful life.

Input:

```text
sensor readings + operating settings + cycle
```

Output:

```text
remaining useful life
```

### Start Simple

Use one of:

- Random Forest
- XGBoost
- CatBoost

Do not start with LSTM immediately. Build a baseline first.

### Metrics

Track:

- MAE
- RMSE
- predicted vs actual RUL plot

### Output

Saved model:

```text
models/rul_model.pkl
```

Example prediction:

```json
{
  "unit_id": 7,
  "cycle": 140,
  "predicted_rul": 37,
  "health_score": 0.31,
  "status": "warning"
}
```

---

## Phase 3 — Build Simple Anomaly Detection

### Goal

Detect abnormal telemetry behavior.

Start with explainable methods before advanced methods.

### Option A — Rolling Z-Score

Compare the current sensor value against its recent rolling average and standard deviation.

Example logic:

```text
if abs(z_score) > 3:
    anomaly = true
```

This is simple, explainable, and good for a first version.

### Option B — Isolation Forest

Train an Isolation Forest on sensor values and operating settings.

Output:

```text
normal or anomalous
anomaly score
```

### Recommended First Version

Start with rolling z-score, then add Isolation Forest later.

### Output

For each engine/cycle:

```json
{
  "unit_id": 7,
  "cycle": 140,
  "anomaly_score": 0.82,
  "anomaly_status": "warning",
  "top_warning_sensors": ["sensor_4", "sensor_11"]
}
```

---

## Phase 4 — Create the FastAPI Backend

### Goal

Serve telemetry and predictions to Open MCT.

### Suggested Endpoints

```text
GET /units
GET /sensors
GET /telemetry/history
GET /telemetry/latest
GET /predictions/rul
GET /predictions/anomaly
GET /health-status
```

### Example Endpoint

```text
GET /telemetry/history?unit_id=7&sensor=sensor_4&start=1&end=100
```

Example response:

```json
[
  {
    "timestamp": 1,
    "value": 1398.2
  },
  {
    "timestamp": 2,
    "value": 1401.7
  }
]
```

### Output

A working backend where endpoints can be tested in:

```text
/docs
```

---

## Phase 5 — Telemetry Replay

### Goal

Simulate live mission telemetry.

The dataset is historical, so replay it like a live stream.

### Basic Idea

```text
engine 7, cycle 1
engine 7, cycle 2
engine 7, cycle 3
...
```

Every second or every few seconds, the latest telemetry changes.

### Two Approaches

#### Simple Polling

Open MCT asks FastAPI for latest values every few seconds.

```text
Open MCT → GET /telemetry/latest
```

#### Advanced Streaming

FastAPI pushes telemetry updates to Open MCT using WebSockets.

```text
FastAPI WebSocket → Open MCT realtime telemetry
```

### Recommended First Version

Start with polling. Add WebSockets later.

### Output

You can watch an engine “run” cycle by cycle.

---

## Phase 6 — Open MCT Integration

### Goal

Display telemetry in a mission-control-style dashboard.

### Tasks

1. Set up an Open MCT app.
2. Create a custom telemetry dictionary.
3. Define telemetry objects:
   - engines
   - sensors
   - health score
   - anomaly score
   - predicted RUL
   - alert status
4. Build a telemetry provider that calls FastAPI.
5. Display historical and latest telemetry.

### Example Open MCT Object Tree

```text
Engine 7
  ├── sensor_2
  ├── sensor_3
  ├── sensor_4
  ├── sensor_11
  ├── predicted_rul
  ├── health_score
  └── anomaly_score
```

### Output

Open MCT dashboard showing:

- raw sensor charts
- predicted RUL
- health score
- anomaly score
- alert status

---

## Phase 7 — Alerting Logic

### Goal

Turn model outputs into decision-friendly status levels.

### Alert Levels

```text
NORMAL
WATCH
WARNING
CRITICAL
```

### Example Logic

```text
NORMAL:
anomaly_score < 0.4 and health_score > 0.7

WATCH:
anomaly_score >= 0.4 or health_score <= 0.7

WARNING:
anomaly_score >= 0.7 or health_score <= 0.4

CRITICAL:
anomaly_score >= 0.9 or health_score <= 0.2
```

### Example API Response

```json
{
  "unit_id": 7,
  "status": "warning",
  "reason": "High anomaly score and decreasing predicted RUL"
}
```

This makes the system useful instead of only displaying numbers.

---

## Phase 8 — Package the Project

### Goal

Make the project easy to run, explain, and demo.

### Tasks

1. Add a strong README.
2. Add an architecture diagram.
3. Add screenshots.
4. Add model evaluation results.
5. Add Docker.
6. Add simple run instructions.

### Docker Target

Eventually this command should start the project:

```bash
docker compose up
```

It should run:

```text
FastAPI backend
Open MCT frontend
database or local data volume
```

### Output

A polished GitHub portfolio project.

---

## Phase 9 — Advanced Upgrades

Only do these after the MVP works.

### Upgrade A — Better RUL Model

Add sequence models:

- LSTM
- GRU
- 1D CNN

Compare them against the baseline model.

Do not remove the baseline. Keep both and report the comparison.

---

### Upgrade B — LSTM Autoencoder Anomaly Detection

Train a model to reconstruct normal telemetry sequences.

```text
normal pattern → low reconstruction error
abnormal pattern → high reconstruction error
```

This is a strong anomaly detection upgrade.

---

### Upgrade C — Explainability

Add explanations such as:

- top contributing sensors
- largest sensor deviations
- feature importance
- SHAP for tree-based RUL model

Example:

```text
Warning caused mainly by:
1. sensor_11 pressure drift
2. sensor_4 temperature increase
3. sensor_7 vibration instability
```

---

### Upgrade D — Time-Series Database

Add one of these later:

- PostgreSQL
- TimescaleDB
- InfluxDB

Only add this after the API and Open MCT integration work.

---

### Upgrade E — Real-Time WebSocket Streaming

Replace polling with live push updates.

```text
FastAPI WebSocket → Open MCT realtime telemetry
```

---

# Recommended Build Order

```text
1. C-MAPSS data preprocessing
2. RUL baseline model
3. simple anomaly detection
4. FastAPI backend
5. telemetry replay
6. Open MCT dashboard
7. alert status logic
8. Docker + README
9. advanced ML upgrades
```

---

# Clean MVP Feature List

## AeroGuard MVP

AeroGuard MVP replays NASA C-MAPSS turbofan telemetry through a FastAPI backend and visualizes engine sensor behavior in Open MCT. It includes basic anomaly detection, predicted remaining useful life, health scoring, and alert statuses for simulated mission-control monitoring.

### MVP Features

- Load C-MAPSS data
- Process engine telemetry
- Predict RUL using a baseline model
- Calculate health score
- Detect basic anomalies
- Serve telemetry through FastAPI
- Replay telemetry
- Display telemetry in Open MCT
- Show alert status

---

# Feature Control Rule

Before adding a feature, ask:

> Does this help monitor telemetry, detect abnormal behavior, predict equipment health, or make the system easier to run?

If yes, it probably belongs.

If no, skip it.

Good upgrades:

1. better anomaly detection
2. better RUL prediction
3. real-time telemetry replay
4. clear alerting
5. explainability
6. Dockerized deployment

Avoid early:

- authentication
- Kubernetes
- too many datasets
- too many model types at once
- custom frontend before Open MCT works
- over-engineered microservices
- real-time database before basic replay works

---

# Model-First vs Dashboard-First

## Option 1 — Model-First

Train RUL and anomaly models first, then connect them to Open MCT.

### Pros

- easier to validate ML
- better for learning time-series modeling
- less frontend frustration early

### Cons

- project may feel like a notebook for longer

---

## Option 2 — Dashboard-First

Get Open MCT displaying simple telemetry first, then add ML.

### Pros

- faster visual progress
- easier to understand the full system
- more motivating

### Cons

- Open MCT setup may slow progress early

---

## Recommended Approach — Hybrid

Build the system skeleton early, then improve it with ML.

```text
1. Process C-MAPSS data
2. Build simple FastAPI telemetry endpoint
3. Connect Open MCT to display one engine/sensor
4. Add RUL model
5. Add anomaly detection
6. Add health score and alerts
```

This gives you a working system early while still leaving room for serious ML work.

---

# Final Portfolio Description

AeroGuard is an end-to-end aerospace telemetry monitoring system that uses NASA C-MAPSS turbofan degradation data to simulate mission telemetry, detect sensor anomalies, predict remaining useful life, and visualize system health through Open MCT.

---

# Possible Resume Bullet

Built an end-to-end aerospace telemetry monitoring system using NASA C-MAPSS turbofan degradation data, FastAPI, Open MCT, and time-series ML models to simulate mission telemetry, detect anomalous sensor behavior, and predict remaining useful life for engine health monitoring.
