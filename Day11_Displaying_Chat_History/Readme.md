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

## 🧠 The Problem from Day 10

In **Day 10**, messages were stored in **Streamlit session state**, so they appeared correctly in the UI.

❌ **But the LLM never saw that history.**

### Example Conversation — *Day 10 Behavior*

```
User: What's the capital of France?
AI: Paris

User: What's the population?
AI: What location are you asking about? ❌
```

### 🚨 Why this is BAD

* The chat UI **looks** like it remembers
* The LLM **actually has zero memory**
* This creates a **fake chat experience**

### ❌ Root Cause

Only the **current prompt** was sent to the LLM:

```python
response = call_llm(prompt)  # Only sends current message!
```

> ⚠️ **Brutal truth**: Storing messages in session state alone does NOTHING for LLM memory.

---

## ✅ The Solution (Day 11)

In **Day 11**, we fix this properly by sending the **entire conversation history** to the LLM.

### Same Conversation — *Day 11 Behavior*

```
User: What's the capital of France?
AI: Paris

User: What's the population?
AI: Paris has approximately 2.1 million people in the city proper... ✅
```

🎯 **Now the AI remembers context and answers follow-ups correctly.**

---

## 🔑 How It Works (High Level)

We **rebuild the entire conversation** and send it as one prompt:

```python
# Build the full conversation history for context
conversation = "\n\n".join([
    f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
    for msg in st.session_state.messages
])
full_prompt = f"{conversation}\n\nAssistant:"

response = call_llm(full_prompt)  # Sends entire conversation!
```

> 💡 **Interview gold**: LLMs are stateless. Memory must be injected every request.



## 🧩 How It Works: Step-by-Step



### 1️⃣ Initialize with a Welcome Message

```python
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "assistant", "content": "Hello! I'm your AI assistant. How can I help you today?"}
    ]
```

### Why this matters

* 👋 **Pre-populated list**: Chat doesn’t start empty
* 🤝 **Better UX**: Feels welcoming and alive
* 🧠 **Consistent baseline** for reset

---

### 2️⃣ Sidebar Statistics and Controls

```python
with st.sidebar:
    st.header("Conversation Stats")
    user_msgs = len([m for m in st.session_state.messages if m["role"] == "user"])
    assistant_msgs = len([m for m in st.session_state.messages if m["role"] == "assistant"])
    st.metric("Your Messages", user_msgs)
    st.metric("AI Responses", assistant_msgs)
```

### Key concepts

* 🧩 **List comprehension** filters messages by role
* 🧮 `st.metric()` displays live counters

> 📌 **Brutal mentor note**: This is state-derived UI — always recomputed, never stored.

---

### 3️⃣ Clear History Button

```python
if st.button("Clear History"):
    st.session_state.messages = [
        {"role": "assistant", "content": "Hello! I'm your AI assistant. How can I help you today?"}
    ]
    st.rerun()
```

### Why this matters

* 🔄 **Resets to known-good state**
* 🧹 Prevents stale context
* ⚡ `st.rerun()` refreshes UI immediately

---

### 4️⃣ Enhanced Message Display

```python
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])
```

### Why `st.markdown()`

* ✨ Supports rich formatting
* 📋 Bullet points, **bold**, *italics*
* Better than `st.write()` for chat

---

### 5️⃣ Loading Indicator with `st.spinner`

```python
with st.spinner("Thinking..."):
    response = call_llm(full_prompt)
```

### Why this matters

* ⏳ Shows AI is working
* 🧠 Prevents “frozen app” confusion
* 📌 Removed in Day 12 when streaming is added

---

### 6️⃣ Passing Conversation History to the LLM (CRITICAL)

```python
conversation = "\n\n".join([
    f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
    for msg in st.session_state.messages
])
full_prompt = f"{conversation}\n\nAssistant:"

response = call_llm(full_prompt)
```

### Why this matters

* 🧠 **This is the memory**
* Without it → AI forgets everything
* With it → Follow-up questions work

> 🔥 **Brutal truth**: Session state ≠ memory. Prompt = memory.

---

### 7️⃣ Updating Sidebar Stats with `st.rerun()`

```python
st.session_state.messages.append({"role": "assistant", "content": response})
st.rerun()
```

### Why this matters

* 📊 Sidebar updates instantly
* ❌ Without it → metrics lag one step
* ✅ Better UX and consistency

---

## 🎯 Final Result

When this code runs, you get:

* 👋 A welcoming initial message
* 📊 Live-updating conversation stats
* 🧹 Clear History reset
* 🧠 **TRUE conversation memory**
* 💬 Natural follow-up question handling

---

## 📚 Resources

* 📘 `st.metric` Documentation
* 📘 `st.rerun` Documentation
* 🧠 Streamlit Session State Management




