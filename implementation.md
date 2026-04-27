# Building MediBot: An Instructor's Guide 🩺

Welcome to the MediBot curriculum! The goal of this document is to act as your personal instructor. Rather than just giving you the final product, this guide will walk you through **how** and **why** we implemented the project the way we did.

By reading through this, you will understand the fundamentals of Retrival-Augmented Generation (RAG) and Langchain ReAct agents, and how to build them step-by-step.

---

## 🛑 Course Overview: The MediBot Architecture

To build an AI Symptom Checker, we had to solve a fundamental problem: Large Language Models (LLMs) like Gemini hallucinate. If you ask an LLM for medical advice directly, it might confidently make things up. 

To fix this, we implement **Tools** and **RAG**. Instead of letting Gemini guess the disease, we give Gemini a set of "Specialized Tools". We force the AI to use these tools to look up real, statistically-backed medical records from your CVS files.

### The 4 Phases of our Implementation

1. **Phase 1: Exploratory Data Analysis (EDA) - "Understanding our Data"**
2. **Phase 2: RAG & FAISS - "Building the AI's Brain"**
3. **Phase 3: The Multi-Agent Tools - "Giving the AI Hands"**
4. **Phase 4: ReAct Agent & Gradio UI - "The Conductor"**

---

## Phase 1: Exploratory Data Analysis (EDA)
**File:** `scripts/eda.py`

Before we code the AI, we need to understand the raw data inside `data/raw`. 
- **What we did**: We wrote a script (`eda.py`) that loads your data into `pandas` DataFrames.
- **Why we did it**: We found that some symptoms had extra spaces (e.g., `" spoting_ urination"`). These formatting errors trick AI into thinking `"spoting urination"` is a different symptom than `"spoting_urination"`. We cleaned these up by standardizing strings using `.strip().replace('_', ' ')`.
- **Outcome**: The script also generated visual correlation plots in your `plots/` folder so you can see which diseases have the highest severity weights.

---

## Phase 2: RAG & Vector Databases (FAISS)
**File:** `src/data_loading/processor.py` & `src/data_loading/vectorstore.py`

We have files like `dataset.csv` holding diseases and symptoms. We need the AI to search this quickly. 

### How RAG Works Here:
1. **Processor**: `processor.py` acts as a blender. It reads all four datasets (Mapping, Severity, Description, Precautions), matches them by the `Disease` column, and creates a large paragraph for each disease in plain english. We call these `LangChain Documents`.
2. **Vector Space**: In `vectorstore.py`, we take these paragraphs and convert them into **numbers (Embeddings)** using an AI model named `all-MiniLM-L6-v2`. 
3. **FAISS**: Facebook AI Similarity Search (FAISS) is the database we used to save these number embeddings. Later, when a user types "I have a headache", we turn that text into numbers, find the closest matching numbers in FAISS, and extract the matching disease!

---

## Phase 3: Giving the AI "Tools"
**Files**: `src/agents/diagnosis.py`, `severity.py`, `description.py`, `precaution.py`, `tools.py`

The core of our "Multi-Agent" strategy relies on explicitly separating concerns. We wrote native Python functions and wrapped them with LangChain's `@tool` decorator. 

### What is a Tool?
A Tool is standard python code that an LLM can choose to trigger.
- `severity_tool`: Simply looks at the user’s symptoms, maps them to the `Symptom-severity.csv`, averages the weight, and returns if things are "Low, Moderate, or High Urgency."
- `diagnosis_tool`: Takes the user's symptoms and queries the FAISS vector database we built in Phase 2, returning the Top 3 matches.

When building your own agents, creating specialized, isolated tools ensures the AI gets highly deterministic outputs!

---

## Phase 4: The ReAct Agent 
**File**: `src/app.py`

This is where the magic happens. "ReAct" stands for **Reasoning** + **Acting**.
We use the `create_tool_calling_agent` pipeline from LangChain. 

### The System Prompt
Notice how specific the instructions are inside `app.py`:
```python
"Workflow:\n"
"1. First, use `severity_tool` to assess the urgency of the symptoms.\n"
"2. Second, use `diagnosis_tool` to predict the disease.\n"
"3. Then, you can provide `description_tool` and `precaution_tool` if asked..."
```
This forces Gemini to think methodically. It will *reason*: "The user wants a diagnosis. First, I must call `severity_tool`." It then *acts* (triggers the python code), observes the result, and loops back into reasoning.

### The Observability (Langfuse)
In the `.env` layer, passing observability keys allows an external dashboard to "eavesdrop" on the ReAct loop. You can see how long Gemini took to think, what tool it chose, and the exact query it sent to FAISS!

---

## 🛠️ Homework / Next Steps for You
Try modifying the system! Here is an exercise for you:
1. Open `src/agents/severity.py`. 
2. Scroll to line 31 where it says `if avg_severity >= 5.5:`. 
3. Try changing the text (`"High Urgency! "`) to something else. 
4. Restart the app using `python src/app.py` and see your changes securely take effect without modifying the machine learning portion itself!
