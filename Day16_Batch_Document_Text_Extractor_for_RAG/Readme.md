# 📘 Day 16 — Batch Document Text Extractor for RAG

## 🧠 Goal of Day 16

Day 16 marks the **true beginning of RAG (Retrieval‑Augmented Generation)**.

🎯 **Objective**:

* Upload **multiple documents in batch** (TXT, MD, PDF)
* Extract **raw text** reliably
* Store it in **Snowflake tables**
* Prepare clean data for **chunking, embeddings, and retrieval**

> ⚠️ **Brutal mentor truth**: RAG does NOT start with embeddings. It starts with **clean, structured text ingestion**.

---

## 🧩 What This App Does (High Level)

* 📤 Batch file upload (TXT / Markdown / PDF)
* 📄 Extracts raw text from each file
* 🧮 Calculates metadata (word count, character count)
* 🗄️ Saves everything into Snowflake
* 🔍 Allows querying and reviewing stored documents

This is **production‑grade ingestion**, not a toy uploader.

---

## 🧱 Core Libraries Used

```python
import streamlit as st
from pypdf import PdfReader
import io
import pandas as pd
from datetime import datetime
```

* `streamlit` → UI framework
* `pypdf` → PDF text extraction
* `io` → In‑memory byte handling
* `pandas` → Preview tables
* `datetime` → Metadata timestamps

---

## 🔌 Snowflake Connection (Environment‑Aware)

```python
try:
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()
```

### Why this matters

* ✅ Works inside **Streamlit in Snowflake**
* ✅ Works **locally** and on **Streamlit Community Cloud**
* ❌ No environment‑specific hacks

> 💡 **Interview line**: “We use environment‑aware Snowpark session detection for portability.”

---

## 🗂️ Session State: Database Configuration

```python
if 'database' not in st.session_state:
    st.session_state.database = "RAG_DB"
if 'schema' not in st.session_state:
    st.session_state.schema = "RAG_SCHEMA"
if 'table_name' not in st.session_state:
    st.session_state.table_name = "EXTRACTED_DOCUMENTS"
```

### Why this matters

* 🧠 Persistent configuration across reruns
* 🔁 Editable without breaking state
* 🏗️ Dynamic table targeting

---

## 🧰 Database Setup UI

* Editable **Database / Schema / Table** fields
* Auto‑creation when saving
* Clear target location preview

```text
DATABASE.SCHEMA.TABLE
```

> ⚠️ **Brutal truth**: Hard‑coding database names is amateur hour.

---

## 📥 Sample Dataset (Optional Accelerator)

* 100 TXT files
* Customer reviews
* Perfect for batch testing

```text
review‑001.txt → review‑100.txt
```

> 💡 Lets you test **realistic RAG ingestion immediately**.

---

## 📤 File Upload (Batch‑Enabled)

```python
uploaded_files = st.file_uploader(
    "Choose file(s)",
    type=["txt", "md", "pdf"],
    accept_multiple_files=True
)
```

### Why this matters

* 📦 True batch ingestion
* 📄 Mixed document formats
* ⚙️ Scales to hundreds of files

---

## 🔁 Replace vs Append Mode (Critical)

* Detects if table already exists
* Default behavior adapts automatically

### Replace Mode

* 🚨 Deletes existing rows
* ✅ Clean re‑ingestion

### Append Mode

* ➕ Adds new documents
* ⚠️ Risk of duplication

> 🔥 **Brutal rule**: Always know whether your ingestion is **destructive or additive**.

---

## ⚙️ Text Extraction Logic (Per File)

### TXT / Markdown

```python
extracted_text = uploaded_file.read().decode("utf‑8")
```

### PDF

```python
pdf_reader = PdfReader(io.BytesIO(uploaded_file.read()))
for page in pdf_reader.pages:
    extracted_text += page.extract_text() + "\n\n"
```

### Metadata Calculated

* 📏 Word count
* 📐 Character count
* 📎 File name / size / type

---

## 📊 Extraction Progress & Feedback

* 📈 Progress bar
* ⚠️ Per‑file error handling
* ✅ Success / failure metrics

> 💡 Users always know **what succeeded and what failed**.

---

## 🗄️ Snowflake Table Design

```sql
CREATE TABLE IF NOT EXISTS ... (
  DOC_ID NUMBER AUTOINCREMENT,
  FILE_NAME VARCHAR,
  FILE_TYPE VARCHAR,
  FILE_SIZE NUMBER,
  EXTRACTED_TEXT VARCHAR,
  UPLOAD_TIMESTAMP TIMESTAMP_NTZ,
  WORD_COUNT NUMBER,
  CHAR_COUNT NUMBER
)
```

### Why this schema works

* 🔑 Surrogate key for tracking
* 🧠 Raw text preserved
* 📊 Metadata supports analytics
* 🧱 Ready for chunking & embeddings

---

## 💾 Saving Data Safely

* Escapes quotes
* Inserts documents one‑by‑one
* Stores table references in session state

```python
st.session_state.rag_source_table
```

> 💡 Enables **downstream RAG apps** without reconfiguration.

---

## 🔍 View & Query Saved Documents

Features:

* 📊 Record count
* 📋 Document metadata table
* 📖 Full‑text viewer per document

> ⚠️ Only shows data for the **currently selected table** — avoids stale results.

---

## 🎯 Final Result (Why Day 16 Matters)

After Day 16, you now have:

* ✅ Clean document ingestion
* ✅ Batch processing
* ✅ Persistent Snowflake storage
* ✅ Review & inspection UI
* ✅ RAG‑ready text source

🔥 **Brutal takeaway**:

> Embeddings are useless without disciplined ingestion. Day 16 is the real foundation of RAG.

```python
import streamlit as st
from pypdf import PdfReader
import io
import pandas as pd
from datetime import datetime

# Establish Snowflake connection
# Connect to Snowflake
try:
    # Works in Streamlit in Snowflake
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    # Works locally and on Streamlit Community Cloud
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()

st.title(":material/description: Batch Document Text Extractor")
st.write("Upload multiple documents at once to extract text and save to Snowflake for RAG applications.")

# Initialize session state for database configuration
if 'database' not in st.session_state:
    st.session_state.database = "RAG_DB"
if 'schema' not in st.session_state:
    st.session_state.schema = "RAG_SCHEMA"
if 'table_name' not in st.session_state:
    st.session_state.table_name = "EXTRACTED_DOCUMENTS"

# Main configuration container
with st.container(border=True):
    st.subheader(":material/analytics: Database Setup")

    # Database configuration
    col1, col2, col3 = st.columns(3)
    with col1:
        st.session_state.database = st.text_input("Database", value=st.session_state.database, key="db_input")
    with col2:
        st.session_state.schema = st.text_input("Schema", value=st.session_state.schema, key="schema_input")
    with col3:
        st.session_state.table_name = st.text_input("Table Name", value=st.session_state.table_name, key="table_input")
    
    st.info(f":material/location_on: Target location: `{st.session_state.database}.{st.session_state.schema}.{st.session_state.table_name}`")
    st.caption(":material/lightbulb: Database will be created automatically when you save documents")
    
    st.divider()
    
    # Download Review Data section
    st.subheader(":material/download: Download Review Data")
    st.write("To get started quickly, download our sample dataset of 100 customer reviews from Avalanche winter sports equipment.")
    
    col1, col2 = st.columns([2, 1])
    with col1:
        st.info(":material/info: **Sample Dataset**: 100 customer review files (TXT format) with product feedback, sentiment scores, and order information.")
    with col2:
        st.link_button(
            ":material/download: Download review.zip",
            "https://github.com/streamlit/30DaysOfAI/raw/refs/heads/main/assets/review.zip",
            use_container_width=True
        )
    
    with st.expander(":material/help: How to use the sample data"):
        st.markdown("""
        **Steps:**
        1. Click the **Download review.zip** button above
        2. Unzip the downloaded file on your computer
        3. Use the **Upload Documents** section below to select all 100 review files
        4. Click **Extract Text** to process and save to Snowflake
        
        **What's included:**
        - 100 customer review files (`review-001.txt` to `review-100.txt`)
        - Each review contains: product name, date, review summary, sentiment score, and order ID
        - Perfect for testing batch processing and building RAG applications
        
        **Tip:** You can upload all 100 files at once for optimal batch processing!
        """)
    
    st.divider()
    
    # File uploader
    st.subheader(":material/upload: Upload Documents")
    uploaded_files = st.file_uploader(
        "Choose file(s)",
        type=["txt", "md", "pdf"],
        accept_multiple_files=True,
        help="Supported formats: TXT, MD, PDF. Upload multiple files at once!"
)

    # Check if table exists to set default replace_mode value
    table_exists = False
    try:
        check_result = session.sql(f"""
            SELECT COUNT(*) as CNT FROM {st.session_state.database}.{st.session_state.schema}.{st.session_state.table_name}
        """).collect()
        table_exists = True  # Table exists if query succeeds
    except:
        table_exists = False  # Table doesn't exist
    
    # Set checkbox value based on table existence
    replace_mode = st.checkbox(
        f":material/sync: Replace Table Mode for `{st.session_state.table_name}`",
        value=table_exists,  # True if table exists, False if it doesn't
        help=f"When enabled, clears all existing data in {st.session_state.database}.{st.session_state.schema}.{st.session_state.table_name} before saving new documents"
    )
    
    if replace_mode:
        st.warning(f":material/warning: **Replace Mode Enabled** - All existing documents in `{st.session_state.table_name}` will be deleted before saving new ones.")
    else:
        st.info(f":material/add: **Append Mode** - New documents will be added to `{st.session_state.table_name}`.")

# Get values from session state for use in the rest of the code
database = st.session_state.database
schema = st.session_state.schema
table_name = st.session_state.table_name

# Display upload info
if uploaded_files:
    with st.container(border=True):
        st.subheader(":material/upload: Uploaded Documents")
        st.success(f":material/folder: {len(uploaded_files)} file(s) uploaded")
        
        # Preview selected files
        with st.expander(":material/assignment: View Selected Files", expanded=False):
            file_list_df = pd.DataFrame([
                {
                    "File Name": f.name,
                    "Size": f"{f.size:,} bytes",
                    "Type": "TXT" if f.name.lower().endswith('.txt') 
                           else "Markdown" if f.name.lower().endswith('.md')
                           else "PDF" if f.name.lower().endswith('.pdf')
                           else "Unknown"
                }
                for f in uploaded_files
            ])
            st.dataframe(file_list_df, use_container_width=True)
        
        # Process files button
        process_button = st.button(
            f":material/sync: Extract Text from {len(uploaded_files)} File(s)",
            type="primary",
            use_container_width=True
        )
    
    if process_button:
        # Initialize progress tracking
        success_count = 0
        error_count = 0
        extracted_data = []
        
        progress_bar = st.progress(0, text="Starting extraction...")
        status_container = st.empty()
        
        for idx, uploaded_file in enumerate(uploaded_files):
            progress_pct = (idx + 1) / len(uploaded_files)
            progress_bar.progress(progress_pct, text=f"Processing {idx+1}/{len(uploaded_files)}: {uploaded_file.name}")
            
            try:
                # Determine file type from extension
                if uploaded_file.name.lower().endswith('.txt'):
                    file_type = "TXT"
                elif uploaded_file.name.lower().endswith('.md'):
                    file_type = "Markdown"
                elif uploaded_file.name.lower().endswith('.pdf'):
                    file_type = "PDF"
                else:
                    file_type = "Unknown"
                
                # Reset file pointer
                uploaded_file.seek(0)
                
                # Extract text based on file type
                extracted_text = ""
                
                if uploaded_file.name.lower().endswith(('.txt', '.md')):
                    # Handle TXT and Markdown files
                    extracted_text = uploaded_file.read().decode("utf-8")
                
                elif uploaded_file.name.lower().endswith('.pdf'):
                    # Handle PDF files
                    pdf_reader = PdfReader(io.BytesIO(uploaded_file.read()))
                    
                    # Extract text from all pages
                    for page in pdf_reader.pages:
                        page_text = page.extract_text()
                        if page_text:
                            extracted_text += page_text + "\n\n"
                
                # Check if extraction was successful
                if extracted_text and extracted_text.strip():
                    # Calculate metadata
                    word_count = len(extracted_text.split())
                    char_count = len(extracted_text)
                    
                    # Store extracted data
                    extracted_data.append({
                        'file_name': uploaded_file.name,
                        'file_type': file_type,
                        'file_size': uploaded_file.size,
                        'extracted_text': extracted_text,
                        'word_count': word_count,
                        'char_count': char_count
                    })
                    
                    success_count += 1
                else:
                    error_count += 1
                    status_container.warning(f":material/warning: No text extracted from: {uploaded_file.name}")
                    
            except Exception as e:
                error_count += 1
                status_container.error(f":material/cancel: Error processing {uploaded_file.name}: {str(e)}")
        
        progress_bar.empty()
        status_container.empty()
        
        # Display results
        with st.container(border=True):
            st.subheader(":material/analytics: Documents Written to a Database Table")
            
            col1, col2, col3 = st.columns(3)
            with col1:
                st.metric(":material/check_circle: Successful", success_count)
            with col2:
                st.metric(":material/cancel: Failed", error_count)
            with col3:
                st.metric(":material/analytics: Total Words", f"{sum(d['word_count'] for d in extracted_data):,}")
            
            # Store in session state for review
            if extracted_data:
                st.session_state.extracted_data = extracted_data
                st.success(f":material/check_circle: Successfully extracted text from {success_count} file(s)!")
                
                # Preview extracted data
                with st.expander(":material/visibility: Preview First 3 Files"):
                    for data in extracted_data[:3]:
                        with st.container(border=True):
                            st.markdown(f"**{data['file_name']}**")
                            st.caption(f"{data['word_count']:,} words")
                            preview_text = data['extracted_text'][:200]
                            if len(data['extracted_text']) > 200:
                                preview_text += "..."
                            st.text(preview_text)
                    
                    if len(extracted_data) > 3:
                        st.caption(f"... and {len(extracted_data) - 3} more")
                
                # Save to Snowflake
                with st.status("Saving to Snowflake...", expanded=True) as status:
                    try:
                        # Ensure database and schema exist
                        st.write(":material/looks_one: Setting up database structure...")
                        session.sql(f"CREATE DATABASE IF NOT EXISTS {database}").collect()
                        session.sql(f"CREATE SCHEMA IF NOT EXISTS {database}.{schema}").collect()
                        
                        # Create table if it doesn't exist
                        st.write(":material/looks_two: Creating table if needed...")
                        create_table_sql = f"""
                        CREATE TABLE IF NOT EXISTS {database}.{schema}.{table_name} (
                            DOC_ID NUMBER AUTOINCREMENT,
                            FILE_NAME VARCHAR,
                            FILE_TYPE VARCHAR,
                            FILE_SIZE NUMBER,
                            EXTRACTED_TEXT VARCHAR,
                            UPLOAD_TIMESTAMP TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
                            WORD_COUNT NUMBER,
                            CHAR_COUNT NUMBER
                        )
                        """
                        session.sql(create_table_sql).collect()
                        
                        # Replace mode: clear existing data
                        if replace_mode:
                            st.write(":material/sync: Replace mode: Clearing existing data...")
                            try:
                                session.sql(f"TRUNCATE TABLE {database}.{schema}.{table_name}").collect()
                                st.write("   :material/check_circle: Existing data cleared")
                            except Exception as e:
                                st.write(f"   :material/warning: No existing data to clear")
                        
                        # Insert all extracted data
                        st.write(f":material/looks_3: Inserting {len(extracted_data)} document(s)...")
                        
                        for idx, data in enumerate(extracted_data, 1):
                            st.caption(f"Saving {idx}/{len(extracted_data)}: {data['file_name']}")
                            # Escape single quotes in text
                            safe_text = data['extracted_text'].replace("'", "''")
                            insert_sql = f"""
                            INSERT INTO {database}.{schema}.{table_name}
                            (FILE_NAME, FILE_TYPE, FILE_SIZE, EXTRACTED_TEXT, WORD_COUNT, CHAR_COUNT)
                            VALUES ('{data['file_name']}', '{data['file_type']}', {data['file_size']}, 
                                    '{safe_text}', {data['word_count']}, {data['char_count']})
                            """
                            session.sql(insert_sql).collect()
                        
                        status.update(label=":material/check_circle: All documents saved!", state="complete", expanded=False)
                        
                        mode_msg = "replaced in" if replace_mode else "saved to"
                        st.success(f":material/check_circle: Successfully {mode_msg} `{database}.{schema}.{table_name}`\n\n:material/description: {len(extracted_data)} document(s) now in table")
                        
                        # Store references in session state for downstream apps
                        st.session_state.rag_source_table = f"{database}.{schema}.{table_name}"
                        st.session_state.rag_source_database = database
                        st.session_state.rag_source_schema = schema
                        
                        st.balloons()
                        
                    except Exception as e:
                        st.error(f"Error saving to Snowflake: {str(e)}")
            else:
                st.warning("No text was successfully extracted from any file.")

st.divider()

# View all saved documents section
with st.container(border=True):
    st.subheader(":material/search: View Saved Documents")
    
    # Check if table exists and show record count
    try:
        count_result = session.sql(f"""
            SELECT COUNT(*) as CNT FROM {database}.{schema}.{table_name}
        """).collect()
        
        if count_result:
            record_count = count_result[0]['CNT']
            if record_count > 0:
                st.warning(f":material/warning: **{record_count} record(s)** currently in table `{database}.{schema}.{table_name}`")
            else:
                st.info(":material/inbox: **Table is empty** - No documents uploaded yet.")
    except:
        st.info(":material/inbox: **Table doesn't exist yet** - Upload and save documents to create it.")
    
    query_button = st.button("Query Table", type="secondary", use_container_width=True)
    
    if query_button:
        try:
            full_table_name = f"{database}.{schema}.{table_name}"
            
            # Query the table
            query_sql = f"""
            SELECT DOC_ID, FILE_NAME, FILE_TYPE, FILE_SIZE, UPLOAD_TIMESTAMP, WORD_COUNT, CHAR_COUNT
            FROM {full_table_name}
            ORDER BY UPLOAD_TIMESTAMP DESC
            """
            df = session.sql(query_sql).to_pandas()
        
            # Store in session state for persistence
            st.session_state.queried_docs = df
            st.session_state.full_table_name = full_table_name
            st.rerun()
                
        except Exception as e:
            st.error(f"Error: {str(e)}")
            st.info(":material/lightbulb: Table may not exist yet. Upload and save documents first!")
    
    # Display query results if available
    if 'queried_docs' in st.session_state and 'full_table_name' in st.session_state:
        # Use current session state values for dynamic table name display
        current_full_table_name = f"{st.session_state.database}.{st.session_state.schema}.{st.session_state.table_name}"
        
        # Only show results if they match the current table (avoid showing stale data from a different table)
        if st.session_state.full_table_name == current_full_table_name:
            df = st.session_state.queried_docs
            
            if len(df) > 0:
                st.code(f"{current_full_table_name}", language="sql")
                
                # Summary metrics
                col1, col2, col3 = st.columns(3)
                with col1:
                    st.metric("Documents", len(df))
                with col2:
                    st.metric("Words", f"{df['WORD_COUNT'].sum():,}")
                with col3:
                    st.metric("Characters", f"{df['CHAR_COUNT'].sum():,}")
                
                st.divider()
                
                # Display documents table
                st.dataframe(
                    df[['DOC_ID', 'FILE_NAME', 'FILE_TYPE', 'WORD_COUNT', 'UPLOAD_TIMESTAMP']],
                    use_container_width=True
                )
                
                # Option to view full text of a document
                with st.expander(":material/menu_book: View Full Document Text"):
                    doc_id = st.selectbox(
                        "Select Document ID:",
                        options=df['DOC_ID'].tolist(),
                        format_func=lambda x: f"Doc #{x} - {df[df['DOC_ID']==x]['FILE_NAME'].values[0]}"
                    )
                    
                    if st.button("Load Text"):
                        text_sql = f"SELECT EXTRACTED_TEXT, FILE_NAME FROM {current_full_table_name} WHERE DOC_ID = {doc_id}"
                        text_result = session.sql(text_sql).to_pandas()
                        if len(text_result) > 0:
                            doc = text_result.iloc[0]
                            # Store in session state
                            st.session_state.loaded_doc_text = doc['EXTRACTED_TEXT']
                            st.session_state.loaded_doc_name = doc['FILE_NAME']
                    
                    # Display loaded text if available
                    if 'loaded_doc_text' in st.session_state:
                        st.text_area(
                            st.session_state.loaded_doc_name,
                            value=st.session_state.loaded_doc_text,
                            height=400
                        )
            else:
                st.info(":material/inbox: Table is empty. Upload files above!")
        else:
            st.info(f":material/sync: Showing results for a different table. Click 'Query Table' to refresh.")
    else:
        st.info(":material/inbox: No documents queried yet. Click 'Query Table' to view saved documents.")

st.divider()
st.caption("Day 16: Batch Document Text Extractor for RAG | 30 Days of AI")
```

## 📥 Download Sample Review Data

To get started quickly, download the **sample dataset of 100 customer reviews** from *Avalanche winter sports equipment*.

* 📦 **File**: `review.zip`
* 📄 **Contents**: 100 TXT review files

### 🧭 How to Use

1. Click the **Download** link to get `review.zip`
2. Unzip the file on your computer
3. You will see files named:

   * `review-001.txt` → `review-100.txt`
4. Open the app and use **Upload Documents**
5. Select **all 100 files at once**
6. Click **Extract Text** to process and save them to Snowflake

> 💡 **Tip**: Upload all 100 files together to see true **batch processing** in action.

---

## 📦 What’s Included in the Sample Data

* ✅ 100 customer review files (TXT format)
* ✅ Each review contains:

  * Product name
  * Review date
  * Review summary
  * Sentiment score
  * Order ID

🎯 **Perfect for**:

* Batch ingestion testing
* RAG document pipelines
* Chunking & embedding demos (Day 17+)

---

## 🧠 How It Works: Step-by-Step

Let’s break down **exactly** what each part of the code does.


## 1️⃣ Database Configuration and Session State

```python
import streamlit as st
from pypdf import PdfReader
import io

# Connect to Snowflake
try:
    # Works in Streamlit in Snowflake
    from snowflake.snowpark.context import get_active_session
    session = get_active_session()
except:
    # Works locally and on Streamlit Community Cloud
    from snowflake.snowpark import Session
    session = Session.builder.configs(st.secrets["connections"]["snowflake"]).create()

# Initialize session state for persistence
if 'database' not in st.session_state:
    st.session_state.database = "RAG_DB"
    st.session_state.schema = "RAG_SCHEMA"
    st.session_state.table_name = "EXTRACTED_DOCUMENTS"
```

### 🔍 Explanation

* `streamlit` → UI framework
* `PdfReader` → PDF text extraction
* `try / except` → Environment-aware Snowflake connection
* `session` → Active Snowflake session used everywhere
* **Session state init** → Database, schema, and table persist across reruns

> ⚠️ **Brutal truth**: If you don’t persist config in session state, your app will fight itself.

---

## 2️⃣ Batch File Upload

```python
uploaded_files = st.file_uploader(
    "Choose file(s)",
    type=["txt", "md", "pdf"],
    accept_multiple_files=True,
    help="Supported formats: TXT, MD, PDF. Upload multiple files at once!"
)

if uploaded_files:
    st.success(f":material_check_circle: {len(uploaded_files)} file(s) uploaded")
```

### 🔍 Why this matters

* `accept_multiple_files=True` → **True batch uploads**
* File type restriction → Prevents invalid formats
* `uploaded_files` → List of file objects
* Status message → Confirms batch size

---

## 3️⃣ Process Button and Progress Tracking

```python
process_button = st.button(
    f":material_sync: Extract Text from {len(uploaded_files)} File(s)",
    type="primary",
    use_container_width=True
)

if process_button:
    success_count = 0
    error_count = 0
    extracted_data = []

    progress_bar = st.progress(0, text="Starting extraction...")
    status_container = st.empty()
```

### 🔍 Why this matters

* Dynamic button label → Shows workload size
* `type="primary"` → Clear call-to-action
* Scoped variables → Only exist after click
* Progress UI → Real-time feedback

---

## 4️⃣ Replace Table Mode (Critical)

```python
try:
    result = session.sql(f"SELECT COUNT(*) FROM {full_table_name}").collect()
    table_exists = True
except:
    table_exists = False

replace_mode = st.checkbox(
    f":material_sync: Replace Table Mode for `{st.session_state.table_name}`",
    value=table_exists,
    help=f"When enabled, replaces all existing data in {full_table_name}"
)
```

### 🔍 Why this matters

* Smart default behavior
* Replace = destructive
* Append = additive

> 🔥 **Brutal rule**: Never ingest blindly. Always know if you’re deleting data.

---

## 5️⃣ Extract Text from Files

```python
for idx, uploaded_file in enumerate(uploaded_files):
    progress_pct = (idx + 1) / len(uploaded_files)
    progress_bar.progress(progress_pct, text=f"Processing {idx+1}/{len(uploaded_files)}: {uploaded_file.name}")

    if uploaded_file.name.lower().endswith(('.txt', '.md')):
        extracted_text = uploaded_file.read().decode("utf-8")
    elif uploaded_file.name.lower().endswith('.pdf'):
        pdf_reader = PdfReader(io.BytesIO(uploaded_file.read()))
        extracted_text = ""
        for page in pdf_reader.pages:
            page_text = page.extract_text()
            if page_text:
                extracted_text += page_text + "\n\n"
```

### 🔍 Why this matters

* `enumerate()` → Index + file
* `.endswith()` → Reliable file detection
* UTF-8 decoding → Text safety
* Page-by-page PDF extraction

---

## 6️⃣ Handle Replace Mode

```python
if replace_mode:
    try:
        session.sql(f"TRUNCATE TABLE {full_table_name}").collect()
    except:
        pass
```

### 🔍 Why this matters

* `TRUNCATE` → Fast bulk deletion
* Keeps schema intact
* Safe when table doesn’t exist

---

## 7️⃣ Save to Snowflake

```python
session.sql(f"CREATE DATABASE IF NOT EXISTS {st.session_state.database}").collect()
session.sql(f"CREATE SCHEMA IF NOT EXISTS {st.session_state.database}.{st.session_state.schema}").collect()
```

```sql
CREATE TABLE IF NOT EXISTS ... (
  DOC_ID NUMBER AUTOINCREMENT,
  FILE_NAME VARCHAR,
  FILE_TYPE VARCHAR,
  FILE_SIZE NUMBER,
  EXTRACTED_TEXT VARCHAR,
  UPLOAD_TIMESTAMP TIMESTAMP_NTZ,
  WORD_COUNT NUMBER,
  CHAR_COUNT NUMBER
)
```

### 🔍 Why this schema matters

* `AUTOINCREMENT` → Unique document IDs
* `EXTRACTED_TEXT` → Full document content
* Timestamp + metadata → Auditability

---

## 8️⃣ Query and View Saved Documents

```python
if st.button(":material_analytics: Query Table"):
    df = session.sql(f"SELECT * FROM {full_table_name}").to_pandas()
    st.session_state.queried_docs = df
    st.rerun()
```

### 🔍 Why this matters

* Confirms ingestion worked
* Full-text inspection
* Session state prevents data loss on rerun

---

## 9️⃣ Integration with Day 17 (RAG Chunking)

```python
st.session_state.rag_source_table = f"{database}.{schema}.{table_name}"
st.session_state.rag_source_database = database
st.session_state.rag_source_schema = schema
```

### Why this matters

* 🔁 Seamless handoff to Day 17
* 🧠 No reconfiguration needed
* 📦 All batches merge into one corpus

---

## 🎯 Final Outcome

After this step, you have:

* ✅ Batch document ingestion
* ✅ Clean extracted text
* ✅ Persistent Snowflake storage
* ✅ Metadata-rich table
* ✅ **RAG-ready corpus**

🔥 **Brutal takeaway**:

> If EXTRACTED_TEXT is wrong, everything downstream is wrong. Fix ingestion first.

---

## 📚 Resources

* 📘 `st.file_uploader` Documentation
* 📘 `pypdf` Documentation
* 📘 Snowflake `AUTOINCREMENT`
* 📘 Streamlit Session State Management




