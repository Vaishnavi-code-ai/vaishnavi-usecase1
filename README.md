# IDAMP: Intent-Driven Agentic Medallion Pipeline

IDAMP turns raw retail CSV files and a plain-language business question into
cleansed analytics tables and an executive report. A Streamlit interface guides
you through the pipeline, while LangChain ReAct agents use Anthropic Claude to
coordinate the data engineering work. DuckDB runs analytical SQL against the
Gold-layer data.

## What it does

1. Upload one or more raw CSV files and enter a business question in Streamlit.
2. Profile the data and generate source-to-target mapping (STTM) rules.
3. Review and approve the mappings at the Bronze, Silver, and Gold checkpoints.
4. Ingest and cleanse the data through the Medallion layers:

   ```text
   Raw CSV -> Bronze -> Silver -> Gold -> HTML report
   ```

5. Query the Gold data with DuckDB and produce an executive report with charts.

The agents inspect their inputs, plan their work, call Python tools to execute
transformations, and verify the resulting artifacts. Data transformations are
performed by Python; Claude coordinates the agent workflow.

## Technology

- Python
- LangChain and LangGraph ReAct agents
- Anthropic Claude via `langchain-anthropic`
- Streamlit
- Pandas and Parquet
- DuckDB for report queries
- Plotly for report charts

## Prerequisites

- Python 3.10 or later
- An Anthropic API key
- Windows PowerShell (commands below use the requested `C:\iv` virtual
  environment path)

## Setup

Run these commands from the repository root:

```powershell
python -m venv C:\iv
C:\iv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Configure `.env`

Create a file named `.env` in the repository root and add your Anthropic
credentials and model:

```dotenv
ANTHROPIC_API_KEY=your_anthropic_api_key
ANTHROPIC_MODEL=claude-sonnet-4-5
```

Replace the placeholder with your own API key. Do not share or commit this file:
`.env` is listed in `.gitignore`.

## Run the app

With the virtual environment active, start Streamlit from the repository root:

```powershell
streamlit run app/streamlit_app.py
```

Open the local URL printed by Streamlit (usually
`http://localhost:8501`) in your browser.

## Using the pipeline

1. Upload CSV file(s) in the app.
2. Enter a business question that relates to the columns in your data.
3. Run each phase and review the generated STTM before approving it.
4. After the final phase, open the generated report.

Example question:

> Which product category had the highest total sales, and how did sales change
> by month?

## Project layout

```text
app/
  streamlit_app.py       Streamlit entry point
agents/
  orchestrator.py        Phase orchestration and approval workflow
  profiler.py            CSV profiling
  sttm_generator.py      Bronze, Silver, and Gold STTM generation
  bronze_agent.py        Bronze ingestion and audit CSV
  silver_agent.py        Silver cleansing
  gold_agent.py          Gold materialisation
  reporter.py            DuckDB analysis and HTML report generation
core/
  config.py              Environment and data-path configuration
  audit.py               JSONL event audit logger
  llm_retry.py           LLM retry support
  observability.py       Agent traces
data/                    Pipeline inputs and generated layer artifacts
reports/                 Generated reports
audit_logs/              Pipeline audit logs
tests/                   Project tests
requirements.txt         Python dependencies
```

## Changes and additions in this version

- **Bronze audit CSV:** after Bronze ingestion, `write_audit_log` appends one
  record per output Parquet file to `audit_logs/bronze_audit.csv`. Records
  include the run ID, timestamp, row count, total null count, and per-column
  null counts.
- **Claude API compatibility:** Anthropic model factories omit an explicit
  `temperature` parameter, avoiding a request validation error for models that
  reject it.
- **Deterministic STTM generation:** Bronze and Silver mappings are generated
  deterministically from the profiled data and Parquet schemas; the Gold STTM
  remains intent-driven.
- **App entry point:** the Streamlit app is located at
  `app/streamlit_app.py`.
