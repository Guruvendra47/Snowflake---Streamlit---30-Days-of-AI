# 🗓️ Day 4 — Caching your App

## 🧠 Goal of the Day

For today's challenge, our goal is to create a **Streamlit web application** that calls a **Snowflake Cortex Large Language Model (LLM)**.

We need to build an interface where a user can:

* ✍️ Enter a prompt
* 🚀 Send it to a powerful AI model (like **Claude 3.5 Sonnet**) running securely inside **Snowflake**
* 📩 Get a response back
* ⏱️ See how long the request took

Once that's done, we will display the AI's answer directly in the web app, along with the execution time.

---

## 🧩 See the Code

```python
import streamlit as st
import time
import json
from snowflake.snowpark.functions import ai_complete

st.title(":material/cached: Caching your App")
# Connect to Snowflake
try:
    # Works in Streamlit in Snowflake
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    # Works locally and on Streamlit Community Cloud
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()

@st.cache_data
def call_cortex_llm(prompt_text):
    model = "claude-3-5-sonnet"
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt_text).alias("response")
    )
    
    # Get and parse response
    response_raw = df.collect()[0][0]
    response_json = json.loads(response_raw)
    return response_json

prompt = st.text_input("Enter your prompt", "Why is the sky blue?")

if st.button("Submit"):
    start_time = time.time()
    response = call_cortex_llm(prompt)
    end_time = time.time()
    
    st.success(f"*Call took {end_time - start_time:.2f} seconds*")
    st.write(response)

# Footer
st.divider()
st.caption("Day 4: Caching your App | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step‑by‑Step

Let's break down what each part of the code does.

---

## 1️⃣ Setup: Imports and Session

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
```

* 📦 `import ...`: These lines import all the necessary libraries:

  * **streamlit** → web app UI
  * **time** → measure response speed
  * **json** → parse the LLM's output
  * **snowpark** → connect to Snowflake and use AI functions

* 🔁 **try / except block**: Automatically detects the environment and connects appropriately, working in all deployment scenarios

---

## 2️⃣ Defining the Cortex LLM Call

```python
@st.cache_data
def call_cortex_llm(prompt_text):
    model = "claude-3-5-sonnet"
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt_text).alias("response")
    )
    
    # Get and parse response
    response_raw = df.collect()[0][0]
    response_json = json.loads(response_raw)
    return response_json
```

* 🧠 `@st.cache_data`: This is a Streamlit **decorator** that cleverly saves the result of this function.

  * First time → calls the LLM (3–5 seconds)
  * Same prompt again → instant response (< 0.1 sec)
  * Change even **one character** → cache miss → LLM runs again

* 🤖 `ai_complete(...)`: Core Snowpark function that securely calls the specified model (**Claude 3.5 Sonnet**) running in **Snowflake Cortex**

* 🧾 `df.collect()[0][0]`:

  * Executes the query
  * Fetches one row and one column
  * Returns a raw text string

* 🔓 `json.loads(response_raw)`:

  * Parses the raw JSON string
  * Converts it into a structured Python dictionary

---

## 3️⃣ Building the Web App Interface

```python
prompt = st.text_input("Enter your prompt", "Why is the sky blue?")

if st.button("Submit"):
    start_time = time.time()
    response = call_cortex_llm(prompt)
    end_time = time.time()
    
    st.success(f"*Call took {end_time - start_time:.2f} seconds*")
    st.write(response)
```

* ✏️ `st.text_input(...)`: Draws a text input box with a label and default text

* 🔘 `st.button("Submit")`:

  * Draws a button
  * Code runs **only when clicked**

* ⏱️ `start_time` / `end_time`:

  * Capture execution time

* 🔁 `call_cortex_llm(prompt)`:

  * Sends user input to Cortex LLM
  * Receives structured response

* ✅ `st.success(...)`:

  * Displays a green success box with elapsed time

* 📄 `st.write(response)`:

  * Displays the entire response dictionary

---

## 🖥️ Final Result

When this code runs, you will see:

* A simple webpage
* A text box for input
* A submit button
* The LLM's response displayed below
* The time taken for the request

---

## 📚 Resources

* 📘 **st.cache_data Documentation**
* 📘 **Caching in Streamlit**
* 📘 **SiS Caching Limitations**


