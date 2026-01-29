# 🗓️ Day 9 — Understanding Session State

## 🎯 Goal of the Day

For today's challenge, our goal is to solve the **Amnesia Problem** in Streamlit apps.

We need to understand:

* ❓ Why **standard Python variables reset** on every interaction
* 🧠 How **Session State** preserves values across reruns
* 🔢 How to build a counter that actually **remembers its value** when buttons are clicked

By the end of this day, you will clearly see **what goes wrong** and **how to fix it correctly**.

---

## 🧩 See the Code

```python
import streamlit as st

st.title(":material/memory: Understanding Session State")

st.warning("**Instructions:** Try clicking the + and - buttons in both columns to see the difference.")

# Create two columns for side-by-side comparison
col1, col2 = st.columns(2)

# --- COLUMN 1: THE WRONG WAY ---
with col1:
    st.header(":material/cancel: Standard Variable")
    st.write("This resets on every click.")

    # This line runs every time you click ANY button on the page.
    # It effectively erases your progress immediately.
    count_wrong = 0
    
    # We use nested columns here to put the + and - buttons side-by-side
    subcol_left, subcol_right = st.columns(2)
    
    with subcol_left:
        # Note: We must give every button a unique 'key'
        if st.button(":material/add:", key="std_plus"):
            count_wrong += 1

    with subcol_right:
        if st.button(":material/remove:", key="std_minus"):
            count_wrong -= 1
    
    st.metric("Standard Count", count_wrong)
    st.caption("It never gets past 1 or -1 because `count_wrong` resets to 0 before the math happens.")


# --- COLUMN 2: THE RIGHT WAY ---
with col2:
    st.header(":material/check_circle: Session State")
    st.write("This memory persists.")

    # 1. Initialization: Create the key only if it doesn't exist yet
    if "counter" not in st.session_state:
        st.session_state.counter = 0
    
    # We use nested columns here as well
    subcol_left_2, subcol_right_2 = st.columns(2)

    with subcol_left_2:
        # 2. Modification: Update the dictionary value (Increment)
        if st.button(":material/add:", key="state_plus"):
            st.session_state.counter += 1

    with subcol_right_2:
        # 2. Modification: Update the dictionary value (Decrement)
        if st.button(":material/remove:", key="state_minus"):
            st.session_state.counter -= 1
    
    # 3. Read: Display the value
    st.metric("State Count", st.session_state.counter)
    st.caption("This works because we only set the counter to 0 if it doesn't exist.")

# Footer
st.divider()
st.caption("Day 9: Understanding Session State | 30 Days of AI")
```

---

## 📘 Explanation

### 🔍 How It Works: Step-by-Step

Streamlit runs your **entire Python script from top to bottom** every time you interact with a widget (like clicking a button). This creates a logic trap if you aren’t careful.

---

## 1️⃣ The Trap: Standard Variables

```python
with col1:
    st.header(":material_cancel: Standard Variable")

    # This line runs every time you click ANY button.
    count_wrong = 0

    if st.button(":material_add:", key="std_plus"):
        count_wrong += 1  # Becomes 0 + 1 = 1

    st.metric("Standard Count", count_wrong)
```

* ❌ `count_wrong = 0` is the **culprit**
* Every click causes Streamlit to **rerun the script**
* That line executes again, immediately resetting the value

📌 Result: The counter never goes beyond `1` or `-1`

* 🔑 `key="std_plus"`:

  * Each widget must have a **unique key**
  * Prevents Streamlit from confusing buttons across columns

---

## 2️⃣ The Fix: Initialization Pattern

```python
with col2:
    st.header(":material_check_circle: Session State")

    # 1. Initialization: Create the key only if it doesn't exist yet
    if "counter" not in st.session_state:
        st.session_state.counter = 0
```

* 🧠 `st.session_state`:

  * A **persistent dictionary** that survives reruns

* ✅ `if "counter" not in st.session_state:`:

  * Ensures the value is set **only once**
  * On future reruns, this block is skipped

📌 This is the **standard Session State pattern** you should always use.

---

## 3️⃣ Modifying and Reading State

```python
# 2. Modification: Update the dictionary value
if st.button(":material_add:", key="state_plus"):
    st.session_state.counter += 1

# 3. Read: Display the value
st.metric("State Count", st.session_state.counter)
```

* 🔁 `st.session_state.counter += 1`:

  * Updates the **stored value**, not a temporary variable

* 📊 `st.metric(...)`:

  * Reads directly from Session State
  * Value keeps increasing instead of resetting

---

## 🚀 Why This Matters

This concept is **critical** because:

* 🧠 Chatbots need to remember conversation history
* 🛒 Apps need to track user actions
* 📊 Dashboards need to preserve filters and selections

This prepares us perfectly for the next step:

> 💬 Applying **Session State** to our chatbot so it remembers past messages

---

## 📚 Resources

* 📘 **Session State Documentation**
* 📘 **Session State Guide**


