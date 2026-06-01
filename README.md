# WealthWise

Plan Smart, Retire Confident.

WealthWise is an AI-assisted, privacy-first platform that helps retirees and near-retirees model scenarios, analyze retirement outcomes, and get plain-language guidance.

## PS

Plan Smart is the core idea behind the product: help users compare decisions before they make them.

## Objective

Build a system that combines financial modeling, AI assistance, and predictive insights to support better retirement planning.

## Pipeline

1. Collect user inputs and financial details.
2. Process the data through the backend and prediction services.
3. Generate scenarios, insights, and recommendations.
4. Present results in the web interface for review and action.

## Tech Stack

- Frontend: React, Vite, Tailwind CSS, Chart.js
- Backend: Django REST Framework
- ML Service: FastAPI, Scikit-learn, XGBoost
- Data and AI: Pinecone, document processing, API integrations

## Project Structure

```
WealthWise/
├── frontend/
├── backend/
├── predictions/
└── README.md
```

## Features

- Financial analytics and scenario modeling
- AI-powered chatbot for document and retirement queries
- Predictive modeling for retirement-related insights
- Modern responsive interface

## How To Clone

```bash
git clone https://github.com/Krishaaashah/wealthwise.git
cd wealthwise
```

## How To Run

### Backend

```bash
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
pip install -r requirements_rag.txt
python manage.py migrate
python manage.py runserver
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Prediction Service

```bash
cd predictions
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
fastapi run app.py
```

## Images









Add screenshots and other images here.

#### Chatbot
- `POST /api/chatbot/chat/` - Send message to chatbot
- `GET /api/chatbot/history/` - Get conversation history
- `DELETE /api/chatbot/history/` - Clear conversation history

#### Financial Data
- `GET /api/financial/stocks/` - Get stock data
- `POST /api/financial/portfolio/` - Add portfolio data

#### ML Predictions
- `POST /predictions/life-expectancy/` - Predict life expectancy
- `POST /predictions/risk-profile/` - Calculate risk profile

## 🧠 AI & ML Features

### Conversation Intelligence
- **Intent Classification**: Automatically categorizes user requests
- **Context Awareness**: Maintains conversation history
- **Multi-turn Conversations**: Handles complex, ongoing discussions

### Document Processing
- **PDF Text Extraction**: Extracts text from uploaded documents
- **Information Extraction**: Uses LLM to extract structured data
- **Semantic Search**: Vector-based document search with Pinecone

### Predictive Analytics
- **Health-based Life Expectancy**: ML model considering lifestyle factors
- **Risk Assessment**: Portfolio risk analysis
- **Retirement Projections**: Financial planning algorithms

## 📊 Data Models

### User Data Structure
```json
{
  "personal": {
    "name": "string",
    "age": "number",
    "dateOfBirth": "date",
    "gender": "string",
    "location": "string",
    "maritalStatus": "string",
    "numberOfDependants": "number"
  },
  "financial": {
    "currentSalary": "number",
    "yearsOfService": "number",
    "employerType": "string",
    "pensionScheme": "string",
    "pensionBalance": "number",
    "portfolioValue": "number"
  },
  "health": {
    "height": "number",
    "weight": "number",
    "bmi": "number",
    "physicalActivity": "string",
    "smokingStatus": "string",
    "medicalConditions": "object"
  }
}
```

## 🔒 Security Features

- **JWT Authentication**: Secure token-based authentication
- **User Isolation**: Each user's data and documents are isolated
- **API Rate Limiting**: Protection against abuse
- **Input Validation**: Comprehensive data validation
- **CORS Configuration**: Secure cross-origin requests

## 🚀 Deployment

### Production Setup

1. **Backend (Django)**
   ```bash
   # Set production environment variables
   DEBUG=False
   ALLOWED_HOSTS=your-domain.com
   
   # Use production database (PostgreSQL recommended)
   # Configure static files serving
   python manage.py collectstatic
   
   # Use production WSGI server (Gunicorn)
   pip install gunicorn
   gunicorn wealthwise.wsgi:application
   ```

2. **Frontend (React)**
   ```bash
   # Build for production
   npm run build
   
   # Serve with nginx or deploy to Vercel/Netlify
   ```

3. **ML Service (FastAPI)**
   ```bash
   # Use production ASGI server (Uvicorn)
   pip install uvicorn
   uvicorn app:app --host 0.0.0.0 --port 8001
   ```


## 🆘 Troubleshooting

### Common Issues

1. **Pinecone Connection Error**
   ```
   Solution: Check PINECONE_API_KEY and PINECONE_ENVIRONMENT in .env
   ```

2. **Google AI API Error**
   ```
   Solution: Verify GOOGLE_API_KEY and check API quotas
   ```

3. **CORS Issues**
   ```
   Solution: Update CORS_ALLOWED_ORIGINS in Django settings
   ```

4. **Database Migration Errors**
   ```bash
   python manage.py makemigrations --empty appname
   python manage.py migrate --fake-initial
   ```