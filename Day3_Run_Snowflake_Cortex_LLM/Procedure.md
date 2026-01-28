# 📅 Day 3  
## ✍️ Write Streams

For today's challenge, our goal is to **run a Snowflake Cortex LLM using the `snowflake.cortex.Complete` Python API** ❄️🧠.  
We need to build a **Streamlit app** that:

- 🤖 Lets a user **select a model**
- ✍️ Enter a prompt
- 🌊 **Streams the response back**
- ⏱️ Displays the AI's response **in real-time, word by word**, as it's being generated

---

## 🧪  Code:

```python
import streamlit as st
from snowflake.cortex import Complete
import time

st.title(":material/airwave: Write Streams")

# Connect to Snowflake
try:
    # Works in Streamlit in Snowflake
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    # Works locally and on Streamlit Community Cloud
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create() 

llm_models = ["claude-3-5-sonnet", "mistral-large", "llama3.1-8b"]
model= st.selectbox("Select a model", llm_models)

example_prompt = "What is Python?"
prompt = st.text_area("Enter prompt", example_prompt)

# Choose streaming method
streaming_method = st.radio(
    "Streaming Method:",
    ["Direct (stream=True)", "Custom Generator"],
    help="Choose how to stream the response"
)

if st.button("Generate Response"):

    # Method 1: Direct streaming with stream=True
    if streaming_method == "Direct (stream=True)":
        with st.spinner(f"Generating response with `{model}`"):
            stream_generator = Complete(
                        session=session,
                        model=model,
                        prompt=prompt,
                        stream=True,
                    )
                    
            st.write_stream(stream_generator)
    
    else:
        # Method 2: Custom generator (for compatibility)
        def custom_stream_generator():
            """
            Alternative streaming method for cases where
            the generator is not compatible with st.write_stream
            """
            output = Complete(
                session=session,
                model=model,
                prompt=prompt
            )
            for chunk in output:
                yield chunk
                time.sleep(0.01)  # Small delay for smooth streaming
        
        with st.spinner(f"Generating response with `{model}`"):
            st.write_stream(custom_stream_generator)

# Footer
st.divider()
st.caption("Day 3: Write streams | 30 Days of AI")
````

---

## 📖 Explanation

### 🧩 How It Works: Step-by-Step

Let's break down what each part of the code does.

---

## 1️⃣ Imports and Session

```python
import streamlit as st
from snowflake.cortex import Complete
import time
```

```python
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

* `import streamlit as st`: Imports the library needed to build the web app's user interface (UI) 🖥️
* `from snowflake.cortex import Complete`: ⭐ Key import. Instead of using the SQL function `ai_complete`, we are importing the **direct Python `Complete` class** from the Cortex SDK, which is designed for programmatic use
* `import time`: Added for the custom generator method, which uses a small delay to smooth out streaming ⏱️
* `try/except block`: Automatically detects the environment and connects appropriately (Streamlit in Snowflake vs Local/Community Cloud)
* `session`: The established Snowflake connection ❄️

---

## 2️⃣ Configure the User Interface

```python
llm_models = ["claude-3-5-sonnet", "mistral-large", "llama3.1-8b"]
model = st.selectbox("Select a model", llm_models)
```

```python
example_prompt = "What is Python?"
prompt = st.text_area("Enter prompt", example_prompt)
```

```python
# Choose streaming method
streaming_method = st.radio(
    "Streaming Method:",
    ["Direct (stream=True)", "Custom Generator"],
    help="Choose how to stream the response"
)
```

* `llm_models = [...]`: Defines a Python list of model names the user can choose from 🤖
* `model = st.selectbox(...)`: Creates a drop-down menu labeled **"Select a model"**
* `prompt = st.text_area(...)`: Creates a multi-line text box for the user's prompt and pre-populates it with an example
* `streaming_method = st.radio(...)`: ⭐ Adds a radio button to let users choose between **two streaming methods** and see the difference in behavior

---

## 3️⃣ Stream the LLM Response

This app demonstrates **two methods** for streaming responses 🌊:

---

### 🟢 Method 1: Direct Streaming (`stream=True`)

```python
if st.button("Generate Response"):
    with st.spinner(f"Generating response with `{model}`"):
        stream_generator = Complete(
                    session=session,
                    model=model,
                    prompt=prompt,
                    stream=True,
                )

        st.write_stream(stream_generator)
```

* `stream=True`: The simplest approach
* Tells `Complete` to return a generator that yields tokens as they arrive
* ✅ Works when the API's streaming output is directly compatible with `st.write_stream()`

---

### 🟡 Method 2: Custom Generator (Compatibility Mode)

```python
def custom_stream_generator():
    """
    Alternative streaming method for cases where
    the generator is not compatible with st.write_stream
    """
    output = Complete(
        session=session,
        model=model,
        prompt=prompt
    )
    for chunk in output:
        yield chunk
        time.sleep(0.01)  # Small delay for smooth streaming
```

```python
with st.spinner(f"Generating response with `{model}`"):
    st.write_stream(custom_stream_generator)
```

* **When to use**: If `stream=True` doesn’t work with `st.write_stream()`
* **How it works**: Manually yields chunks from the response with a small delay for smooth visual streaming
* **Docstring**: Clearly explains why this alternative method exists
* ⭐ **Best practice**: More reliable for chatbots and complex prompts (used in later days)

---

## 💡 Why streaming matters

Without streaming ❌, users stare at a blank screen while the LLM generates the full response.

With streaming ✅:

* Words appear immediately ✨
* The app feels faster and more responsive
* Even though total generation time is the same

---

## 📚 Resources

* 🔗 Cortex Complete Python API
* 🔗 `st.write_stream` Documentation

---

