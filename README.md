# MediBot: Your AI-Powered Symptom Checker

MediBot is an enterprise-grade AI-powered medical assistant utilizing LangChain's ReAct framework, multi-agent logic, and FAISS-based Retrieval-Augmented Generation (RAG) to serve as an intelligent symptom checker.

## Architecture

- **ReAct Agents**: A central agent orchestrates 4 specialized AI tools:
  - **Disease Diagnosis Agent**: Predicts probable illnesses using FAISS retrieval.
  - **Symptom Severity Agent**: Assesses medical urgency.
  - **Disease Description Agent**: Provides detailed descriptions of predicted conditions.
  - **Precaution Advisor Agent**: Advises on preventive measures and self-care.
- **RAG**: FAISS index stores vectorized medical knowledge generated from structured symptom mappings.
- **UI**: Intuitive Gradio chatbot interface allowing free-text natural conversations.

## Setup

1. **Clone & Environment**:
```bash
# In the medibot root directory, create a virtual environment
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

2. **Environment Variables**:
Ensure you have a `.env` file containing your valid **Google API Key** for Gemini.
```bash
# Edit .env and put your key
cp .env.example .env
```

3. **Data Ingestion & FAISS Creation**:
You can run the exploratory data analysis script to generate plots for the raw datasets, and then build the vector database.
```bash
export PYTHONPATH=.
python scripts/eda.py
python src/data_loading/vectorstore.py
```

4. **Run Server**:
```bash
export PYTHONPATH=.
python src/app.py
```
This will launch the Gradio server on `127.0.0.1:7860`. Open it in your browser and start consulting MediBot!
