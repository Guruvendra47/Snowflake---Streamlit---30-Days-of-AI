
## 🔍 How It Works — Step by Step

### 1️⃣ Import Libraries & Connect to Snowflake

```python
import streamlit as st
from snowflake.snowpark.functions import ai_complete
import json
```

* `streamlit`: Builds the UI
* `ai_complete`: Snowflake Cortex function to call LLMs
* `json`: Used to parse the AI response

### Environment Detection Logic

```python
try:
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    from snowflake.snowpark import Session
    session = Session.builder.configs(
        st.secrets["connections"]["snowflake"]
    ).create()
```

✅ This makes the app **portable**:

* Works in **Streamlit in Snowflake**
* Works **locally**
* Works on **Streamlit Community Cloud**

---

### 2️⃣ Choose Model & Capture User Input

```python
model = "claude-3-5-sonnet"
prompt = st.text_input("Enter your prompt:")
```

* `model`: Specifies which Cortex LLM to use
* `st.text_input`: Captures user input from the UI

---

### 3️⃣ Run LLM Inference (Button Click)

```python
if st.button("Generate Response"):
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt).alias("response")
    )
```

🧠 Key idea:

* `session.range(1)` creates a **single-row DataFrame**
* The LLM is executed as part of a **Snowpark DataFrame operation**
* Output is stored in a column named `response`

👉 Think of it as:

> “Run the AI like a SQL function.”

---

### 4️⃣ Fetch & Display the AI Response

```python
response_raw = df.collect()[0][0]
response = json.loads(response_raw)
st.write(response)
```

* `.collect()` executes the query in Snowflake
* Cortex returns **JSON**, not plain text
* `json.loads()` converts it into a Python dictionary
* `st.write()` displays the result

Example raw response:

```json
{"choices":[{"messages":"Hello! How can I help you?"}]}
```

---

## 🤔 Why Use `ai_complete()`?

**Advantages**

* Integrates naturally with Snowpark DataFrames
* Ideal for data pipelines and SQL-style workflows
* Secure execution inside Snowflake

**Limitations**

* Returns JSON (needs parsing)
* No streaming support

👉 In **Day 3**, we will use the **Python `Complete()` API**, which:

* Is simpler
* Supports streaming responses

---

## 📌 Key Takeaways (Interview-Ready)

* Snowflake Cortex allows **LLMs to run directly inside Snowflake**
* `AI_COMPLETE` is a **Snowpark-native AI function**
* Streamlit + Cortex = **secure AI apps without external APIs**
* Data never leaves Snowflake → **enterprise-grade security**

---

## 📚 Resources

* 📘 Snowflake Cortex LLM Functions
* 📘 COMPLETE Function Reference
* 📘 Available LLM Models

---
