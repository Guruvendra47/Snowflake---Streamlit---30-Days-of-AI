# Day 1: Connect Streamlit to Snowflake


## 🧩 Streamlit App Code

```python
import streamlit as st

st.title(":material/vpn_key: Day 1: Connect to Snowflake")

# Connect to Snowflake
try:
    # Works in Streamlit in Snowflake
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    # Works locally and on Streamlit Community Cloud
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()

# Query Snowflake version
version = session.sql("SELECT CURRENT_VERSION()").collect()[0][0]

# Display results
st.success(f"Successfully connected! Snowflake Version: {version}")
```

---

## ⚙️ How It Works — Step by Step

### 1️⃣ Import Streamlit

```python
import streamlit as st
```

* Imports the Streamlit library
* Used to build the web UI and display messages

---

### 2️⃣ Connect to Snowflake (Environment-Aware)

```python
try:
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()
```

This logic automatically detects where the app is running:

* **Streamlit in Snowflake (SiS)** → Uses the active Snowflake session (no credentials required)
* **Local machine / Streamlit Community Cloud** → Uses credentials stored securely in `secrets.toml`

✅ **Why this matters:**

* One codebase works everywhere
* No environment-specific rewrites
* Production-ready pattern

---

### 3️⃣ Execute SQL from Streamlit

```python
version = session.sql("SELECT CURRENT_VERSION()").collect()[0][0]
```

* `session.sql(...)` runs SQL directly in Snowflake
* `.collect()` retrieves results into Python
* `[0][0]` extracts the version string

This query validates that the connection is real and working.

---

### 4️⃣ Display Result in UI

```python
st.success(f"Successfully connected! Snowflake Version: {version}")
```

* Displays a green success message
* Confirms both connectivity and query execution

---

## 🔐 Connection Configuration (Local / Community Cloud Only)

If running **outside Streamlit in Snowflake**, create the following file:

```
.streamlit/secrets.toml
```

```toml
[connections.snowflake]
account = "xy12345.us-east-1"       # Snowsight → Account → View account details
user = "yourusername"
password = "yourpassword"
role = "ACCOUNTADMIN"
warehouse = "COMPUTE_WH"
database = "SNOWFLAKE_LEARNING_DB"
schema = "PUBLIC"
```

⚠️ **Security Note:**

* Add `.streamlit/secrets.toml` to `.gitignore`
* Never commit credentials to GitHub

---

## 🧪 Expected Output

When the app runs successfully, you should see:

* A green success message
* The current Snowflake version displayed

This confirms:

* Authentication is successful
* Streamlit can execute SQL on Snowflake

---

## ❗ Troubleshooting

* **Streamlit in Snowflake fails:**

  * Refresh the app
  * Verify role and warehouse permissions

* **Local / Cloud fails:**

  * Ensure `secrets.toml` exists
  * Verify all 7 required parameters are present

---

## 📚 References

* Streamlit in Snowflake Documentation
* Streamlit Secrets Management
* Snowpark Python API
* Get started with Streamlit


