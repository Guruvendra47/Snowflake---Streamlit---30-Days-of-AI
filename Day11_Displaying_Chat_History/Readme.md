# 🗓️ Day 11 — Displaying Chat History

## 🎯 Goal of the Day

For today's challenge, our goal is to **enhance our chatbot with better history management**.

We will:

* 👋 Add a **welcome message**
* 📊 Display **conversation statistics** in the sidebar
* 🧹 Provide a way to **clear chat history**
* 🧠 Pass the **full conversation context** to the LLM

Once completed, we’ll have a **more polished chatbot experience** with visible conversation tracking and true memory.

> **Note:** We also use `st.rerun()` to ensure sidebar stats update immediately after each response.

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
    if isinstance(response_json, dict):
        return response_json.get("choices", [{}])[0].get("messages", "")
    return str(response_json)

st.title(":material/chat: Chatbot with History")

# Initialize messages
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "assistant", "content": "Hello! I'm your AI assistant. How can I help you today?"}
    ]

# Sidebar to show conversation stats
with st.sidebar:
    st.header("Conversation Stats")
    user_msgs = len([m for m in st.session_state.messages if m["role"] == "user"])
    assistant_msgs = len([m for m in st.session_state.messages if m["role"] == "assistant"])
    st.metric("Your Messages", user_msgs)
    st.metric("AI Responses", assistant_msgs)
    
    if st.button("Clear History"):
        st.session_state.messages = [
            {"role": "assistant", "content": "Hello! I'm your AI assistant. How can I help you today?"}
        ]
        st.rerun()

# Display all messages from history
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# Chat input
if prompt := st.chat_input("Type your message..."):
    # Add and display user message
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)
    
    # Generate and display assistant response
    with st.chat_message("assistant"):
        with st.spinner("Thinking..."):
            # Build the full conversation history for context
            conversation = "\n\n".join([
                f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
                for msg in st.session_state.messages
            ])
            full_prompt = f"{conversation}\n\nAssistant:"
            
            response = call_llm(full_prompt)
        st.markdown(response)
    
    # Add assistant response to state
    st.session_state.messages.append({"role": "assistant", "content": response})
    st.rerun()

st.divider()
st.caption("Day 11: Displaying Chat History | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 The Problem from Day 10

In **Day 10**, messages were stored and displayed, but the **LLM could not see chat history**.

**Example (Day 10 behavior):**

```
User: What's the capital of France?
AI: Paris

User: What's the population?
AI: What location are you asking about? ❌
```

Even though the UI showed history, only the **current prompt** was sent:

```python
response = call_llm(prompt)
```

---

## ✅ The Solution (Day 11)

We now send the **entire conversation history** to the LLM:

```python
conversation = "\n\n".join([
    f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
    for msg in st.session_state.messages
])
full_prompt = f"{conversation}\n\nAssistant:"

response = call_llm(full_prompt)
```

**Result:**

```
User: What's the capital of France?
AI: Paris

User: What's the population?
AI: Paris has approximately 2.1 million people... ✅
```

---

## 🧠 How It Works: Step-by-Step

### 1️⃣ Initialize with a Welcome Message

* Starts the chat with an assistant greeting
* Makes the app feel friendly and alive

---

### 2️⃣ Sidebar Statistics and Controls

* Uses list comprehensions to count messages
* Displays metrics using `st.metric()`

---

### 3️⃣ Clear History Button

* Resets chat back to the welcome message
* Uses `st.rerun()` to immediately refresh UI

---

### 4️⃣ Enhanced Message Display

* Uses `st.markdown()` instead of `st.write()`
* Enables rich formatting in messages

---

### 5️⃣ Loading Indicator with `st.spinner()`

* Shows the user the AI is working
* Prevents the app from feeling frozen

---

### 6️⃣ Passing Conversation History to the LLM

* Rebuilds the full conversation every turn
* Sends it as context to Snowflake Cortex
* Enables **true conversational memory**

---

### 7️⃣ Updating Sidebar Stats with `st.rerun()`

* Forces immediate rerun after assistant response
* Sidebar metrics update instantly

---

## 🖥️ Final Result

When this code runs, you will see:

* 👋 A chatbot with a welcome message
* 📊 Live-updating conversation statistics
* 🧹 A clear history button
* 🧠 True conversational memory

---

## 📚 Resources

* 📘 **st.metric Documentation**
* 📘 **st.rerun Documentation**
* 📘 **Session State Management**


