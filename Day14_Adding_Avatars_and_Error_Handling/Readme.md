# 🗓️ Day 14 — Adding Avatars and Error Handling

## 🎯 Goal of the Day

Building on previous chatbots, today we add **visual polish** with avatars and **robust error handling** to move toward a **production‑ready chatbot**.

With these changes:

* 🎭 Users can **personalize chat avatars**
* 🛡️ The app **handles API failures gracefully**
* 🚫 Errors no longer crash the application
* 💼 The chatbot feels reliable and professional

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

st.title(":material/account_circle: Adding Avatars and Error Handling")

# Initialize system prompt if not exists
if "system_prompt" not in st.session_state:
    st.session_state.system_prompt = "You are a helpful assistant."

# Initialize messages
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "assistant", "content": "Hello! I'm your AI assistant. How can I help you today?"}
    ]

# Sidebar configuration
with st.sidebar:
    st.header(":material/settings: Settings")
    
    # Avatar customization
    st.subheader(":material/palette: Avatars")
    user_avatar = st.selectbox(
        "Your Avatar:",
        ["👤", "🧑‍💻", "👨‍🎓", "👩‍🔬", "🦸", "🧙"],
        index=0
    )
    
    assistant_avatar = st.selectbox(
        "Assistant Avatar:",
        ["🤖", "🧠", "✨", "🎯", "💡", "🌟"],
        index=0
    )
    
    st.divider()
    
    # System prompt
    st.subheader(":material/description: System Prompt")
    st.text_area(
        "Customize behavior:",
        height=100,
        key="system_prompt",
        help="Define how the AI should behave and respond"
    )
    
    st.divider()
    
    # Debug toggle to simulate errors
    st.subheader(":material/bug_report: Debug Mode")
    simulate_error = st.checkbox(
        "Simulate API Error",
        value=False,
        help="Enable this to test the error handling mechanism"
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
            {"role": "assistant", "content": "Hello! I'm your AI assistant. How can I help you today?"}
        ]
        st.rerun()

# Display all messages from history with custom avatars
for message in st.session_state.messages:
    avatar = user_avatar if message["role"] == "user" else assistant_avatar
    with st.chat_message(message["role"], avatar=avatar):
        st.markdown(message["content"])

# Chat input
if prompt := st.chat_input("Type your message..."):
    # Add and display user message
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user", avatar=user_avatar):
        st.markdown(prompt)
    
    # Generate response with error handling
    with st.chat_message("assistant", avatar=assistant_avatar):
        try:
            # Simulate error if debug mode is enabled
            if simulate_error:
                raise Exception("Simulated API error: Service temporarily unavailable (429)")
            
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

Respond to the user's latest message."""
                
                response_text = call_llm(full_prompt)
                for word in response_text.split(" "):
                    yield word + " "
                    time.sleep(0.02)
            
            with st.spinner("Processing"):
                response = st.write_stream(stream_generator)
            
            # Add assistant response to state
            st.session_state.messages.append({"role": "assistant", "content": response})
            st.rerun()
            
        except Exception as e:
            error_message = f"I encountered an error: {str(e)}"
            st.error(error_message)
            st.info(":material/lightbulb: **Tip:** This might be a temporary issue. Try again in a moment, or rephrase your question.")

st.divider()
st.caption("Day 14: Adding Avatars and Error Handling | 30 Days of AI")
```
# 📘 Explanation:

## 🧠 How It Works: Step-by-Step

Day 14 **keeps everything from previous days** and adds two production-grade features:

* 🎭 **Custom avatars** (visual personalization)
* 🚨 **Robust error handling** (stability & professionalism)

This is where your chatbot stops feeling like a demo and starts feeling like a **real product**.

---

## ✅ What’s Kept from Previous Days

The following functionality is reused **unchanged**:

* 🔁 **Streaming responses with custom generator** *(Day 12)*
* ⏳ **Spinner showing “Processing” status** *(Day 12)*
* 🎭 **System prompt customization** *(Day 13)*
* 🧠 **Full conversation history** *(Day 11)*
* 📊 **Sidebar with conversation stats** *(Day 11)*
* 🧹 **Clear History button** *(Day 11)*
* 👋 **Welcome message** *(Day 11)*
* 💬 **Chat interface using `st.chat_message()`** *(Days 8–11)*

> 📌 **Brutal mentor note**: This proves architectural discipline — features are layered, not hacked in.

## 🆕 What’s New: Avatars & Error Handling


### 1️⃣ Avatar Configuration

```python
USER_AVATAR = ":material_account_circle:"
ASSISTANT_AVATAR = ":material_smart_toy:"

user_avatar = st.selectbox(
    "Your Avatar:",
    [":material_account_circle:", "🧑‍💻", "👨‍🎓", "👩‍🔬", "🦸", "🧙"],
    index=0
)
```

### 🔍 Why this matters

* 🎨 **Default avatars**: Constants ensure consistent styling
* 🧍 **User choice**: Users personalize their chat experience
* 🤖 **Both avatars**: User and assistant avatars are independent

> ⚠️ **Brutal truth**: UX polish matters. Interviewers notice this immediately.

---

### 2️⃣ Debug Mode Toggle

```python
st.subheader(":material_bug_report: Debug Mode")
simulate_error = st.checkbox(
    "Simulate API Error",
    value=False,
    help="Enable this to test the error handling mechanism"
)
```

### 🔍 Why this matters

* 🧪 **Testing tool**: Trigger failures without waiting for real ones
* 🎓 **Educational value**: Demonstrates resilience clearly
* 🔕 **Default OFF**: Normal behavior unless testing
* 💬 **Help text**: Explains purpose to users

> 💡 **Interview line**: “We built a debug toggle to validate error handling paths safely.”

---

### 3️⃣ Using Avatars in Chat Messages

```python
for message in st.session_state.messages:
    avatar = user_avatar if message["role"] == "user" else assistant_avatar
    with st.chat_message(message["role"], avatar=avatar):
        st.markdown(message["content"])
```

### 🔍 Why this matters

* 🔄 **Dynamic avatar selection**: Based on message role
* 🧩 `avatar=` parameter:

  * Emojis
  * Image URLs
  * Image file paths

> 📌 **Brutal clarity**: This is presentation logic — cleanly separated from LLM logic.

---

### 4️⃣ Error Handling with `try / except` and Streaming

```python
try:
    # Simulate error if debug mode is enabled
    if simulate_error:
        raise Exception("Simulated API error: Service temporarily unavailable (429)")

    # Custom generator for reliable streaming
    def stream_generator():
        # Build the full conversation history for context
        conversation = "\n\n".join([
            f"{'User' if msg['role'] == 'user' else 'Assistant'}: {msg['content']}"
            for msg in st.session_state.messages
        ])
        full_prompt = f"{conversation}\n\nAssistant:"

        response_text = call_llm(full_prompt)
        for word in response_text.split():
            yield word + " "
            time.sleep(0.02)

    with st.spinner("Processing"):
        response = st.write_stream(stream_generator)

    st.session_state.messages.append({"role": "assistant", "content": response})

except Exception as e:
    error_message = f"I encountered an error: {str(e)}"
    st.error(error_message)
    st.info(":material_lightbulb: **Tip:** This might be a temporary issue. Try again in a moment, or rephrase your question.")
```

### 🔍 Why this matters

* 🛡️ **`try / except` block**: Catches API, network, or logic failures
* 🧪 **Simulated error**: Controlled testing via debug mode
* 🔴 `st.error()`:

  * Red error box
  * Clear failure message
* 🔵 `st.info()`:

  * Helpful guidance
  * Preserves user trust
* 🔁 **Custom generator**: Streaming remains intact
* ⏳ **Spinner wrapper**: Visual feedback during processing

> ⚠️ **Brutal truth**: No error handling = amateur app. Period.

---

## 5️⃣ Why Error Handling Matters (Non-Negotiable)

🚨 **LLM APIs WILL fail in production.**

Common failure reasons:

* ⛔ **Rate limiting** — too many requests
* 🌐 **Network issues** — transient outages
* 🧠 **Model overload** — high demand timeouts
* 🚫 **Invalid input** — safety or validation errors

### By handling errors gracefully, we:

* ✅ Prevent app crashes
* 📢 Keep users informed
* 🧭 Provide actionable suggestions
* 🧹 Avoid storing failed responses in history

> 🧠 **Burn this in memory**: Stability is a feature.

---

## 🚀 Final Result

When this code runs, you get:

* 🎭 A **visually polished chatbot** with avatars
* 🚨 **Robust error handling**
* 🧪 A **debug mode** for testing failures
* 💼 A professional, production-ready UX

---

## 🧪 Try It Out

1. Enable **“Simulate API Error”** in the sidebar
2. Send a message
3. Observe:

   * 🔴 Red error message
   * 💡 Helpful tip
   * ✅ App continues running
4. Disable the checkbox and retry

> 📌 **Key point**: The app fails gracefully — it does NOT crash.

---

## 📚 Resources

* 📘 `st.chat_message` Avatar Parameter
* 📕 `st.error` Documentation
* 🧠 Python Exception Handling




