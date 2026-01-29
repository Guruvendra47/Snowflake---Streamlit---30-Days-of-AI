# 🗓️ Day 13 — Adding a System Prompt  

---

## 📌 What You’re Learning Today

On **Day 13**, you extend **Day 12’s streaming chatbot** by adding **System Prompts**.

### ❓ What is the point?
A **system prompt** controls the *personality and behavior* of the AI.

✅ Same model  
✅ Same user question  
❌ Completely different answers — **based on character**

Examples:
- 🏴‍☠️ Pirate  
- 👨‍🏫 Teacher  
- 🎤 Comedian  
- 🤖 Robot  

> 💡 **Interview-ready truth**:  
> A *system prompt* is how you steer an LLM’s tone, behavior, and constraints **without changing the model itself**.

---

## 🧠 Key Concept (DO NOT MISS THIS)

⚠️ **System Prompt ≠ User Prompt**

| Type | Purpose |
|----|----|
| System Prompt | Defines **who the AI is** |
| User Prompt | Defines **what the user wants** |

👉 System prompt always has **higher priority**.

---

## 🧩 Full Working Code (UNCHANGED)

> ❌ Do NOT modify this code  
> ✅ This is production-valid Streamlit + Snowflake Cortex code

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

st.title(":material/chat: Customizable Chatbot")

# Initialize system prompt if not exists
if "system_prompt" not in st.session_state:
    st.session_state.system_prompt = "You are a helpful pirate assistant named Captain Starlight. You speak with pirate slang, use nautical metaphors, and end sentences with 'Arrr!' when appropriate. Be helpful but stay in character."

# Initialize messages with a personality-appropriate greeting
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "assistant", "content": "Ahoy! Captain Starlight here, ready to help ye navigate the high seas of knowledge! Arrr!"}
    ]

# Sidebar configuration
with st.sidebar:
    st.header(":material/theater_comedy: Bot Personality")
    
    # Preset personalities
    st.subheader("Quick Presets")
    col1, col2 = st.columns(2)
    
    with col1:
        if st.button(":material/sailing: Pirate"):
            st.session_state.system_prompt = "You are a helpful pirate assistant named Captain Starlight. You speak with pirate slang, use nautical metaphors, and end sentences with 'Arrr!' when appropriate."
            st.rerun()
    
    with col2:
        if st.button(":material/school: Teacher"):
            st.session_state.system_prompt = "You are Professor Ada, a patient and encouraging teacher. You explain concepts clearly, use examples, and always check for understanding."
            st.rerun()
    
    col3, col4 = st.columns(2)
    
    with col3:
        if st.button(":material/mood: Comedian"):
            st.session_state.system_prompt = "You are Chuckles McGee, a witty comedian assistant. You love puns, jokes, and humor, but you're still genuinely helpful. You lighten the mood while providing useful information."
            st.rerun()
    
    with col4:
        if st.button(":material/smart_toy: Robot"):
            st.session_state.system_prompt = "You are UNIT-7, a helpful robot assistant. You speak in a precise, logical manner. You occasionally reference your circuits and processing units."
            st.rerun()
    
    st.divider()
    
    st.text_area(
        "System Prompt:",
        height=200,
        key="system_prompt"
    )
    
    st.divider()
    
    # Conversation stats
    st.header("Conversation Stats")
    user_msgs = len([m for m in st.session_state.messages if m["role"] == "user"])
    assistant_msgs = len([m for m in st.session_state.messages if m["role"] == "assistant"])
    st.metric("Your Messages", user_msgs)
    st.metric("AI Responses", assistant_msgs)
    
    if st.button("Clear History"):
        st.session_state.messages = [
            {"role": "assistant", "content": "Ahoy! Captain Starlight here, ready to help ye navigate the high seas of knowledge! Arrr!"}
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
    
    # Generate and display assistant response with streaming
    with st.chat_message("assistant"):
        # Custom generator for reliable streaming
        def stream_generator():
            # Build the full conversation history for context
            conversation = "\n\n".join([
                f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
                for msg in st.session_state.messages
            ])
            
            # Create prompt with system instruction
            full_prompt = f"""{st.session_state.system_prompt}

Here is the conversation so far:
{conversation}

Respond to the user's latest message while staying in character."""
            
            response_text = call_llm(full_prompt)
            for word in response_text.split(" "):
                yield word + " "
                time.sleep(0.02)
        
        with st.spinner("Processing"):
            response = st.write_stream(stream_generator)
    
    # Add assistant response to state
    st.session_state.messages.append({"role": "assistant", "content": response})
    st.rerun()  # Force rerun to update sidebar stats

st.divider()
st.caption("Day 13: Adding a System Prompt | 30 Days of AI")
```


## 🧠 How It Works: Step-by-Step

Day 13 **keeps everything from Day 12** (streaming with a custom generator) and **adds system prompt customization** to control chatbot personalities.

---

## ✅ What’s Kept from Day 12

The following features are **unchanged** and carried forward:

* 🔁 **Streaming responses** using `st.write_stream()` *(Day 12)*
* ⚙️ **Custom generator** for reliable streaming *(Day 12)*
* ⏳ **Spinner** showing `Processing` status *(Day 12)*
* 🧠 **Full conversation history** passed to the LLM *(Day 11)*
* 📊 **Sidebar conversation stats** *(Day 11)*
* 🧹 **Clear History** button *(Day 11)*
* 👋 **Welcome message** *(Day 11)*

> 📌 **Brutal truth**: This confirms you are **layering features correctly**, not rewriting the app every day.


## 🆕 What’s New: System Prompts & Personalities


## 1️⃣ Initialize System Prompt and Messages Early

```python
# Initialize system prompt if not exists
if "system_prompt" not in st.session_state:
    st.session_state.system_prompt = "You are a helpful pirate assistant named Captain Starlight. You speak with pirate slang, use nautical metaphors, and end sentences with 'Arrr!' when appropriate. Be helpful but stay in character."

# Initialize messages with a personality-appropriate greeting
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "assistant", "content": "Ahoy! Captain Starlight here, ready to help ye navigate the high seas of knowledge! Arrr!"}
    ]
```

### 🔍 Why this matters

* ⏱️ **Early initialization**: Happens before sidebar widgets are created
* 💾 **Session State**: Ensures the system prompt persists across reruns
* 🎭 **Personality-appropriate greeting**: Welcome message matches the default persona

> ⚠️ **Brutal mentor note**: If you initialize this late, presets will break or reset unexpectedly.

---

## 2️⃣ Preset Personality Buttons (Top of Sidebar)

```python
with st.sidebar:
    st.header(":material_theater_comedy: Bot Personality")

    # Preset personalities
    st.subheader("Quick Presets")
    col1, col2 = st.columns(2)

    with col1:
        if st.button(":material_sailing: Pirate"):
            st.session_state.system_prompt = "You are a helpful pirate assistant..."
            st.rerun()

    with col2:
        if st.button(":material_library_books: Teacher"):
            st.session_state.system_prompt = "You are Professor Ada, a patient and encouraging teacher..."
            st.rerun()

    col3, col4 = st.columns(2)

    with col3:
        if st.button(":material_mood: Comedian"):
            st.session_state.system_prompt = "You are Chuckles McGee, a witty comedian assistant..."
            st.rerun()

    with col4:
        if st.button(":material_smart_toy: Robot"):
            st.session_state.system_prompt = "You are UNIT-7, a helpful robot assistant..."
            st.rerun()
```

### 🔍 Why this matters

* 📍 **Preset buttons first**: Better UX — users see options immediately
* ⚡ **Quick switches**: One click = instant personality change
* 🔄 `st.rerun()`: Forces UI refresh so the text area updates
* 🎭 **Four personas**:

  * 🏴‍☠️ Pirate — *Captain Starlight*
  * 👨‍🏫 Teacher — *Professor Ada*
  * 🎤 Comedian — *Chuckles McGee*
  * 🤖 Robot — *UNIT-7*

> 💡 **Interview-ready line**: “We dynamically control LLM behavior using system prompts stored in Streamlit session state.”

---

## 3️⃣ The System Prompt Text Area (Below Presets)

```python
st.divider()

st.text_area(
    "System Prompt:",
    height=200,
    key="system_prompt"
)
```

### 🔍 Why this matters

* 🔑 `key="system_prompt"`: Auto-binds to `st.session_state.system_prompt`
* 🚫 **No conflict warning**: Avoids using both `key` and `value`
* ✍️ **Editable**: Users can tweak presets or write custom prompts
* 📐 **Placed below presets**: Logical flow — select → edit

> ⚠️ **Brutal rule**: Never mix `key` and `value` in Streamlit inputs unless you enjoy bugs.

---

## 4️⃣ Injecting the System Prompt with Streaming

```python
# Custom generator for reliable streaming
def stream_generator():
    # Build the full conversation history for context
    conversation = "\n\n".join([
        f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
        for msg in st.session_state.messages
    ])

    # Create prompt with system instruction
    full_prompt = f"""{st.session_state.system_prompt}

Here is the conversation so far:
{conversation}

Respond to the user's latest message while staying in character."""

    response_text = call_llm(full_prompt)
    for word in response_text.split(" "):
        yield word + " "
        time.sleep(0.02)

with st.spinner("Processing"):
    response = st.write_stream(stream_generator)
```

### 🔍 Why this matters

* 🧠 **System prompt first**: Sets behavior before conversation context
* 🎭 **“Stay in character”**: Reinforces persona consistency
* 🔁 **Custom generator**: Simulates streaming word-by-word
* 🧩 `call_llm()`: Uses SQL-based `ai_complete()` for compatibility
* ✂️ `split(" ")`: Splits on spaces only (not all whitespace)
* ⏳ **Spinner wrapper**: Visual feedback before streaming starts

> ⚠️ **Brutal clarity**: This is simulated streaming — not native token streaming. Still valid for UI.

---

## 5️⃣ Why System Prompts Matter (Memorize This)

System prompts are powerful because they:

* 🎯 **Define behavior** — how the LLM responds
* 🎨 **Set tone & style** — formal, casual, humorous, technical
* 🎭 **Enable role-playing** — domain-specific assistants
* 🚧 **Provide constraints** — topics, formats, rules

> 🧠 **Burn this in memory**: Same model + same question ≠ same answer when system prompts differ.

---

## 🚀 Final Result

When this code runs, you get:

* ✅ A chatbot with a **personality selector** in the sidebar
* 🔁 Live **streaming responses**
* 🎭 Multiple personas controlled by **system prompts**

👉 Try asking the **same question** with different personas and observe the behavior change.

---

## 📚 Resources

* 📖 Prompt Engineering Guide
* 📘 `st.text_area` Documentation
* 🧠 System Prompts Best Practices

---

