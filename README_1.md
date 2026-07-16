# CSV Data Explorer

A simple Streamlit web app for uploading, exploring, cleaning, and querying CSV data — built as a beginner portfolio project.

## Features
- Upload a CSV file
- Preview the first 10 rows
- See row/column counts, missing values, and duplicate rows at a glance
- Remove duplicate rows and handle missing values
- Save the cleaned data into a local SQLite database
- Run simple SQL `SELECT` queries against that database
- View a basic bar/line chart of a numeric column
- Download the cleaned data as a CSV

## Tech stack
- [Streamlit](https://streamlit.io/) — the web app framework
- [pandas](https://pandas.pydata.org/) — data handling
- [SQLite](https://www.sqlite.org/) (via Python's built-in `sqlite3` module) — local database, no server required

No paid APIs, no AWS, no external accounts needed.

## Getting started

See the setup instructions provided alongside this project for step-by-step install/run instructions.

Quick version:

```bash
pip install -r requirements.txt
streamlit run app.py
```

Then try uploading `sample_data.csv` to see the app in action.

## Project files
- `app.py` — the Streamlit application
- `requirements.txt` — Python dependencies
- `sample_data.csv` — example data for testing
- `.gitignore` — files Git should ignore
