# Wealthwise — AI-Assisted Retirement Planning Platform

> **Plan Smart. Retire Confident.**

**WealthWise** is a full-stack, AI-powered retirement planning platform built for the Indian financial context. It replaces generic actuarial averages with personalised, data-driven predictions: a machine-learning model estimates your individual life expectancy from health and lifestyle data, a time-series model forecasts inflation dynamically from real CPI data, and a RAG-powered chatbot lets you interrogate your own pension documents in plain language.

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [Objectives](#objectives)
- [System Architecture](#system-architecture)
- [Pipeline & Workflow](#pipeline--workflow)
- [Tech Stack](#tech-stack)
- [Dataset](#dataset)
- [ML Models & Evaluation](#ml-models--evaluation)
- [RAG Pipeline Evaluation](#rag-pipeline-evaluation)
- [Project Structure](#project-structure)
- [Environment Variables](#environment-variables)
- [How to Run](#how-to-run)
- [API Reference](#api-reference)
- [Security](#security)
- [Future Scope](#future-scope)

---

## Problem Statement

Traditional retirement planning tools suffer from three critical shortcomings:

1. **Static life expectancy** — most tools assume a fixed lifespan of ~85 years regardless of individual health, BMI, smoking history, or chronic conditions. This leads to systematic under-saving or over-saving.

2. **Fixed inflation assumptions** — a static 6% annual rate cannot capture the volatile, cyclically driven nature of India's Consumer Price Index, which is sensitive to monsoon cycles, commodity shocks, and RBI monetary policy.

3. **Inaccessible pension documents** — policy documents, NPS/EPF statements, and insurance prospectuses are dense with legal jargon that non-expert users cannot easily parse, leading to uninformed decisions or expensive reliance on advisors.

Together, these gaps expose a large share of India's working population to the risk of outliving their savings.

---

## Objectives

1. Build an **XGBoost regression model** that predicts personalised life expectancy from health and lifestyle biomarkers (BMI, exercise, diet, smoking, alcohol, hypertension, diabetes, heart disease, cholesterol, gender), eliminating population-average assumptions.

2. Implement an **ARMA time-series model** trained on ten years of Indian CPI data (2013–2023) to produce dynamic, data-backed inflation forecasts that feed directly into corpus projections.

3. Develop a **RAG chatbot** (Google Gemini + Pinecone) that allows users to upload pension PDFs and receive precise, cited answers grounded strictly in their own documents — with no model hallucination on financial facts.

4. Build an **interactive React dashboard** that unifies all predictions, financial scenario modelling, portfolio risk scoring, and document Q&A into a single cohesive user experience.

---

## System Architecture


wealthwise follows a distributed three-tier architecture:
<img width="720" height="376" alt="image" src="https://github.com/user-attachments/assets/88b302d9-6671-4510-8f12-601277bc54e5" />

```
User (Browser)
     │  HTTPS
     ▼
Frontend  ──────────────────────────────────────────────────────────
React 19 + Vite + Redux Toolkit + Tailwind CSS + Chart.js/Recharts
     │  REST (JWT)
     ▼
Backend  ────────────────────────────────────────────────────────────
Django REST Framework
 • JWT authentication & Google OAuth
 • User profiles, financial data, chat history (SQLite → PostgreSQL)
 • Financial computation layer (corpus projection, break-even, tax)
 • Intent classification → routes to RAG or deterministic calculation
     │  Internal HTTP
     ├──────────────────────────────────┐
     ▼                                  ▼
ML Service (FastAPI)            LLM / Vector Services
 • XGBoost life expectancy       • Google Gemini (Pro/Flash)
 • ARMA inflation forecasting    • Pinecone vector database
 • scikit-learn pipelines        • LangChain PDF ingestion
     │                                  │
     └──── Zerodha KiteConnect ─────────┘
           yfinance (historical prices)
```

---

## Pipeline & Workflow
<img width="670" height="219" alt="image" src="https://github.com/user-attachments/assets/5cb1d510-36c0-4a32-82c8-024dac4919df" />


### Onboarding Flow
1. User registers / logs in (email+password or Google OAuth → JWT issued).
2. Guided five-step onboarding collects: personal info → income & employment → retirement preferences → health data → risk tolerance.
3. Each step validates in real-time, persists to Django backend, and updates the Redux store.

### ML Prediction Flow
1. Health and financial inputs submitted from the frontend.
2. Django stores raw data and forwards a JSON payload to the FastAPI prediction service.
3. FastAPI runs the XGBoost pipeline (preprocessing → encoding → inference) and returns predicted life expectancy in years.
4. FastAPI also runs the ARMA model and returns a 3-year inflation forecast with 95% confidence intervals.
5. Django uses both predictions in the financial computation layer to calculate genuine real return rates: `(1 + Nominal Rate) / (1 + Inflation Rate) - 1`.

### RAG Chatbot Flow
1. User sends a text query or uploads a PDF.
2. Gemini Flash classifies intent into one of four classes: generic pension query | document Q&A | financial data extraction | educational query.
3. For document queries: the query is embedded with the Gemini embed model and a vector similarity search is run against the user's private Pinecone namespace.
4. Top-k relevant document chunks are retrieved and appended to the prompt.
5. Gemini Pro generates a response grounded in the retrieved context only.
6. Messages, timestamps, and sending users are persisted to the database for conversation continuity.

### Dashboard & Analytics
The analytics dashboard exposes 13 visualisations including:
- Retirement corpus projections (base / aggressive / conservative scenarios)
- Break-even analysis: lump-sum vs. annuity payout
- Payout comparison with tax impact
- Inflation forecast chart with confidence intervals
- Optimal retirement age heatmap
- Real-time Zerodha portfolio holdings, P&L, and stock trend charts

---

## Tech Stack

| Layer | Technology | Role |
|---|---|---|
| Frontend | React 19 + Vite | Component-based SPA; fast HMR build |
| Frontend | Redux Toolkit | Global state: session, onboarding, risk data |
| Frontend | React Router | Client-side routing across multi-step flows |
| Frontend | Tailwind CSS | Responsive utility-first styling |
| Frontend | Chart.js / Recharts | Retirement corpus, inflation, and portfolio charts |
| Backend | Django REST Framework | API orchestration, auth, user management |
| Backend | JWT Authentication | Stateless secure session management |
| Backend | django-allauth | Google OAuth social login |
| Backend | SQLite / PostgreSQL | Relational persistence (dev / prod) |
| ML Service | FastAPI | High-performance async microservice for ML inference |
| ML Service | XGBoost | Life expectancy regression model |
| ML Service | StatsModels (ARMA) | Indian CPI inflation time-series forecasting |
| ML Service | scikit-learn | Preprocessing pipelines and evaluation utilities |
| AI / Chatbot | Google Gemini Pro/Flash | LLM for intent classification and response generation |
| AI / Chatbot | Pinecone | Vector database for RAG document retrieval |
| AI / Chatbot | LangChain | Framework connecting LLMs to PDF data sources |
| Integrations | Zerodha KiteConnect | Live portfolio holdings and mutual fund data |
| Integrations | yfinance | Historical stock price retrieval |
| DevOps | Git / GitHub | Version control |
| DevOps | Postman | API testing and documentation |

---

## Dataset

Two datasets were used, both sourced from Kaggle:

| Dataset | Source | Purpose | Key Features |
|---|---|---|---|
| Life Expectancy Based on Individual Lifestyle | Kaggle | Train the XGBoost life expectancy model | BMI, physical activity, diet quality, smoking status, alcohol consumption, hypertension, diabetes, heart disease, gender, age |
| Consumer Price Index (CPI) 2013–2023 | Kaggle | Train and evaluate the ARMA inflation model | Monthly Indian CPI data (132 observations, Jan 2013–Dec 2023) |

**Pre-processing notes:**
- Life expectancy dataset: categorical features encoded via ordinal or one-hot encoding inside a scikit-learn `Pipeline` object. Top features by permutation importance: smoking status, physical activity, heart disease history, gender.
- CPI dataset: Augmented Dickey-Fuller test confirmed non-stationarity; first-differencing of the log-transformed series applied before ARMA fitting.

---

## ML Models
<img width="716" height="419" alt="image" src="https://github.com/user-attachments/assets/3a3ab737-648c-47b1-91ed-5796ca1865c3" />


### XGBoost Life Expectancy Model

Hyperparameters selected via cross-validation grid search optimising RMSE on a held-out test set. The trained pipeline is serialised (pickled) and loaded at FastAPI startup for sub-second inference.\


### ARMA Inflation Forecasting Model

Model order selected by minimising AIC, BIC, and HQIC across a grid of (p, q) values. Selected model: AIC = 342.60, BIC = 360.20, HQIC = 349.75. Residual autocorrelation confirms no exploitable serial structure remains.


The ARMA forecast feeds a 3-year ahead CPI projection (with 95% confidence intervals) directly into the dashboard's corpus projection engine.
<img width="444" height="188" alt="image" src="https://github.com/user-attachments/assets/97499b09-ddb3-4101-ba5e-a22907d127a5" />

---

## RAG Pipeline Evaluation
<img width="295" height="185" alt="image" src="https://github.com/user-attachments/assets/4f693cd8-7b43-48d0-8283-dd81e75b6048" />


Evaluated using the **RAGAS** framework (LLM-as-a-jurist methodology), which independently assesses four dimensions of retrieval-augmented generation quality.

| Metric | Score | Interpretation |
|---|---|---|
| Context Precision | **1.0000** | Every retrieved chunk was relevant — zero irrelevant retrieval |
| Context Recall | 0.9464 | System retrieved 94.6% of all reference-relevant content |
| Faithfulness | **0.9107** | 91% of generated statements are grounded in retrieved context; minimal hallucination |
| Answer Relevancy | 0.7784 | 78% of answer content directly addresses the question; identified for future prompt tuning |

The perfect Context Precision score reflects the Pinecone user-namespace isolation strategy: the retrieval search is scoped strictly to the individual user's uploaded PDFs, eliminating cross-user or parametric-knowledge contamination.

---

## Project Structure

```
wealthwise/
├── frontend/                  # React 19 + Vite SPA
│   ├── src/
│   │   ├── components/        # Reusable UI components
│   │   ├── pages/             # Dashboard, Chat, Learn, Finance
│   │   ├── store/             # Redux Toolkit slices
│   │   └── utils/             # API helpers, formatters
│   ├── package.json
│   └── vite.config.js
│
├── backend/                   # Django REST Framework
│   ├── wealthwise/            # Django project settings
│   ├── users/                 # Auth, profiles, onboarding
│   ├── chatbot/               # RAG pipeline, intent classification
│   ├── financial/             # Corpus projections, risk scoring, Zerodha
│   ├── requirements.txt
│   └── requirements_rag.txt   # RAG-specific dependencies
│
├── predictions/               # FastAPI ML microservice
│   ├── app.py                 # FastAPI entrypoint
│   ├── life_expectancy.py     # XGBoost pipeline
│   ├── inflation.py           # ARMA forecasting model
│   └── requirements.txt
│
├── render.yaml                # Render.com deployment config
├── .gitignore
└── README.md
```

---

## Environment Variables

Create a `.env` file in each service directory. **Never commit secrets to version control.**

### `backend/.env`

```env
SECRET_KEY=your-django-secret-key
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Database (PostgreSQL for production)
DATABASE_URL=postgres://user:password@host:5432/dbname

# Google OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# Pinecone (RAG)
PINECONE_API_KEY=your-pinecone-api-key
PINECONE_ENVIRONMENT=your-pinecone-environment
PINECONE_INDEX_NAME=your-index-name

# Google Gemini
GOOGLE_API_KEY=your-gemini-api-key

# Zerodha KiteConnect
ZERODHA_API_KEY=your-zerodha-api-key
ZERODHA_API_SECRET=your-zerodha-api-secret

# ML service URL
PREDICTION_SERVICE_URL=http://localhost:8001
```

### `predictions/.env`

```env
# No external secrets required by default; inherits GOOGLE_API_KEY if needed
```

### `frontend/.env`

```env
VITE_API_BASE_URL=http://localhost:8000
```

---

## How to Run

### Prerequisites

- Python 3.10+
- Node.js 18+
- Git

---

### 1. Clone the Repository

```bash
git clone https://github.com/Krishaaashah/wealthwise.git
cd wealthwise
```

---

### 2. Backend (Django REST API)

```bash
cd backend

# Create and activate virtual environment
python -m venv venv

# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements_rag.txt

# Apply database migrations
python manage.py migrate

# (Optional) Create a superuser for Django admin
python manage.py createsuperuser

# Start the development server
python manage.py runserver
# Runs on http://localhost:8000
```

---

### 3. Prediction Service (FastAPI ML Microservice)

Open a **new terminal**:

```bash
cd predictions

python -m venv venv

# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt

# Start the FastAPI service
fastapi run app.py
# Runs on http://localhost:8001
```

> The Django backend calls this service internally. Ensure it is running before using the dashboard's prediction features.

---

### 4. Frontend (React + Vite)

Open another **new terminal**:

```bash
cd frontend

npm install

npm run dev
# Runs on http://localhost:5173
```

Open your browser at `http://localhost:5173`.

---

### Production Build

```bash
# Frontend
cd frontend
npm run build          # outputs to dist/

# Backend
pip install gunicorn
gunicorn wealthwise.wsgi:application --bind 0.0.0.0:8000

# ML Service
pip install uvicorn
uvicorn app:app --host 0.0.0.0 --port 8001
```

---

## API Reference

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register/` | Register with email and password |
| POST | `/api/auth/login/` | Obtain JWT access + refresh tokens |
| POST | `/api/auth/token/refresh/` | Refresh access token |
| GET | `/api/auth/google/` | Google OAuth login redirect |

### Chatbot

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/chatbot/chat/` | Send message or upload PDF; returns AI response |
| GET | `/api/chatbot/history/` | Retrieve conversation history |
| DELETE | `/api/chatbot/history/` | Clear conversation history |

### Financial Data

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/financial/stocks/` | Fetch live Zerodha portfolio data |
| POST | `/api/financial/portfolio/` | Submit manual portfolio data |
| GET | `/api/financial/dashboard/` | Retrieve corpus projections and analytics |

### ML Predictions

| Method | Endpoint | Description |
|---|---|---|
| POST | `/predictions/life-expectancy/` | Predict life expectancy from health biomarkers |
| GET | `/predictions/inflation/` | Fetch 3-year CPI inflation forecast |
| POST | `/predictions/risk-profile/` | Calculate portfolio risk category |

---

## Security

- **JWT Bearer Tokens** — all Django API endpoints require a valid token.
- **User Namespace Isolation** — each user's Pinecone vectors are stored in a private namespace; cross-user retrieval is architecturally impossible.
- **Encrypted API Keys** — Zerodha KiteConnect credentials are encrypted at rest and never exposed to the frontend.
- **API Rate Limiting** — prevents abuse of ML inference and LLM endpoints.
- **Input Validation** — real-time enforcement of logical constraints (e.g. retirement age > current age; EPF contribution ≤ 12%) in both frontend and backend.
- **CORS** — restricted to explicitly allowed origins.
- **No Hardcoded Secrets** — all credentials sourced from environment files; none exist in the codebase.

---

## Troubleshooting

| Issue | Resolution |
|---|---|
| `Pinecone Connection Error` | Verify `PINECONE_API_KEY` and `PINECONE_ENVIRONMENT` in `backend/.env` |
| `Google AI API Error` | Check `GOOGLE_API_KEY` and review API quota in Google Cloud Console |
| `CORS Error` | Add the frontend origin to `CORS_ALLOWED_ORIGINS` in Django settings |
| `Database Migration Error` | Run `python manage.py makemigrations` then `python manage.py migrate --fake-initial` |
| `FastAPI service unreachable` | Confirm `predictions/` service is running on port 8001 and `PREDICTION_SERVICE_URL` is set correctly |
| `Zerodha data not loading` | Ensure KiteConnect credentials are valid and the access token has not expired |

---

## Future Scope

- **Longitudinal health data** — integrate wearable device inputs (heart rate, sleep, activity) to update life expectancy predictions continuously rather than at a single point in time.
- **Enhanced inflation model** — replace ARMA with VAR/VARMAX incorporating RBI Repo Rate, crude oil prices, and the food price index as exogenous variables.
- **RAG answer relevancy improvement** — add a cross-encoder reranking stage before context augmentation to push answer relevancy beyond the current 0.7784 RAGAS score.
- **Multi-asset modelling** — extend portfolio analysis to include real estate, gold, and alternative investments, which are significant components of Indian household wealth.
- **NPS direct API** — integrate the National Pension System API (subject to regulatory approval) for real-time NPS account data ingestion.
- **Mobile apps** — deploy as iOS and Android applications to reach Tier 2 and Tier 3 city users where smartphone penetration exceeds internet banking adoption.
- **Multilingual support** — add regional language LLM inference to democratise access for non-English-speaking users.

