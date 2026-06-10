# SalesIQ – LLM-Powered Business Analytics Engine

An end-to-end sales forecasting and analytics system that combines a **3-stage stacked ML model** with an **LLM-powered conversational assistant** and an **interactive Power BI dashboard** — built on real-world product-level quarterly sales data spanning FY23–FY26.

---

## Project Overview

SalesIQ addresses a core business challenge: generating accurate next-quarter sales forecasts and making those forecasts explainable to non-technical stakeholders. The system has three integrated layers:

1. **ML Forecasting Engine** — predicts FY26 Q2 sales for 30 products using a stacked ensemble model
2. **Power BI Dashboard** — visualizes KPIs, trends, lifecycle analysis, and forecast comparisons
3. **LLM Business Analyst** — a conversational AI assistant that reads dashboard data and answers natural-language business queries, and generates structured reports on demand

---

## Dataset

- **Source:** Product-level quarterly bookings data (CFL External Data Pack)
- **Scope:** 30 products × 12 quarters (FY23 Q2 → FY26 Q1)
- **Features per product:** Cost Rank, Product Name, Lifecycle Stage (Sustaining / Decline / NPI-Ramp), Quarterly Sales
- **Total supervised training samples:** 240 (walk-forward windowed from 30 products × 8 time steps)

**Key Business KPIs:**
| Metric | Value |
|---|---|
| Total Historical Sales | ~3M units |
| FY26 Q1 Actual Sales | 196K units |
| ML Forecast (FY26 Q2) | ~3M units |
| Demand Planner Forecast (Q2) | 2.46M units |
| Avg Forecast Deviation | ~155 units |
| Max Forecast Deviation | ~10.29K units |

---

## Model Architecture

A **3-stage stacked ensemble** was designed to progressively correct prediction errors:
Stage 1: Random Forest (Primary Forecaster)
↓ computes residuals
Stage 2: XGBoost (Residual Corrector — learns what RF got wrong)
↓ RF prediction + XGB correction
Stage 3: Ridge Meta-Learner (Blends RF & XGBoost outputs)
↓ Final Forecast

---

### Feature Engineering (11 features per time step)
| Feature | Description |
|---|---|
| `lag1`, `lag2`, `lag3`, `lag4` | Sales from previous 1–4 quarters |
| `roll4_mean` | Rolling 4-quarter average |
| `roll4_std` | Rolling 4-quarter standard deviation (volatility) |
| `yoy_growth` | Year-over-year growth rate (FY26 Q1 vs FY25 Q1) |
| `t_idx` | Time index (captures trend position) |
| `trend_slope` | OLS regression slope per product |
| `roll2_mean` | 2-quarter rolling mean (recent momentum) |
| `roll_ratio` | Current vs recent rolling average (over-performance signal) |

### Model Performance
| Stage | Model | MAE | RMSE | R² |
|---|---|---|---|---|
| Stage 1 | Random Forest | — | — | ~0.97+ |
| Stage 2 | RF + XGBoost Residual | — | — | Improved |
| Stage 3 | Full Stack + Ridge Meta | — | — | Best |
| Cross-Val | RF (5-Fold CV) | Stable | — | Mean ± Std reported |

> Actual metric values are printed in notebook output on run.

---

## Power BI Dashboard

The dashboard is structured across 2 pages:

**Page 1 — Sales Performance**
- **KPI Cards:** Total sales, FY26 Q1 actuals, forecast comparison
- **Quarterly Sales Trend (Line Chart):** FY23–FY26 trend showing gradual decline with sharper drop near FY26
- **Sales by Product Lifecycle (Donut Chart):** ~80% from Sustaining, remainder from Decline and NPI-Ramp
- **Forecast Comparison (Bar Chart):** ML forecast vs Demand Planner forecast per product

**Page 2 — Model Insights**
- **Forecast Deviation (Bar Chart):** Difference between ML and Demand Planner forecasts — identifies products with highest planning vs model mismatch
- **Projected Growth Q2 vs Q1:** Product-level expected growth/decline
- **KPI Metrics:** Avg deviation ~155, Max deviation ~10.29K

---

## LLM Business Analyst

A conversational agent powered by **LLaMA 3 8B (via OpenRouter)** that is pre-loaded with full dashboard context and answers questions like a business analyst.

**Capabilities:**
- Explain any dashboard component with specific numbers
- Answer dataset and trend questions
- Identify anomalies and forecast mismatches
- Generate three types of downloadable reports:
  - `"sales report"` → Sales performance report
  - `"technical report"` → Model and methodology report
  - `"general report"` → Full business summary report

**How it works:**
```python
# Context-injected system prompt with real KPIs, chart descriptions, and business rules
# Multi-turn conversation memory across the session
# Keyword-based report type detection
# Model: meta-llama/llama-3-8b-instruct via OpenRouter API
```

---

## Tech Stack

| Layer | Tools |
|---|---|
| Data Cleaning & EDA | Python, pandas, NumPy |
| Feature Engineering | NumPy (vectorised ops, broadcasting, OLS via `lstsq`) |
| ML Modeling | scikit-learn (RandomForest, Ridge, KFold CV), XGBoost |
| Visualization | matplotlib, seaborn (heatmaps, violin plots, small multiples) |
| Dashboard | Power BI |
| LLM Integration | LLaMA 3 8B via OpenRouter API |
| Notebook Environment | Google Colab |

---

## Project Structure
SalesIQ/
├── notebooks/
│   ├── 01_eda_and_ml_forecasting.ipynb   # Data cleaning, EDA, feature engineering, stacked model, FY26 Q2 predictions
│   └── 02_llm_chatbot.ipynb              # LLM assistant with multi-turn conversation and report generation
├── dashboard/
│   └── SalesIQ_Dashboard.pbix            # Power BI dashboard (2-page interactive report)
├── outputs/
│   └── CFL2026_FY26Q2_Predictions.csv    # Final forecasts for all 30 products
├── requirements.txt
└── README.md
---

## Setup & Usage

### 1. Clone the repository
```bash
git clone https://github.com/Mitu0601/SalesIQ.git
cd SalesIQ
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Run ML Forecasting Notebook
Open `notebooks/01_eda_and_ml_forecasting.ipynb` in Jupyter or Google Colab. Add your dataset file (`CFL_External_Data_Pack_Phase1.xlsx`) to the working directory and run all cells.

### 4. Run the LLM Chatbot
Open `notebooks/02_llm_chatbot.ipynb`. Add your OpenRouter API key and run all cells. Interact via terminal:
You: Explain the sales trend
Agent: Based on the Quarterly Sales Trend chart, total sales show a gradual decline from FY23 to FY26...
You: Generate a sales report
Agent: [Structured report with KPIs, trends, and insights]

----
### 5. View Dashboard
Open `dashboard/SalesIQ_Dashboard.pbix` in Power BI Desktop.

---

## requirements.txt
numpy
pandas
matplotlib
seaborn
scikit-learn
xgboost
openpyxl
requests
jupyter

---

## Key Insights

- **Sustaining products dominate (~80% of sales)** — business is heavily dependent on a mature product base
- **Overall trend shows decline** — sharper drop near FY26 signals urgency for NPI-Ramp acceleration
- **ML model forecasts ~3M vs Demand Planner's 2.46M** — gap of ~540K units suggests model detects recovery signals the planning team may be missing
- **High-volume products drive the largest forecast deviations** — model improvements should prioritize top-10 ranked products

---

