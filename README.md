# SkillPulse

**Real-Time Job Market & Skills-Demand Tracker** — an end-to-end ML pipeline that scrapes live job postings, extracts in-demand skills, and predicts salaries.

Built as a portfolio project to practice the full ML lifecycle: data ingestion → cleaning → feature engineering → model training → serving → monitoring — not just a notebook that stops at a metric.

---

## What it does

- Pulls live job postings from the **Adzuna API** across India, UK, and US (~6,000 postings collected)
- Stores everything in a normalized **MySQL** schema (5 tables) — chosen over flat files/pandas-only storage to keep raw data integrity separate from processed data
- Extracts **46 technical skills** from job descriptions via a regex-based pipeline, mapped many-to-many against postings
- Trains an **XGBoost** salary predictor (Optuna hyperparameter tuning, 30 trials, 5-fold cross-validation) — ~$31K USD mean absolute error on held-out data
- Serves predictions through a **FastAPI** backend (`/health`, `/skills/top`, `/predict`)
- Visualizes everything in a 4-view **Streamlit** dashboard: market overview, skill intelligence, salary predictor, model diagnostics
- Monitors for data drift with **Evidently AI**, auto-retraining when drift crosses a 30% threshold
- Fully containerized with **Docker Compose**

## Architecture

```
Adzuna API ──▶ MySQL (raw_postings) ──▶ Cleaning & Dedup ──▶ MySQL (processed_postings)
                                              │
                                              ▼
                                    Skill Extraction (regex)
                                              │
                                              ▼
                              Feature Matrix (46 skills + country flags)
                                              │
                                              ▼
                          XGBoost + Optuna ──▶ model_runs (MySQL)
                                              │
                              ┌───────────────┴───────────────┐
                              ▼                                ▼
                        FastAPI (/predict)              Streamlit Dashboard
                              │
                              ▼
                  Evidently Drift Monitor ──▶ Auto-Retrain (≥30% drift)
```

**Why MySQL-first instead of pandas/flat files:** raw scraped data and processed/cleaned data live in separate tables (`raw_postings` vs `processed_postings`), so cleaning logic can be re-run or changed without re-scraping or losing the original data. This was a deliberate trade-off for data integrity over quick iteration speed.

## Tech stack

`Python` · `MySQL` · `XGBoost` · `Optuna` · `FastAPI` · `Streamlit` · `Evidently AI` · `Docker Compose` · `Pandas`

## Key results & trade-offs

- **Deduplication:** fingerprint-based matching cut duplicate postings by ~62%
- **Salary model:** XGBoost, tuned via Optuna — Test MAE ~0.31 (log scale), ≈$31K USD average error
- **Documented trade-off:** India was excluded from salary model training due to data quality issues in that subset, rather than including it and silently degrading model performance. This is a decision, not an oversight — flagged here and in the code.

## Project status

This project is under active development. Rather than overstate what's done, here's the honest breakdown:

### ✅ Done
- Adzuna scraping across 3 countries, ~6,000 postings collected
- MySQL schema (5 tables), regex skill extraction (46 skills)
- Cleaning, deduplication, EDA, salary standardization (USD)
- XGBoost model with Optuna tuning + 5-fold CV, model runs logged to MySQL
- FastAPI serving layer (health, top skills, predict)
- Streamlit dashboard (4 views)
- Evidently AI drift monitoring + conditional auto-retrain
- Docker Compose for full local stack

### 🟡 Partially done
- Skill extraction is regex-only — spaCy/LLM-based extraction not yet implemented
- Baseline model is default XGBoost, not a Linear Regression baseline for comparison
- Train/test split is random, not time-based (older postings → train, newest → test)
- Feature set is skills + country only — no seniority, location granularity, or posting recency yet
- Streamlit reads the model/DB directly rather than calling the FastAPI backend
- Deployed locally via Docker only — not yet on a public cloud URL

### ❌ Not yet built
- RemoteOK API integration (Adzuna only, currently)
- Weekly automated re-scraping / scheduled retraining (Airflow or cron)
- "Skills to learn next" recommender in the dashboard
- Public live deployment (Render / Hugging Face Spaces)
- LLM vs. regex skill-extraction benchmark

## Roadmap

**Next up:**
1. Linear Regression baseline + documented % improvement from XGBoost
2. Time-based train/test split
3. Seniority and recency features
4. "Skills to learn next" recommender
5. Public deployment

## Running locally

```bash
git clone https://github.com/Harshdeepsingh120/skillpulse.git
cd skillpulse
docker-compose up
```

*(Add exact env vars / API key setup instructions here once finalized.)*

---

Built by [Harshdeep Singh](https://linkedin.com/in/harshdeepsingh12) — CS undergrad, LPU.
