# 🗓️ Day 15 — Model Comparison Arena

## 🎯 Goal of the Day

Day 15 wraps up **Week 2 (Chatbots)** by introducing a **practical model comparison tool**.

After building increasingly sophisticated chatbots (Days 8–14), we now answer a critical real‑world question:

> **Which LLM model should I use?**

For today’s challenge, we build a **side‑by‑side comparison arena** that:

* 🧪 Runs the **same prompt** on two different models
* ⏱️ Measures **latency (response time)**
* 🔢 Estimates **output token count**
* 💬 Displays responses **side‑by‑side** in chat format

This tool helps you make **informed trade‑offs** between speed, cost, and quality.

---

## 🤔 Why This Matters Now

* ✅ You’ve finished building complete chatbots (Days 8–14)
* 🚀 Week 3 starts **RAG applications** tomorrow
* 🤖 Different models behave very differently
* 💰 Cost, latency, and quality all vary by model

This comparison arena gives you **data‑driven clarity** instead of guessing.

---

## 🧩 See the Code

```python
import streamlit as st
import time
import json
from snowflake.snowpark.functions import ai_complete

# Connect to Snowflake
try:
    # Works in Streamlit in Snowflake
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    # Works locally and on Streamlit Community Cloud
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()

# Session state initialization
if "latest_results" not in st.session_state:
    st.session_state.latest_results = None

def run_model(model: str, prompt: str) -> dict:
    """Execute model and collect metrics."""
    start = time.time()

    # Call Cortex Complete function
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt).alias("response")
    )

    # Get response from dataframe
    rows = df.collect()
    response_raw = rows[0][0]
    response_json = json.loads(response_raw)

    # Extract text from response
    text = response_json.get("choices", [{}])[0].get("messages", "") if isinstance(response_json, dict) else str(response_json)

    latency = time.time() - start
    tokens = int(len(text.split()) * 4/3)  # Estimate tokens (1 token ≈ 0.75 words)

    return {
        "latency": latency,
        "tokens": tokens,
        "response_text": text
    }

def display_metrics(results: dict, model_key: str):
    """Display metrics for a model."""
    latency_col, tokens_col = st.columns(2)

    latency_col.metric("Latency (s)", f"{results[model_key]['latency']:.1f}")
    tokens_col.metric("Tokens", results[model_key]['tokens'])

def display_response(container, results: dict, model_key: str):
    """Display chat messages in container."""
    with container:
        with st.chat_message("user"):
            st.write(results["prompt"])
        with st.chat_message("assistant"):
            st.write(results[model_key]["response_text"])

# Model selection
llm_models = [
    "llama3-8b",
    "llama3-70b",
    "mistral-7b",
    "mixtral-8x7b",
    "claude-3-5-sonnet",
    "claude-haiku-4-5",
    "openai-gpt-5",
    "openai-gpt-5-mini"
]

st.title(":material/compare: Select Models")
col_a, col_b = st.columns(2)

col_a.write("**Model A**")
model_a = col_a.selectbox("Model A", llm_models, key="model_a", label_visibility="collapsed")

col_b.write("**Model B**")
model_b = col_b.selectbox("Model B", llm_models, key="model_b", index=1, label_visibility="collapsed")

# Response containers
st.divider()
col_a, col_b = st.columns(2)
results = st.session_state.latest_results

for col, model_name, model_key in [(col_a, model_a, "model_a"), (col_b, model_b, "model_b")]:
    with col:
        st.subheader(model_name)
        container = st.container(height=400, border=True)

        if results:
            display_response(container, results, model_key)

        st.caption("Performance Metrics")
        if results:
            display_metrics(results, model_key)
        else:
            latency_col, tokens_col = st.columns(2)
            latency_col.metric("Latency (s)", "—")
            tokens_col.metric("Tokens", "—")

# Chat input and execution
st.divider()
if prompt := st.chat_input("Enter your message to compare models"):
    with st.status(f"Running {model_a}..."):
        result_a = run_model(model_a, prompt)
    with st.status(f"Running {model_b}..."):
        result_b = run_model(model_b, prompt)

    st.session_state.latest_results = {
        "prompt": prompt,
        "model_a": result_a,
        "model_b": result_b
    }
    st.rerun()

st.divider()
st.caption("Day 15: Model Comparison Arena | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step‑by‑Step

---

## 1️⃣ Model Execution and Metrics Collection

* ⏱️ `time.time()` measures end‑to‑end latency
* 🤖 `ai_complete()` executes the selected model
* 📦 Response is parsed from JSON
* 🔢 Token count is **estimated** using a simple rule:

```
1 token ≈ 0.75 words
```

This approximation is good enough for **relative cost and size comparison**.

---

## 2️⃣ Side‑by‑Side UI Design

* 🎛️ Two dropdowns to select Model A and Model B
* 🧱 Fixed‑height containers ensure fair visual comparison
* 🔁 Loop pattern avoids code duplication
* 📊 Metrics shown consistently for both models

---

## 3️⃣ Sequential Execution Strategy

* ▶️ Model A runs first
* ▶️ Model B runs second
* 🧠 Results stored in Session State
* 🔄 `st.rerun()` refreshes UI with results

Sequential execution is **simpler and clearer** for learning, even though it’s slower than parallel execution.

---

## 🖥️ Final Result

When this code runs, you get:

* 🤖 Two LLMs answering the **same prompt**
* ⏱️ Clear latency comparison
* 🔢 Output size comparison
* 💬 Clean, chat‑style UI

This sets you up perfectly for **Week 3 (RAG applications)** where model choice becomes critical.

---

## 📚 Resources

* 📘 Snowflake Cortex `ai_complete`
* 📘 Streamlit `st.metric`
* 📘 Model Selection Best Practices

