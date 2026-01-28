# 📅 Day 1

## 🔗 Connect to Snowflake

For Day 1, our goal is to establish a connection between our Streamlit app and a Snowflake database. Once that's done, we'll run a simple query to confirm the connection is working and display the Snowflake version in the app.

So to get started, sign up for a free Snowflake trial account using this link and you'll have access for 120 days. The trial includes plenty of credits to get you through the entire 30-day challenge, plus extra for experimenting with other features.

**Note:** The core learning of the 30 Days of AI challenge is tech agnostic and can therefore be adapted to work with other platforms and LLM API providers.

---

## 🧩 See the code:

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
````

---

## 📘 See the explanation

### ⚙️ How It Works: Step-by-Step

Let's break down what each part of the code does.

---

### 🔌 Connection Setup

Before running Day 1, you need to configure your Snowflake connection based on where you're deploying:

#### ⭐ Streamlit in Snowflake (Recommended)

No setup needed! Just create a Streamlit app in Snowsight and the connection works automatically. Skip to Step 1 below.

#### 💻 Local Development or Streamlit Community Cloud

One-time setup required. Create `.streamlit/secrets.toml` in your project folder:

```toml
[connections.snowflake]
account = "xy12345.us-east-1"       # Find in Snowsight → Account → View account details
user = "yourusername"               # Your Snowflake username
password = "yourpassword"           # Your Snowflake password
role = "ACCOUNTADMIN"               # Your role
warehouse = "COMPUTE_WH"            # Your warehouse
database = "SNOWFLAKE_LEARNING_DB"  # Your database
schema = "PUBLIC"                   # Your schema
```

**Important:** Add `.streamlit/secrets.toml` to your `.gitignore` file. Never commit secrets to Git!

For Streamlit Community Cloud, add these same values in your app's secrets settings.

---

### 1️⃣ Import Libraries

```python
import streamlit as st
```

import streamlit as st: Imports the Streamlit library, which is used to build the web app's user interface (UI).

---

### 2️⃣ Connect to Snowflake

```python
# Auto-detect environment and connect
try:
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()
```

from snowflake.snowpark.context import get_active_session: Imports the function that gets the current active Snowflake session. This works automatically in Streamlit in Snowflake (SiS).

session = get_active_session(): Establishes an authenticated Snowflake session. This connection object is stored in the session variable.

except block: If get_active_session() fails (meaning we're running locally or on Community Cloud), we fall back to using secrets from the `.streamlit/secrets.toml` file.

Session.builder.configs(...): Creates a Snowflake session using the connection parameters from st.secrets.

Why the try/except? This pattern makes your code work in all three environments: Streamlit in Snowflake (production), local development, and Streamlit Community Cloud. One codebase works everywhere!

---

### 3️⃣ Query Snowflake Version

```python
# Query and display Snowflake version
version = session.sql("SELECT CURRENT_VERSION()").collect()[0][0]
```

session.sql(...): Uses our session object to execute a raw SQL query ("SELECT CURRENT_VERSION()") on the Snowflake database.

.collect()[0][0]: Fetches the query's result. .collect() brings the data (which is a list of rows) into our script. [0] selects the first row, and the second [0] selects the first column in that row, which contains the version number string.

---

### 4️⃣ Display Results

```python
st.success(f"Successfully connected! Snowflake Version: {version}")
```

st.success(...): Shows a green success message with the Snowflake version number embedded using an f-string. This confirms both that the connection works and displays the version in a single, simple message.

When this code runs, you will see a green success message displaying your Snowflake version. This confirms your connection is working correctly.

---

## ❗ What if it fails?

If the connection fails in Streamlit in Snowflake, try refreshing the app or checking your role permissions. For local development, ensure your `.streamlit/secrets.toml` file exists in the project root with all 7 required parameters (account, user, password, role, warehouse, database, schema).

---

## 📚 Resources

* Streamlit in Snowflake Documentation
* Streamlit Secrets Management
* Snowpark Python API
* Get started with Streamlit

```
```
