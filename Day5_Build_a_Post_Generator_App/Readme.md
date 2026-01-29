# 🗓️ Day 5 — Build a Post Generator App

## 🎯 Goal of the Day

For today's challenge, our goal is to build a **Streamlit web application** that generates a **professional LinkedIn post**.

We need to:

* 🔗 Accept user input such as a **content URL**
* 🎭 Allow the user to choose a **tone** (Professional, Casual, Funny)
* 📏 Control the **length** of the post
* 🤖 Call **Snowflake Cortex AI** from inside the Streamlit app
* 📝 Display the generated LinkedIn post in the UI

---

## 🧩 See the Code

```python
import streamlit as st
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

# Cached LLM Function
@st.cache_data
def call_cortex_llm(prompt_text):
    """Makes a call to Cortex AI with the given prompt."""
    model = "claude-3-5-sonnet"
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt_text).alias("response")
    )
    
    # Get and parse response
    response_raw = df.collect()[0][0]
    response_json = json.loads(response_raw)
    return response_json

# --- App UI ---
st.title(":material/post: LinkedIn Post Generator")

# Input widgets
content = st.text_input("Content URL:", "https://docs.snowflake.com/en/user-guide/views-semantic/overview")
tone = st.selectbox("Tone:", ["Professional", "Casual", "Funny"])
word_count = st.slider("Approximate word count:", 50, 300, 100)

# Generate button
if st.button("Generate Post"):
    # Construct the prompt
    prompt = f"""
    You are an expert social media manager. Generate a LinkedIn post based on the following:

    Tone: {tone}
    Desired Length: Approximately {word_count} words
    Use content from this URL: {content}

    Generate only the LinkedIn post text. Use dash for bullet points.
    """
    
    response = call_cortex_llm(prompt)
    st.subheader("Generated Post:")
    st.markdown(response)

# Footer
st.divider()
st.caption("Day 5: Build a Post Generator App | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step-by-Step

Let's break down what each part of the code does.

---

## 1️⃣ Setup and Snowflake Cortex AI Function

```python
import streamlit as st
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

# Cached LLM Function
@st.cache_data
def call_cortex_llm(prompt_text):
    """Makes a call to Cortex AI with the given prompt."""
    model = "claude-3-5-sonnet"
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt_text).alias("response")
    )

    # Get and parse response
    response_raw = df.collect()[0][0]
    response_json = json.loads(response_raw)
    return response_json
```

* 📦 `import streamlit as st`: Imports Streamlit to build the web application UI and logic
* 🤖 `ai_complete`: Snowpark function used to interact with **Snowflake Cortex AI (LLMs)**
* 🔁 **try / except block**: Automatically detects the environment and connects appropriately (Streamlit in Snowflake, local, or Community Cloud)
* 🔌 `session`: The active Snowflake connection used to execute queries
* 🧠 `@st.cache_data`: Caches the LLM response so the AI call is not repeated unless the input prompt changes
* 🚀 `ai_complete(model=model, prompt=prompt_text)`: Sends the user's prompt to the specified model (**claude-3-5-sonnet**) and returns generated text

---

## 2️⃣ Building the Streamlit UI and User Input

```python
# --- App UI ---
st.title("LinkedIn Post Generator")

# Input widgets
content = st.text_input("Content URL:", "https://docs.snowflake.com/en/user-guide/views-semantic/overview")
tone = st.selectbox("Tone:", ["Professional", "Casual", "Funny"])
word_count = st.slider("Approximate word count:", 50, 300, 100)

# Generate button
if st.button("Generate Post"):
```

* 🏷️ `st.title(...)`: Sets the main title of the application
* 🔗 `st.text_input(...)`: Text field where the user pastes the content URL
* 🎭 `st.selectbox(...)`: Dropdown menu to select the desired tone
* 📏 `st.slider(...)`: Slider to control the approximate length of the post
* 🔘 `st.button("Generate Post")`: Executes the logic only when the user clicks the button

---

## 3️⃣ Prompt Construction and Output Display

```python
# Construct the prompt
prompt = f"""
You are an expert social media manager. Generate a LinkedIn post based on the following:

Tone: {tone}
Desired Length: Approximately {word_count} words
Use content from this URL: {content}

Generate only the LinkedIn post text. Use dash for bullet points.
"""

response = call_cortex_llm(prompt)
st.subheader("Generated Post:")
st.markdown(response)
```

* 🧩 `prompt = f"""..."""`: Dynamically builds the LLM prompt using user inputs
* 🎯 Role-based instruction (`expert social media manager`) helps improve output quality
* 🔁 `call_cortex_llm(prompt)`: Sends the final prompt to Snowflake Cortex AI
* 🧾 `st.subheader(...)`: Clearly labels the output section
* 📝 `st.markdown(response)`: Displays the generated LinkedIn post with Markdown formatting

---

## 🖥️ Final Result

When this code runs, you will see:

* A clean Streamlit web page
* Input fields for URL, tone, and length
* A **Generate Post** button
* A professionally written LinkedIn post displayed below

---

## 📚 Resources

* 📘 **Snowflake Cortex LLM Functions**
* 📘 **st.text_input Documentation**
* 📘 **st.selectbox Documentation**
* 📘 **st.slider Documentation**


