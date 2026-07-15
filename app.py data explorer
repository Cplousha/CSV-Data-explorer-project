"""
CSV Data Explorer
-----------------
A beginner-friendly Streamlit app that lets you:
1. Upload a CSV file
2. Preview and inspect it (rows, columns, missing values, duplicates)
3. Clean it (remove duplicates, handle missing values)
4. Save the cleaned data into a local SQLite database
5. Run simple SQL SELECT queries against that database
6. View a basic chart of the data
7. Download the cleaned CSV

Every section below has comments explaining what it does and why.
"""

# ---------------------------------------------------------
# 1. IMPORTS
# ---------------------------------------------------------
import streamlit as st          # the web app framework
import pandas as pd             # for working with tabular data (CSV -> DataFrame)
import sqlite3                  # Python's built-in SQLite database library
import io                       # helps us turn a DataFrame back into a downloadable CSV

# ---------------------------------------------------------
# 2. PAGE SETUP
# ---------------------------------------------------------
st.set_page_config(page_title="CSV Data Explorer", layout="wide")
st.title("📊 CSV Data Explorer")
st.write(
    "Upload a CSV file, explore it, clean it, save it to a database, "
    "run SQL queries, and download the result."
)

# ---------------------------------------------------------
# 3. SESSION STATE
# ---------------------------------------------------------
# Streamlit re-runs the whole script every time you interact with a widget
# (like clicking a button). "session_state" is how we remember data
# (like our cleaned DataFrame) between those re-runs.
if "df" not in st.session_state:
    st.session_state.df = None          # will hold the current working DataFrame
if "cleaned" not in st.session_state:
    st.session_state.cleaned = False    # tracks whether cleaning has been applied

# Name of the local SQLite database file. It will be created in the
# same folder as this app the first time we save data to it.
DB_FILE = "data_explorer.db"
TABLE_NAME = "cleaned_data"

# ---------------------------------------------------------
# 4. FILE UPLOAD
# ---------------------------------------------------------
st.header("1. Upload your CSV file")

uploaded_file = st.file_uploader("Choose a CSV file", type=["csv"])

if uploaded_file is not None:
    try:
        # Try to read the uploaded file as a CSV.
        df = pd.read_csv(uploaded_file)

        if df.empty:
            st.error("This CSV file is empty. Please upload a file that contains data.")
        else:
            # Store the freshly uploaded data in session_state so it survives re-runs.
            st.session_state.df = df
            st.session_state.cleaned = False
            st.success(f"File uploaded successfully! ({df.shape[0]} rows, {df.shape[1]} columns)")

    except pd.errors.EmptyDataError:
        st.error("The file appears to be empty or not a valid CSV.")
    except pd.errors.ParserError:
        st.error("Could not parse this file. Please make sure it's a properly formatted CSV.")
    except Exception as e:
        # Catch-all so the app never crashes on a bad upload.
        st.error(f"Something went wrong while reading the file: {e}")

# ---------------------------------------------------------
# 5. ONLY SHOW THE REST OF THE APP IF WE HAVE DATA
# ---------------------------------------------------------
if st.session_state.df is not None:
    df = st.session_state.df

    # -------------------------------------------------
    # 5a. PREVIEW
    # -------------------------------------------------
    st.header("2. Preview")
    st.write("First 10 rows of your data:")
    st.dataframe(df.head(10))

    # -------------------------------------------------
    # 5b. BASIC STATS
    # -------------------------------------------------
    st.header("3. Basic Stats")

    num_rows = df.shape[0]
    num_cols = df.shape[1]
    num_missing = int(df.isnull().sum().sum())         # total missing cells in the whole table
    num_duplicates = int(df.duplicated().sum())         # rows that are exact copies of another row

    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Rows", num_rows)
    col2.metric("Columns", num_cols)
    col3.metric("Missing values", num_missing)
    col4.metric("Duplicate rows", num_duplicates)

    # -------------------------------------------------
    # 5c. CLEANING OPTIONS
    # -------------------------------------------------
    st.header("4. Clean the data")

    remove_duplicates = st.checkbox("Remove duplicate rows")

    missing_strategy = st.selectbox(
        "How should we handle missing values?",
        [
            "Do nothing",
            "Drop rows with any missing values",
            "Fill numeric columns with the column average",
            "Fill missing values with a custom value",
        ],
    )

    custom_fill_value = ""
    if missing_strategy == "Fill missing values with a custom value":
        custom_fill_value = st.text_input("Value to use for filling missing cells", value="Unknown")

    if st.button("Apply cleaning"):
        cleaned_df = df.copy()

        try:
            # Remove duplicates if the user asked for it.
            if remove_duplicates:
                cleaned_df = cleaned_df.drop_duplicates()

            # Handle missing values based on the chosen strategy.
            if missing_strategy == "Drop rows with any missing values":
                cleaned_df = cleaned_df.dropna()

            elif missing_strategy == "Fill numeric columns with the column average":
                numeric_cols = cleaned_df.select_dtypes(include="number").columns
                for c in numeric_cols:
                    cleaned_df[c] = cleaned_df[c].fillna(cleaned_df[c].mean())

            elif missing_strategy == "Fill missing values with a custom value":
                cleaned_df = cleaned_df.fillna(custom_fill_value)

            st.session_state.df = cleaned_df
            st.session_state.cleaned = True
            st.success("Cleaning applied! Scroll up to see the updated stats and preview.")
            st.rerun()  # re-run the app so the preview/stats above reflect the cleaned data

        except Exception as e:
            st.error(f"Something went wrong while cleaning the data: {e}")

    # -------------------------------------------------
    # 5d. SAVE TO SQLITE
    # -------------------------------------------------
    st.header("5. Save to a local database")
    st.write(
        f"This will save your current data into a local SQLite file called "
        f"`{DB_FILE}`, in a table called `{TABLE_NAME}`."
    )

    if st.button("Save to SQLite database"):
        try:
            # sqlite3.connect creates the file if it doesn't exist yet.
            conn = sqlite3.connect(DB_FILE)
            df.to_sql(TABLE_NAME, conn, if_exists="replace", index=False)
            conn.close()
            st.success(f"Saved {df.shape[0]} rows to '{DB_FILE}' (table: '{TABLE_NAME}').")
        except Exception as e:
            st.error(f"Could not save to the database: {e}")

    # -------------------------------------------------
    # 5e. RUN SQL QUERIES
    # -------------------------------------------------
    st.header("6. Run a SQL query")
    st.write(
        f"Query the `{TABLE_NAME}` table you just saved. "
        f"Only SELECT statements are allowed (this keeps things safe and simple). "
        f"Example: `SELECT * FROM {TABLE_NAME} LIMIT 5`"
    )

    sql_query = st.text_area("Enter your SQL SELECT query", value=f"SELECT * FROM {TABLE_NAME} LIMIT 5")

    if st.button("Run query"):
        # Basic safety check: only allow queries that start with SELECT.
        # This is a simple beginner-friendly guard, not a full security solution.
        cleaned_query = sql_query.strip().lower()

        if not cleaned_query.startswith("select"):
            st.error("Only SELECT queries are allowed. Your query must start with SELECT.")
        else:
            try:
                conn = sqlite3.connect(DB_FILE)
                result = pd.read_sql_query(sql_query, conn)
                conn.close()
                st.write("Query results:")
                st.dataframe(result)
            except Exception as e:
                st.error(
                    "Your SQL query could not be run. Please check the syntax "
                    f"and make sure you've saved data to the database first. Details: {e}"
                )

    # -------------------------------------------------
    # 5f. CHART
    # -------------------------------------------------
    st.header("7. Chart")

    numeric_columns = df.select_dtypes(include="number").columns.tolist()

    if len(numeric_columns) == 0:
        st.info("No numeric columns found, so there's nothing to chart yet.")
    else:
        chart_column = st.selectbox("Choose a numeric column to chart", numeric_columns)
        chart_type = st.radio("Chart type", ["Bar chart", "Line chart"], horizontal=True)

        try:
            if chart_type == "Bar chart":
                st.bar_chart(df[chart_column])
            else:
                st.line_chart(df[chart_column])
        except Exception as e:
            st.error(f"Could not draw the chart: {e}")

    # -------------------------------------------------
    # 5g. DOWNLOAD CLEANED CSV
    # -------------------------------------------------
    st.header("8. Download your data")

    # Convert the DataFrame back into CSV text in memory (no temp file needed).
    csv_buffer = io.StringIO()
    df.to_csv(csv_buffer, index=False)

    st.download_button(
        label="Download current data as CSV",
        data=csv_buffer.getvalue(),
        file_name="cleaned_data.csv",
        mime="text/csv",
    )

else:
    st.info("👆 Upload a CSV file above to get started.")
