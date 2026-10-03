# RLBased-SmartGridManagement
(SmartGrid)

```markdown
# ⚡ AI-Powered Smart Grid Energy Management System

An AI-based smart grid prototype that combines **renewable energy forecasting, electricity demand forecasting, and reinforcement learning** to intelligently manage battery energy storage and reduce grid dependency and solar energy curtailment.

---

## 📌 Overview

Modern power grids need to continuously balance:

- Electricity demand
- Renewable energy generation
- Battery storage
- Grid energy imports
- Excess renewable energy

This project proposes an AI-driven energy management system that first predicts **future electricity demand and solar generation**, and then uses a **Deep Q-Network (DQN)** reinforcement learning agent to decide how a simulated battery should operate.

The system follows:

```text
Historical Energy Data
        │
        ├───────────────┐
        ↓               ↓
 Load Forecast     Solar Forecast
   XGBoost            XGBoost
        │               │
        └───────┬───────┘
                ↓
        Future Grid State
                ↓
          DQN Agent
          "Grid Brain"
                ↓
       Battery Action
                ↓
       Energy Simulator
                ↓
        Grid Performance
                ↓
           Dashboard
```

The prototype is designed as a **software-based proof of concept** for intelligent energy management.

---

# 🎯 Objectives

The main objectives of this project are:

1. Forecast electricity demand using historical consumption data.
2. Forecast solar generation using historical solar data and weather information.
3. Use the forecasts as inputs to an RL-based energy management agent.
4. Simulate battery charging and discharging decisions.
5. Reduce unnecessary grid energy imports.
6. Reduce renewable energy curtailment.
7. Compare AI-based control with a rule-based controller and a no-battery scenario.
8. Provide a visual dashboard for monitoring the simulated smart grid.

---

# 🧠 System Architecture

```text
                    ┌─────────────────────┐
                    │  Historical Dataset │
                    │      (Ausgrid)      │
                    └──────────┬──────────┘
                               │
               ┌───────────────┴───────────────┐
               ↓                               ↓
       ┌────────────────┐              ┌────────────────┐
       │ Load Processing│              │ Solar Processing│
       └───────┬────────┘              └───────┬────────┘
               ↓                               ↓
       ┌────────────────┐              ┌────────────────┐
       │ Weather Data   │              │ Weather Data   │
       └───────┬────────┘              └───────┬────────┘
               ↓                               ↓
       ┌────────────────┐              ┌────────────────┐
       │ XGBoost Load   │              │ XGBoost Solar  │
       │ Forecast Model │              │ Forecast Model │
       └───────┬────────┘              └───────┬────────┘
               │                               │
               └──────────────┬────────────────┘
                              ↓
                    ┌───────────────────┐
                    │   DQN Agent       │
                    │                   │
                    │ Load Forecast     │
                    │ Solar Forecast    │
                    │ Battery SOC       │
                    │ Time Information   │
                    └─────────┬─────────┘
                              ↓
                     ┌─────────────────┐
                     │ Battery Action  │
                     │                 │
                     │ IDLE            │
                     │ CHARGE          │
                     │ DISCHARGE       │
                     └────────┬────────┘
                              ↓
                    ┌───────────────────┐
                    │ Energy Simulator  │
                    └─────────┬─────────┘
                              ↓
               ┌──────────────┼──────────────┐
               ↓              ↓              ↓
           Grid Import   Battery SOC    Curtailment
                              │
                              ↓
                         Dashboard
```

---

# 📊 Dataset

The project uses the **Ausgrid Solar Home Electricity Dataset (2012–2013)**.

The dataset contains half-hourly electricity information for residential customers in Australia.

The following consumption categories are available:

| Category | Description |
|---|---|
| GC | Gross Consumption |
| GG | Gross Generation |
| CL | Controlled Load |

For this project:

### Load Forecasting

The load forecasting model uses:

```text
GC + CL = Total Load Demand
```

### Solar Forecasting

Solar forecasting uses:

```text
GG = Solar Generation
```

The project aggregates the household-level data into a grid/community-level 30-minute time series.

---

# 🌦️ Weather Data

Historical weather information was incorporated into the forecasting models.

The weather features used are:

- Temperature
- Relative humidity
- Cloud cover
- Precipitation
- Wind speed

Weather information was aligned with the electricity dataset using postcode-based geographical locations.

The final weather data was aggregated to the same 30-minute timeline as the energy data.

---

# 🔧 Data Processing

The raw electricity dataset was processed before model training.

### Load Processing

The following steps were performed:

1. Filter `GC` and `CL`.
2. Remove the incomplete customer record used during preprocessing.
3. Convert the 30-minute columns into a timestamp-based format.
4. Combine GC and CL consumption.
5. Aggregate customers spatially.
6. Investigate zero-demand periods.
7. Perform household-level 3σ spike analysis.
8. Retain potentially legitimate high-demand observations.
9. Create time-based features.
10. Create lag features.
11. Create past-only rolling statistics.
12. Create a next-step forecasting target.

### Solar Processing

Solar generation was processed by:

1. Selecting `GG`.
2. Aggregating solar generation across solar households.
3. Creating 30-minute timestamps.
4. Creating calendar features.
5. Creating cyclical time features.
6. Creating lag features.
7. Creating past-only rolling features.
8. Adding weather information.

---

# 🧮 Feature Engineering

The forecasting models use temporal and historical features.

Examples include:

```text
hour
minute
day_of_week
day_of_month
month
day_of_year
is_weekend
hour_sin
hour_cos
day_of_year_sin
day_of_year_cos
```

Historical features include:

```text
lag_1
lag_2
lag_4
lag_48
lag_96
lag_336
```

Rolling features include:

```text
rolling_mean_2
rolling_mean_4
rolling_mean_48
rolling_std_48
```

The lag values correspond to the 30-minute sampling frequency.

For example:

```text
lag_1   → previous 30-minute interval
lag_2   → previous 1 hour
lag_48  → previous 24 hours
lag_96  → previous 48 hours
lag_336 → previous 7 days
```

---

# 🤖 Forecasting Models

## 1. Solar Forecasting

An **XGBoost regression model** was used for solar generation forecasting.

The original solar model achieved:

| Metric | Result |
|---|---:|
| MAE | 3.3975 kWh |
| RMSE | 7.1927 kWh |
| R² | 0.9645 |

After incorporating weather information:

| Metric | Weather-Enhanced |
|---|---:|
| MAE | **1.8431 kWh** |
| RMSE | **4.0125 kWh** |
| R² | **0.9889** |

The weather-enhanced model therefore showed lower test-set MAE and RMSE and higher R² on the evaluated historical period.

---

# ⚡ Load Forecasting

The load forecasting model also uses **XGBoost regression**.

### Original Model

| Metric | Result |
|---|---:|
| MAE | 7.5718 kWh |
| RMSE | 9.9708 kWh |
| R² | 0.9463 |

### Weather-Enhanced Model

| Metric | Result |
|---|---:|
| MAE | **5.7719 kWh** |
| RMSE | **7.5109 kWh** |
| R² | **0.9695** |

The weather-enhanced model showed improved performance on the evaluated historical test period.

---

# 🧠 Reinforcement Learning

After generating load and solar forecasts, the system passes the predicted values to a **Deep Q-Network (DQN)** agent.

The DQN acts as the energy-management component of the system.

## State

The DQN state contains:

```text
[
    Predicted Load,
    Predicted Solar,
    Battery SOC,
    Hour Sin,
    Hour Cos
]
```

The time features allow the agent to distinguish between different periods of the day.

---

# 🎮 Action Space

The DQN has three possible actions:

| Action | Meaning |
|---|---|
| `0` | IDLE |
| `1` | CHARGE |
| `2` | DISCHARGE |

The agent selects one action at every 30-minute timestep.

---

# 🔋 Battery Model

The battery is a **simulation assumption** for the prototype and is not directly present in the Ausgrid dataset.

Prototype parameters:

```text
Battery Capacity       = 100 kWh
Initial SOC            = 50 kWh
Minimum SOC            = 10 kWh
Maximum SOC            = 100 kWh

Maximum Charge         = 20 kWh / timestep
Maximum Discharge      = 20 kWh / timestep

Charge Efficiency      = 95%
Discharge Efficiency   = 95%
```

The battery is intended to represent a community/microgrid energy storage system.

---

# ⚡ Energy Management Logic

The simulator prioritizes solar generation for serving demand.

### Case 1 — Solar < Load

```text
Solar → Load

Remaining Load
       ↓
Battery
       ↓
Grid
```

If the battery can provide energy, it is discharged according to the selected action.

Any remaining demand is supplied by the grid.

---

### Case 2 — Solar > Load

```text
Solar
 ├──→ Load
 ├──→ Battery
 └──→ Curtailment
```

Solar surplus can charge the battery.

If the battery cannot accept additional energy, the remaining solar is curtailed.

---

# 🏆 Reward Function

The DQN uses a reward designed to penalize:

- Grid energy import
- Excessive battery movement
- Solar curtailment

The prototype reward is:

```text
Reward =
    - Grid Import
    - 0.05 × Battery Movement
    - 0.10 × Solar Curtailment
```

The goal is therefore to reduce unnecessary grid dependency while avoiding excessive battery cycling and renewable-energy curtailment.

---

# 📈 RL Evaluation

Three strategies were compared:

1. No Battery
2. Rule-Based Controller
3. DQN Controller

### Results

| Strategy | Grid Import (kWh) | Solar Curtailment (kWh) | Battery Movement (kWh) |
|---|---:|---:|---:|
| No Battery | 270,871.35 | 8,297.40 | 0 |
| Rule-Based | 268,593.60 | 5,815.68 | 4,759.48 |
| DQN | **268,556.39** | 5,950.10 | 4,503.74 |

### DQN vs No-Battery Case

The logged DQN simulation produced approximately:

```text
Grid Import Reduction       ≈ 0.85%
Solar Curtailment Reduction ≈ 28.29%
```

The DQN simulation ended with:

```text
Final Battery SOC = 10 kWh
```

DQN action distribution:

```text
IDLE        1027
CHARGE       809
DISCHARGE    740
```

---

# 📊 Dashboard

A prototype dashboard was created using Plotly and Jupyter widgets.

The dashboard provides:

### Current Grid State

```text
Predicted Load
Predicted Solar
Battery SOC
Grid Import
Battery Charge
Battery Discharge
Solar Curtailment
```

### DQN Decision

The dashboard displays the action selected by the agent:

```text
IDLE
CHARGE
DISCHARGE
```

### Visualizations

The prototype includes:

- Load vs Solar Forecast
- Battery State of Charge
- Current Energy Flow
- Strategy Comparison

A timestamp slider allows the user to move through the simulated period and inspect the system state.

---

# 🗂️ Project Structure

A suggested repository structure is:

```text
AI-Smart-Grid/
│
├── notebooks/
│   ├── 01_data_processing.ipynb
│   ├── 02_solar_forecasting.ipynb
│   ├── 03_load_forecasting.ipynb
│   ├── 04_weather_integration.ipynb
│   ├── 05_weather_enhanced_forecasting.ipynb
│   ├── 06_dqn_energy_management.ipynb
│   └── 07_dashboard.ipynb
│
├── models/
│   ├── solar_weather_xgboost_model.pkl
│   ├── load_weather_xgboost_model.pkl
│   ├── solar_weather_feature_columns.pkl
│   └── load_weather_feature_columns.pkl
│
├── data/
│   ├── solar_forecasting_weather_processed.csv
│   ├── load_forecasting_weather_processed.csv
│   ├── solar_weather_predictions.csv
│   └── load_weather_predictions.csv
│
├── results/
│   ├── dqn_energy_management_results.csv
│   └── strategy_comparison.csv
│
├── dashboard/
│   └── dashboard.py
│
├── requirements.txt
│
└── README.md
```

> Large raw datasets and generated model files should generally not be committed directly to GitHub if they exceed repository/file-size limits. Dataset download instructions can be provided instead.

---

# 🛠️ Technology Stack

### Programming

- Python

### Machine Learning

- XGBoost
- Scikit-learn
- NumPy
- Pandas

### Reinforcement Learning

- PyTorch
- Deep Q-Network (DQN)

### Data Visualization

- Plotly
- Matplotlib

### Dashboard Prototype

- Jupyter Widgets
- Plotly

### Data Source

- Ausgrid Solar Home Electricity Dataset
- Historical weather data

---

# 📦 Installation

Clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/AI-Smart-Grid.git

cd AI-Smart-Grid
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Recommended Python packages:

```text
numpy
pandas
scikit-learn
xgboost
torch
joblib
plotly
ipywidgets
matplotlib
```

---

# 🚀 Running the Project

The project is organized as a sequential pipeline.

### Step 1 — Data Processing

Prepare the electricity and weather datasets.

### Step 2 — Forecasting

Train or load:

```text
Solar XGBoost Model
Load XGBoost Model
```

### Step 3 — Generate Forecasts

Generate:

```text
Predicted Solar Generation
Predicted Electricity Load
```

### Step 4 — Reinforcement Learning

Feed the forecasts into the DQN environment.

The DQN selects:

```text
IDLE
CHARGE
DISCHARGE
```

### Step 5 — Energy Simulation

The environment calculates:

```text
Grid Import
Battery SOC
Battery Charge
Battery Discharge
Solar Curtailment
Reward
```

### Step 6 — Dashboard

Launch the visualization/dashboard notebook to inspect the simulated grid.

---

# 🔬 Experimental Setup

The forecasting datasets use a chronological split rather than random shuffling.

The data was divided approximately into:

```text
70% Training
15% Validation
15% Testing
```

This preserves the temporal nature of electricity forecasting.

The RL prototype uses the overlapping historical forecast period for its simulation.

---

# ⚠️ Limitations

This project is currently a **prototype / proof of concept** rather than a production grid-control system.

Important limitations include:

### 1. Simulated Battery

The battery is not part of the original Ausgrid dataset.

Its capacity, SOC limits, charge/discharge limits and efficiencies are prototype assumptions.

### 2. Historical Simulation

The DQN is evaluated on a historical forecast sequence rather than a live electrical grid.

### 3. RL Evaluation

The current RL experiment is intended as a proof of concept.

A more rigorous research evaluation would use separate temporal RL training and evaluation periods and multiple independent episodes.

### 4. No Real-Time Grid Control

The system does not directly control physical grid infrastructure.

It produces simulated energy-management decisions.

### 5. No Grid Export

The current prototype does not model exporting excess solar energy to the grid.

Excess solar that cannot be consumed or stored is treated as curtailed energy.

### 6. Carbon Intensity

The current prototype does not use a verified time-varying electricity-grid carbon-intensity dataset in the reward function.

Therefore, carbon optimization is part of the future extension rather than a demonstrated result of the current prototype.

---

# 🔮 Future Scope

The system can be extended into a more complete smart-grid platform.

## 1. Real-Time Data

Integrate:

- Smart meters
- IoT sensors
- Real-time weather APIs
- Renewable generation sensors

---

## 2. Advanced Battery Management

Add:

- Battery degradation
- State of Health
- Battery temperature
- Dynamic charging limits
- Battery replacement cost

---

## 3. Carbon-Aware Optimization

Integrate real-time grid carbon intensity.

The RL agent could optimize:

```text
Cost
+
Grid Import
+
Carbon Emissions
+
Battery Degradation
+
Solar Curtailment
```

---

## 4. Electricity Price Optimization

Introduce dynamic electricity prices so that the agent can learn when to:

```text
Charge
Discharge
Wait
```

based on both demand and electricity prices.

---

## 5. Peer-to-Peer Energy Trading

Future versions could allow households with excess renewable generation to trade energy with nearby consumers.

---

## 6. Grid-Level Deployment

The architecture could eventually be extended from a simulated community battery to:

```text
Homes
   ↓
Microgrid
   ↓
Distribution Network
   ↓
Utility Grid
```

---

## 7. More Advanced RL

Future experiments could compare:

- DQN
- Double DQN
- Dueling DQN
- PPO
- SAC

---

# 🎯 Potential Users

A production version could potentially support:

### Electricity Distribution Utilities

For:

- Demand forecasting
- Renewable integration
- Peak management
- Battery scheduling

### Microgrid Operators

For:

- Local energy balancing
- Battery management
- Renewable utilization

### Commercial Buildings

For:

- Demand management
- Solar utilization
- Energy-cost optimization

### Renewable Energy Operators

For:

- Solar generation forecasting
- Storage scheduling
- Curtailment reduction

### Energy Management Companies

For:

- AI-based energy optimization
- Smart building management
- Distributed energy resource management

---

# 🌍 Project Impact

The proposed system demonstrates how multiple AI components can work together rather than treating forecasting and energy management as separate problems.

The central idea is:

```text
Predict the future
       ↓
Understand the future grid state
       ↓
Take an intelligent action
       ↓
Simulate the result
       ↓
Optimize energy management
```

This creates a foundation for an AI-assisted energy management platform capable of integrating renewable generation, demand forecasting and energy storage.

---

# 👨‍💻 Project Status

| Component | Status |
|---|---|
| Dataset Processing | ✅ Completed |
| Load Forecasting | ✅ Completed |
| Solar Forecasting | ✅ Completed |
| Weather Integration | ✅ Completed |
| Weather-Enhanced Models | ✅ Completed |
| Battery Simulation | ✅ Completed |
| Rule-Based Controller | ✅ Completed |
| DQN Agent | ✅ Completed |
| Strategy Comparison | ✅ Completed |
| Interactive Prototype Dashboard | ✅ Completed |
| Real-Time Grid Integration | 🔄 Future Work |
| Carbon Optimization | 🔄 Future Work |
| P2P Energy Trading | 🔄 Future Work |

---

# 📜 Disclaimer

This repository contains a research/prototype implementation.

The simulated battery parameters and energy-management environment are assumptions created for experimentation. The results should not be interpreted as guaranteed savings or performance for a real electricity grid.

The system is not intended to directly control real electrical infrastructure.

---

# ⭐ Key Takeaway

This project demonstrates a complete AI pipeline for smart-grid energy management:

```text
Electricity Data
       +
Weather Data
       ↓
Demand Forecasting
       +
Solar Forecasting
       ↓
Future Grid State
       ↓
Deep Reinforcement Learning
       ↓
Battery Management
       ↓
Reduced Grid Dependence
       +
Reduced Solar Curtailment
```

The prototype combines **machine learning forecasting + reinforcement learning + energy simulation + visualization** into a single smart-grid energy management workflow.
```


