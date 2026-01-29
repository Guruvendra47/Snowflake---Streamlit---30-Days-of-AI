# 🗓️ Day 12 — Streaming Responses

## 🎯 Goal of the Day

Building on **Day 11** (conversation history and sidebar stats), today we add **streaming responses** to create a more dynamic and responsive chat experience.

Instead of waiting for the full response, users will:

* ⌨️ See the AI’s reply appear **word‑by‑word**
* ⚡ Experience faster *perceived* responses
* 💬 Get a chat UI that feels like modern messaging apps

---

## 🧩 See the Code

```python
import streamlit as st
import json
from snowflake.snowpark.functions import ai_complete
import time

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

st.title(":material/chat: Chatbot with Streaming")

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
    
    # Build the full conversation history for context
    conversation = "\n\n".join([
        f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
        for msg in st.session_state.messages
    ])
    full_prompt = f"{conversation}\n\nAssistant:"
    
    # Generate stream
    def stream_generator():
        response_text = call_llm(full_prompt)
        for word in response_text.split(" "):
            yield word + " "
            time.sleep(0.02)
    
    # Display assistant response with streaming
    with st.chat_message("assistant"):
        with st.spinner("Processing"):
            response = st.write_stream(stream_generator)
    
    # Add assistant response to state
    st.session_state.messages.append({"role": "assistant", "content": response})
    st.rerun()  # Force rerun to update sidebar stats

st.divider()
st.caption("Day 12: Streaming Responses | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step-by-Step

Day 12 focuses on **one key enhancement**: **streaming responses**.

---

## ✅ What’s Kept from Previous Days

* 👋 Welcome message on initialization (Day 11)
* 🧠 Full conversation history passed to the LLM (Day 11)
* 📊 Sidebar with conversation statistics (Day 11)
* 🧹 Clear History button (Day 11)
* 💬 Chat UI with `st.chat_message()` (Days 8–11)

---

## 🆕 What’s New: Streaming Responses

```python
# Build the full conversation history for context
conversation = "\n\n".join([
    f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
    for msg in st.session_state.messages
])
full_prompt = f"{conversation}\n\nAssistant:"

# Generate stream
def stream_generator():
    response_text = call_llm(full_prompt)
    for word in response_text.split(" "):
        yield word + " "
        time.sleep(0.02)

# Display assistant response with streaming
with st.chat_message("assistant"):
    with st.spinner("Processing"):
        response = st.write_stream(stream_generator)

# Add assistant response to state
st.session_state.messages.append({"role": "assistant", "content": response})
st.rerun()
```

---

## 🔄 What Changed from Day 11

* 📦 Conversation building happens **before streaming**
* 🧵 Custom Python **generator function** simulates streaming
* 🤖 `call_llm(full_prompt)` still uses SQL‑based `ai_complete()`
* 📝 Response is split into words and yielded one at a time
* ⏱️ `time.sleep(0.02)` adds a smooth typing effect
* 🌀 `st.write_stream()` renders the stream in real‑time
* 🔁 Final response text is stored in Session State
* 🔄 `st.rerun()` updates sidebar stats immediately

---

## 💡 Why This Approach Works Well

* 🌍 **Universal compatibility**: Works in SiS, Community Cloud, and locally
* 🚀 **Better UX**: Users see instant progress
* ⚡ **Perceived speed**: Streaming *feels* faster than waiting
* 💬 **Natural conversation**: Mimics human typing
* 🧩 **Simple logic**: Easy to understand and customize

---

## 🖥️ Final Result

When this code runs, you will have:

* 💬 A chatbot with full conversation memory
* 📊 Live‑updating sidebar statistics
* ⌨️ Real‑time streaming responses
* ✨ A modern, production‑style chat experience

---

## 📚 Resources

* 📘 **Cortex Complete Streaming**
* 📘 **st.write_stream Documentation**
* 📘 **Build Conversational Apps**


