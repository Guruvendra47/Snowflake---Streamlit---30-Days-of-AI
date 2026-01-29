# 🗓️ Day 10 — Your First Chatbot (with State)

## 🎯 Goal of the Day

For today's challenge, our goal is to **combine chat elements and Session State** to build a chatbot that **remembers the conversation**.

By doing this, we:

* 💬 Use `st.chat_message` to render chat bubbles
* 🧠 Store messages in `st.session_state`
* 🔁 Preserve conversation history across interactions

Once completed, we will have a **working chatbot** that no longer forgets what was said before.

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

def call_llm(prompt_text: str) -> str:
    """Call Snowflake Cortex LLM."""
    df = session.range(1).select(
        ai_complete(model="claude-3-5-sonnet", prompt=prompt_text).alias("response")
    )
    response_raw = df.collect()[0][0]
    response_json = json.loads(response_raw)
    # Extract text from response
    if isinstance(response_json, dict):
        return response_json.get("choices", [{}])[0].get("messages", "")
    return str(response_json)

st.title(":material/chat: My First Chatbot")

# Initialize the messages list in session state
if "messages" not in st.session_state:
    st.session_state.messages = []

# Display all messages from history
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.write(message["content"])

# Chat input
if prompt := st.chat_input("What would you like to know?"):
    # Add user message to state
    st.session_state.messages.append({"role": "user", "content": prompt})
    
    # Display user message
    with st.chat_message("user"):
        st.write(prompt)
    
    # Generate and display assistant response
    with st.chat_message("assistant"):
        response = call_llm(prompt)
        st.write(response)
    
    # Add assistant response to state
    st.session_state.messages.append({"role": "assistant", "content": response})

st.divider()
st.caption("Day 10: Your First Chatbot (with State) | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step-by-Step

Let’s break down how memory is added to the chatbot.

---

## 1️⃣ Initialize Message Storage

```python
st.title(":material_chat: My First Chatbot")

# Initialize the messages list in session state
if "messages" not in st.session_state:
    st.session_state.messages = []
```

* 🧠 `st.session_state.messages = []`:

  * Creates a **persistent list** to store the conversation
  * Each item is a dictionary with:

    * `role` → "user" or "assistant"
    * `content` → message text

* 📌 This message format is **industry standard** (used by OpenAI, Anthropic, etc.)

* ✅ `if "messages" not in ...`:

  * Ensures initialization happens **only once**
  * Prevents wiping chat history on every rerun

---

## 2️⃣ Display Chat History

```python
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.write(message["content"])
```

* 🔁 Loops through all stored messages
* 💬 `st.chat_message(message["role"])`:

  * Automatically renders the correct chat bubble
* 📜 This redraws the **entire conversation** on every rerun

---

## 3️⃣ Handle New Messages

```python
if prompt := st.chat_input("What would you like to know?"):
    st.session_state.messages.append({"role": "user", "content": prompt})
```

* 🧠 **Walrus operator (`:=`)**:

  * Assigns and checks the input in one line

* ➕ `.append(...)`:

  * Adds the user message to session state
  * Must happen **before** rendering the response

---

## 4️⃣ Generate and Store Assistant Response

```python
with st.chat_message("assistant"):
    response = call_llm(prompt)
    st.write(response)

st.session_state.messages.append({"role": "assistant", "content": response})
```

* 🤖 `call_llm(...)`:

  * Calls **Snowflake Cortex** using `ai_complete()`

* 💾 Second `.append(...)`:

  * Stores the assistant’s reply
  * Completes one full conversation turn

### ❓ Why SQL-based `ai_complete()`?

* ✅ Works in **all environments**:

  * Streamlit in Snowflake
  * Streamlit Community Cloud
  * Local development

* ❌ Python SDK can fail due to SSL issues

* ✔️ SQL-based approach is **universally reliable**

---

## 🖥️ Final Result

When this code runs, you now have:

* 💬 A real chatbot UI
* 🧠 Memory across interactions
* 📜 A visible conversation history

---

## ⚠️ What’s Missing? (Room for Improvement)

While functional, this chatbot has limitations:

### Current Limitations

* ❌ No visual feedback while waiting
* ❌ Responses appear all at once
* ❌ No conversation statistics
* ❌ No reset / clear chat button
* ❌ Generic appearance
* ❌ No error handling

🚀 Don’t worry — **all of this will be fixed** in upcoming lessons as we move toward a **production‑ready chatbot**.

---

## 📚 Resources

* 📘 **Build Conversational Apps**
* 📘 **Cortex Complete Function**


