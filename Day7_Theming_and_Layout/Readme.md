# 🗓️ Day 7 — Theming and Layout

## 🎯 Goal of the Day

For today's challenge, we're building upon the app from **Day 6**, but this time the focus is on **theming and layout**.

The objective is to:

* 🎨 Transform a functional app into a **polished, branded experience**
* 🌙 Enable **Dark Mode** using Streamlit theming
* 🧭 Introduce a clean **sidebar-based layout**
* 🎯 Apply custom colors and spacing for a professional look and feel

By the end, we will have a sleek, dark-themed app with clear visual hierarchy and better usability.

---

## 🧩 See the Code

```python
import streamlit as st
import json
import time
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

# Input widgets
st.subheader(":material/input: Input content")
content = st.text_input("Content URL:", "https://docs.snowflake.com/en/user-guide/views-semantic/overview")

with st.sidebar:
    st.title(":material/post: LinkedIn Post Generator v3")
    st.success("An app for generating LinkedIn post using content from input link.")
    tone = st.selectbox("Tone:", ["Professional", "Casual", "Funny"])
    word_count = st.slider("Approximate word count:", 50, 300, 100)

# Generate button
if st.button("Generate Post"):
    
    # Initialize the status container
    with st.status("Starting engine...", expanded=True) as status:
        
        # Step 1: Construct Prompt
        st.write(":material/psychology: Thinking: Analyzing constraints and tone...")

        # Add a slight delay
        time.sleep(2)
        
        prompt = f"""
        You are an expert social media manager. Generate a LinkedIn post based on the following:

        Tone: {tone}
        Desired Length: Approximately {word_count} words
        Use content from this URL: {content}

        Generate only the LinkedIn post text. Use dash for bullet points.
        """
        
        # Step 2: Call API
        st.write(":material/flash_on: Generating: contacting Snowflake Cortex...")

        # Add a slight delay
        time.sleep(2)
        
        # This is the blocking call that takes time
        response = call_cortex_llm(prompt)
        
        # Step 3: Update Status to Complete
        st.write(":material/check_circle: Post generation completed!")
        status.update(label="Post Generated Successfully!", state="complete", expanded=False)

    # Display Result
    with st.container(border=True):
        st.subheader(":material/output: Generated post:")
        st.markdown(response)

# Footer
st.divider()
st.caption("Day 7: Theming and Layout | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step-by-Step

This day focuses specifically on **visual design and structure**, rather than new AI logic.

---

## 1️⃣ Global Styling (Theming)

To apply a custom look across the entire app without writing CSS, we use Streamlit’s native theming support by creating a `.streamlit/config.toml` file.

```toml
# .streamlit/config.toml
[server]
enableStaticServing = true

[theme]
base = "dark"
primaryColor = "#1ed760"
backgroundColor = "#2a2a2a"
secondaryBackgroundColor = "#121212"
codeBackgroundColor = "#121212"
textColor = "#ffffff"
linkColor = "#1ed760"
borderColor = "#7c7c7c"
showWidgetBorder = true
baseRadius = "0.3rem"
buttonRadius = "full"
headingFontWeights = [600, 500, 500, 500, 500, 500]
headingFontSizes = ["3rem", "2.5rem", "2.25rem", "2rem", "1.5rem"]
chartCategoricalColors = ["#ffc862", "#2d46b9", "#b49bc8"]

[theme.sidebar]
backgroundColor = "#121212"
secondaryBackgroundColor = "#000000"
codeBackgroundColor = "#2a2a2a"
borderColor = "#696969"
```

* 🌙 `base = "dark"`: Instantly switches the app to Dark Mode
* 🎨 `primaryColor`: Sets the accent color for buttons and links
* 💊 `buttonRadius = "full"`: Creates pill-shaped buttons
* 🧭 `[theme.sidebar]`: Separately styles the sidebar for visual separation

---

## 2️⃣ Layout Structure: The Sidebar

Instead of stacking everything in the main page, configuration options are moved to the sidebar.

```python
st.subheader(":material_input: Input content")
content = st.text_input("Content URL:", "https://docs.snowflake.com/en/user-guide/views-semantic/overview")

with st.sidebar:
    st.title(":material_post: LinkedIn Post Generator v3")
    st.success("An app for generating LinkedIn post using content from input link.")
    tone = st.selectbox("Tone:", ["Professional", "Casual", "Funny"])
    word_count = st.slider("Approximate word count:", 50, 300, 100)
```

* 📦 `with st.sidebar`: Moves widgets into a collapsible side panel and `:material_post:` Streamlit supports Material Design icons directly in strings. This adds a relevant icon next to the title without needing image files.
* 🎯 Keeps the main area focused on input and output
* 🖼️ Material icons enhance clarity without extra assets

---

## 3️⃣ Progress Feedback with Status Container

```python
with st.status("Starting engine...", expanded=True) as status:

    st.write(":material_psychology: Thinking: Analyzing constraints and tone...")
    time.sleep(2)

    st.write(":material_flash_on: Generating: contacting Snowflake Cortex...")
    time.sleep(2)

    response = call_cortex_llm(prompt)

    st.write(":material_check_circle: Post generation completed!")
    status.update(label="Post Generated Successfully!", state="complete", expanded=False)
```

* ⏳ `st.status()`: Shows a live, step-by-step progress indicator
* ✅ `status.update()`: Marks the task complete and collapses the container
* 🕒 `time.sleep(2)`: Slows transitions so users can read each step (optional in production)

This transforms a generic loading state into a transparent and confidence-building experience.

---

## 4️⃣ Output Grouping and Visual Hierarchy

```python
with st.container(border=True):
    st.subheader(":material_output: Generated post:")
    st.markdown(response)
```

* 📦 `st.container(border=True)`: Wraps output in a bordered card
* 🏷️ Clearly separates final results from input and processing logs
* 🎁 Makes the generated content feel like a deliverable

---

## 🖥️ Final Result

When this code runs, you will see:

* A dark-themed Streamlit application
* Sidebar-based configuration controls
* Clear progress indicators during AI processing
* A clean, bordered output section for the generated post

---

## 📚 Resources

* 📘 **Streamlit Theming Guide**
* 📘 **config.toml Configuration**
* 📘 **st.sidebar Documentation**
* 📘 **Material Icons in Streamlit**


