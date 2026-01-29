# 📅 Day 4  
## 🗄️ Caching your App

For today's challenge, our goal is to **create a Streamlit web application that calls a Snowflake Cortex Large Language Model (LLM)** ❄️🧠.  
We need to build an interface where a user can:

- ✍️ Enter a prompt  
- 🤖 Send it to a powerful AI model (like **Claude 3.5 Sonnet**) running securely inside Snowflake  
- 📥 Get a response  
- ⏱️ See how long the request took  

Once that's done, we will display the AI's answer directly in the web app.

---

## 🧪 See the code:

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
📖 See the explanation
🧩 How It Works: Step-by-Step
Let's break down what each part of the code does.

1️⃣ Setup: Imports and Session
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
import ...: These lines import all the necessary libraries:

streamlit 🖥️ for the web app UI

time ⏱️ to measure response speed

json 📦 to parse the LLM output

Snowpark functions ❄️ to connect to Snowflake and call Cortex AI

try/except block: Automatically detects the environment and connects appropriately, working in all deployment scenarios

2️⃣ Defining the Cortex LLM Call
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
@st.cache_data: ⭐ This is a Streamlit decorator that caches the function result

First call → LLM runs (3–5 seconds)

Same prompt again → Instant response (< 0.1s)

Change even one character → Cache miss → LLM runs again

ai_complete(...): Securely calls the specified model (Claude 3.5 Sonnet) inside Snowflake Cortex

df.collect()[0][0]: Executes the query and extracts the single returned value

json.loads(response_raw): Parses the raw JSON text into a Python dictionary

3️⃣ Building the Web App Interface
prompt = st.text_input("Enter your prompt", "Why is the sky blue?")
if st.button("Submit"):
    start_time = time.time()
    response = call_cortex_llm(prompt)
    end_time = time.time()
    
    st.success(f"*Call took {end_time - start_time:.2f} seconds*")
    st.write(response)
st.text_input(...): Displays a text input box with a default prompt

st.button("Submit"): Runs the code only when the button is clicked

start_time / end_time: Captures execution time

call_cortex_llm(prompt): Calls the cached LLM function

st.success(...): Displays the execution time in a green success box ✅

st.write(response): Displays the full LLM response

When this code runs, you will see a simple webpage with a text box and a button.
After entering a prompt and clicking Submit, the LLM's response appears along with how long the call took.

📚 Resources
🔗 st.cache_data Documentation

🔗 Caching in Streamlit

🔗 Streamlit in Snowflake (SiS) Caching Limitations
