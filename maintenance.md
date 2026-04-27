# MediBot: Production Maintenance Guide 🛠️

When deploying an AI application like MediBot to a production environment (such as Hugging Face Spaces, AWS EC2, or Google Cloud Run), the initial code is only half the battle. This document outlines the Standard Operating Procedures (SOPs) for maintaining, updating, and monitoring the application in the wild.

---

## 1. Updating the Medical Knowledge Base (Vector Database)
The AI's knowledge is entirely dependent on the underlying FAISS database. If the medical datasets (`dataset.csv`, `Symptom-severity.csv`, etc.) are updated with *new* illnesses or refined severity weights:

**Procedure:**
1. Upload the newly modified CSV files into the `data/raw/` directory on the production server.
2. Re-trigger the vector ingestion pipeline to recreate the embeddings.
   ```bash
   export PYTHONPATH=.
   source venv/bin/activate
   # This will parse the CSVs and overwrite the existing index at /data/faiss_index
   python src/data_loading/vectorstore.py
   ```
3. Restart the application daemon/container to force `src/app.py` to load the fresh database into memory.

## 2. API Key Rotation and Security
In production, your Google Generative AI key and Langfuse keys might be compromised or need standard scheduled rotation (e.g., every 90 days).

**Procedure:**
1. Generate the new keys from Google AI Studio / Langfuse dashboard.
2. SSH into your deployment server (or access your PaaS Config Vars).
3. Replace the keys in the `.env` file.
4. If using Docker or a serverless environment, ensure the `.env` file is loaded at startup or the environment variables are securely injected via a Secrets Manager (like AWS Secrets Manager).

*Warning: Never commit your `.env` to the remote GitHub repository.*

## 3. Telemetry and Model Monitoring (Observability)
LLM applications exhibit non-deterministic behavior. Users might type unexpected queries. 

To maintain quality:
1. **Enable Observability:** In your production `src/app.py`, remove or modify the hardcoded `os.environ['LANGCHAIN_TRACING_V2'] = "false"` so that production traffic is logged.
2. **Review Traces:** Weekly, log into your LangSmith or Langfuse console.
3. **Monitor Costs:** Track the token consumption of `gemini-1.5-pro`. If costs exceed thresholds, consider swapping the model to a cheaper version (`gemini-1.5-flash`) inside `src/app.py`.
4. **Agent Fallbacks:** Identify queries where the LLM "fails" to use tools correctly. Use those failures to iteratively improve the `system_prompt` instructions in `src/app.py`.

## 4. Frontend & Dependency Management (Dependency Churn)
Python ML Ecosystems update rapidly, notably `langchain`, `langgraph`, and `gradio`.

**Procedure:**
1. Avoid running `pip install -U gradio` directly on production.
2. In your local staging environment, update versions incrementally. 
3. Re-run `python scripts/eda.py` and `python src/app.py` locally to verify there are no API breakage issues (e.g., keyword arguments changing).
4. Update `requirements.txt` with frozen versions (e.g., `gradio==4.36.0`) before pushing to the production server.
