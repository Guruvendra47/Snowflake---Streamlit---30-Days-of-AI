# 🗓️ Day 6 — Status UI for Long-Running Task

## 🎯 Goal of the Day

For today's challenge, our goal is to build a **v2 of the LinkedIn Post Generator** web app.

In this version, we:

* 🔗 Integrate a **Streamlit frontend** with **Snowflake Cortex AI**
* 🤖 Use the **Claude 3.5 Sonnet** model to generate social media content
* ⏳ Handle **long‑running AI calls** gracefully
* 📊 Provide **real‑time status updates** so the app does not feel frozen

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
st.title(":material/post: LinkedIn Post Generator v2")

# Input widgets
content = st.text_input("Content URL:", "https://docs.snowflake.com/en/user-guide/views-semantic/overview")
tone = st.selectbox("Tone:", ["Professional", "Casual", "Funny"])
word_count = st.slider("Approximate word count:", 50, 300, 100)

# Generate button
if st.button("Generate Post"):
    
    # Initialize the status container
    with st.status("Starting engine...", expanded=True) as status:
        
        # Step 1: Construct Prompt
        st.write(":material/psychology: Thinking: Analyzing constraints and tone...")
        prompt = f"""
        You are an expert social media manager. Generate a LinkedIn post based on the following:

        Tone: {tone}
        Desired Length: Approximately {word_count} words
        Use content from this URL: {content}

        Generate only the LinkedIn post text. Use dash for bullet points.
        """
        
        # Step 2: Call API
        st.write(":material/flash_on: Generating: contacting Snowflake Cortex...")
        
        # This is the blocking call that takes time
        response = call_cortex_llm(prompt)
        
        # Step 3: Update Status to Complete
        st.write(":material/check_circle: Post generation completed!")
        status.update(label="Post Generated Successfully!", state="complete", expanded=False)

    # Display Result
    st.subheader("Generated Post:")
    st.markdown(response)

# Footer
st.divider()
st.caption("Day 6: Status UI for Long-Running Task | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step-by-Step

Let's break down what each part of the code does.

---

## 1️⃣ Connection and AI Function Wrapper

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

@st.cache_data
def call_cortex_llm(prompt_text):
    """Makes a call to Cortex AI with the given prompt."""
    model = "claude-3-5-sonnet"
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt_text).alias("response")
    )
    response_raw = df.collect()[0][0]
    return json.loads(response_raw)
```

* 🔌 `get_active_session()`: Retrieves the current Snowflake connection so the app can access Cortex AI
* 🔁 **try / except block**: Automatically detects the runtime environment (SiS, local, Community Cloud)
* 🧠 `@st.cache_data`: Caches identical prompts to avoid repeated AI calls and reduce cost
* 📄 `session.range(1).select(...)`: Snowpark pattern to execute scalar AI functions
* 🤖 `ai_complete`: Sends the prompt to Snowflake Cortex using **Claude 3.5 Sonnet**

---

## 2️⃣ Building the User Interface

```python
st.title(":material_post: LinkedIn Post Generator v2")

content = st.text_input("Content URL:", "https://docs.snowflake.com/en/user-guide/views-semantic/overview")
tone = st.selectbox("Tone:", ["Professional", "Casual", "Funny"])
word_count = st.slider("Approximate word count:", 50, 300, 100)
```

* 🏷️ `st.title`: Sets the main header of the application
* 🔗 `st.text_input`: Field for the source URL
* 🎭 `st.selectbox`: Restricts tone selection to predefined values
* 📏 `st.slider`: Controls the desired length of the generated content

---

## 3️⃣ Execution and Status Management

```python
if st.button("Generate Post"):
    with st.status("Starting engine...", expanded=True) as status:

        st.write(":material_psychology: Thinking: Analyzing constraints and tone...")
        prompt = f"""..."""

        st.write(":material_flash_on: Generating: contacting Snowflake Cortex...")
        response = call_cortex_llm(prompt)

        st.write(":material_check_circle: Post generation completed!")
        status.update(label="Post Generated Successfully!", state="complete", expanded=False)

    st.subheader("Generated Post:")
    st.markdown(response)
```

* 🔘 `if st.button("Generate Post")`: Code executes only when the button is clicked
* ⏳ `st.status(...)`: Displays a live status container so users see progress
* 🧩 Prompt construction dynamically injects user inputs
* ✅ `status.update(...)`: Marks the task as complete and collapses the status UI
* 📝 `st.markdown(response)`: Renders the AI‑generated LinkedIn post

---

## 🖥️ Final Result

When this code runs, you will see:

* A clean Streamlit interface
* Input fields for URL, tone, and length
* A visible progress/status indicator
* A completed LinkedIn post displayed after generation

---

## 📚 Resources

* 📘 **st.status Documentation**
* 📘 **Build an LLM App using LangChain**


