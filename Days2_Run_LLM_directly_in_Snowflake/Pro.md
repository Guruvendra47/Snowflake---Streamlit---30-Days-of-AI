# 📅 Day 2  
## 🤖 Hello, Cortex!

For today's challenge, our goal is to **run a large language model (LLM) directly within Snowflake** ❄️🧠.  
We need to create a simple **Streamlit interface** that:

- ✍️ Accepts a user's prompt  
- 📤 Sends it to a **Snowflake Cortex `AI_COMPLETE` function**  
- 📥 Gets a response  
- 🖥️ Displays the AI's generated response back to the user in the app  

---

## 🧪 Code:

```python
import streamlit as st
from snowflake.snowpark.functions import ai_complete
import json

st.title(":material/smart_toy: Hello, Cortex!")

# Connect to Snowflake
try:
    # Works in Streamlit in Snowflake
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    # Works locally and on Streamlit Community Cloud
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create() 

# Model and prompt
model = "claude-3-5-sonnet"
prompt = st.text_input("Enter your prompt:")

# Run LLM inference
if st.button("Generate Response"):
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt).alias("response")
    )
    
    # Get and display response
    response_raw = df.collect()[0][0]
    response = json.loads(response_raw)
    st.write(response)

# Footer
st.divider()
st.caption("Day 2: Hello, Cortex! | 30 Days of AI")
````

---

## 📖 Explanation

### 🧩 How It Works: Step-by-Step

Let's break down what each part of the code does.

---

## ⚙️ Install prerequisite libraries

In forthcoming lessons we'll leverage **Snowflake's Cortex AI** 🧠❄️ and therefore please install the following prerequisite libraries:

```text
snowflake-ml-python==1.20.0
snowflake-snowpark-python==1.44.0
```

---

### 💻 Locally

Save the above in `requirements.txt` and run:

```bash
pip install -r requirements.txt
```

Or you could also run:

```bash
pip install snowflake-ml-python==1.20.0 snowflake-snowpark-python==1.44.0
```

---

### ☁️ Streamlit Community Cloud

Save the above in `requirements.txt` and include this in the GitHub repo of your app.

---

### ❄️ Streamlit in Snowflake

Click on the **Packages** drop-down 📦 and enter the libraries name as shown.

---

## 1️⃣ Import Libraries & Connect

```python
import streamlit as st
from snowflake.snowpark.functions import ai_complete
import json

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

* `import streamlit as st`: Imports the Streamlit library, which is used to build the web app's user interface (UI) 🖥️
* `from snowflake.snowpark.functions ...`: Imports the specific Cortex AI function `ai_complete` that will run the LLM inference 🤖
* `try/except block`: Automatically detects the environment and uses the appropriate connection method:

  * **Streamlit in Snowflake (SiS)** ❄️: Uses `get_active_session()` for automatic authentication
  * **Locally or on Streamlit Community Cloud** ☁️: Uses `Session.builder` with credentials from `.streamlit/secrets.toml`
* `session`: The established Snowflake connection, ready to execute queries and call Cortex AI functions

### ❓ Why `ai_complete()`?

We use the Snowpark `ai_complete()` function here because it integrates naturally with **Snowpark DataFrames**.
This approach is ideal when you want to process data in a DataFrame pipeline or when you need the response as part of a SQL-like workflow.

⚠️ Trade-offs:

* Returns JSON that needs parsing
* Does not support streaming

👉 In **Day 3**, we'll see the Python `Complete()` API which is simpler for direct calls and supports streaming.

---

## 2️⃣ Set Up Model and UI

```python
# Model and prompt
model = "claude-3-5-sonnet"
prompt = st.text_input("Enter your prompt:")
```

* `model = "claude-3-5-sonnet"`: Sets a variable to specify which LLM we want to use from the models available in Snowflake Cortex 🧠
* `prompt = st.text_input(...)`: Creates a text input box in the Streamlit UI with the label **"Enter your prompt:"** ✍️
* Whatever the user types is stored in the `prompt` variable

---

## 3️⃣ Run Inference on Button Click

```python
# Run LLM inference
if st.button("Generate Response"):
    df = session.range(1).select(
        ai_complete(model=model, prompt=prompt).alias("response")
    )
```

* `if st.button(...)`: Creates a button in the UI 🖱️
* The code inside this block only runs when the user clicks **Generate Response**
* `session.range(1)`: Creates a single-row DataFrame (think of a one-cell spreadsheet 📊)
* `.select()`: Runs our AI function and captures the output
* `.alias("response")`: Renames the output column for easier access

---

## 4️⃣ Fetch and Display the Result

```python
# Get and display response
response_raw = df.collect()[0][0]
response = json.loads(response_raw)
st.write(response)
```

* `df.collect()`: Executes the query in Snowflake and pulls the data back into the app ❄️➡️🖥️
* `[0][0]`: Extracts the first row and first column
* `json.loads(response_raw)`: Converts the JSON string into a Python dictionary
* `st.write(response)`: Displays the final parsed response in the Streamlit app

---

## 📚 Resources

* 🔗 Snowflake Cortex LLM Functions
* 🔗 COMPLETE Function Reference
* 🔗 Available LLM Models

---

```
```
