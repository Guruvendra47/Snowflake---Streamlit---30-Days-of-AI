# 🗓️ Day 8 — Meet the Chat Elements

## 🎯 Goal of the Day

Welcome to **Week 2**! Let’s reflect on what we’ve learned so far.

In **Week 1**, you built **linear apps**:

```
Input → Process → Output
```

Now, we shift gears and start building **chat-based applications**. This is where **Streamlit really shines**, but it introduces a new challenge: the app will eventually need to **remember context**.

For **Day 8**, our goal is to focus **only on the chat user interface (UI)**:

* 💬 Render chat-style message bubbles
* ⌨️ Accept user input via a chat input box
* 🧱 Build the **visual skeleton of a chatbot**

⚠️ This is **not yet a full chatbot**. There is **no memory** and **no AI API call** yet — just the UI foundation.

---

## 🧩 See the Code

```python
import streamlit as st

st.title(":material/chat: Meet the Chat Elements")

# 1. Displaying Static Messages
with st.chat_message("user"):
    st.write("Hello! Can you explain what Streamlit is?")

with st.chat_message("assistant"):
    st.write("Streamlit is an open-source Python framework for building data apps.")
    st.bar_chart([10, 20, 30, 40]) # You can even put charts inside chat messages!

# 2. Chat Input
prompt = st.chat_input("Type a message here...")

# 3. Reacting to Input
if prompt:
    # Display the user's new message
    with st.chat_message("user"):
        st.write(prompt)
    
    # Display a mock assistant response
    with st.chat_message("assistant"):
        st.write(f"You just said:\n\n '{prompt}' \n\n(I don't have memory yet!)")

# Footer
st.divider()
st.caption("Day 8: Meet the Chat Elements | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step-by-Step

We will use **two core chat-specific commands**:

* `st.chat_message()` → for chat bubbles
* `st.chat_input()` → for the chat input box

---

## 1️⃣ Visualizing Static Messages

```python
with st.chat_message("user"):
    st.write("Hello! Can you explain what Streamlit is?")

with st.chat_message("assistant"):
    st.write("Streamlit is an open-source Python framework for building data apps.")
    st.bar_chart([10, 20, 30, 40])
```

* 💬 `st.chat_message("role")`:

  * Creates a chat bubble
  * Streamlit automatically assigns **avatar and alignment** based on the role

    * `"user"` → right-aligned
    * `"assistant"` → left-aligned

* 📊 `st.bar_chart(...)`:

  * Shows that chat messages can contain **rich content**
  * Not limited to plain text (charts, images, tables all work)

---

## 2️⃣ The Chat Input Widget

```python
prompt = st.chat_input("Type a message here...")
```

* ⌨️ `st.chat_input(...)`:

  * Creates a chat-style input box pinned to the **bottom of the screen**
  * Automatically manages layout so it doesn’t overlap messages

* 🧠 `prompt`:

  * Stores the user’s message
  * Remains `None` until the user presses **Enter**

---

## 3️⃣ Reacting to Input (The “Glitchy” Loop)

```python
if prompt:
    with st.chat_message("user"):
        st.write(prompt)

    with st.chat_message("assistant"):
        st.write(f"You just said:\n\n '{prompt}' \n\n(I don't have memory yet!)")
```

* 🔘 `if prompt:`:

  * Ensures this logic runs **only after** the user submits a message

* 🔁 User message is echoed back immediately

* 🤖 Assistant sends a **mock response** (no AI yet)

---

## ⚠️ Important Observation (Day 8)

If you:

1. Type a message → it appears
2. Type a second message → the **first message disappears** ❌

### ❓ Why does this happen?

Streamlit **re-runs the entire script** on every interaction.
Since we are **not storing chat history**, previous messages are lost.

---

## 🚀 What’s Coming Next

To fix the disappearing messages, we need the **most important concept in Streamlit app development**:

> 🧠 **Session State**

That’s exactly what we’ll cover in **Day 9**.

---

## 📚 Resources

* 📘 **st.chat_message Documentation**
* 📘 **st.chat_input Documentation**
* 📘 **Build a Basic LLM Chat App**


