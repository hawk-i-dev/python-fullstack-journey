# Day 11 AI — Build an AI Dataset Explorer

Today’s EOD project: analyze AI request data like a real AI/product team.

You will use Pandas to answer:

```text
Which topics are most requested?
Which model is slowest?
What is the success rate?
Which topic gets the best feedback?
```

## Start your environment

**Windows PowerShell**

```powershell
cd "$env:USERPROFILE\Documents\python-ai-mastery"
.\.venv\Scripts\Activate.ps1
cd projects
New-Item -ItemType Directory -Force day_11_ai_dataset_explorer
cd day_11_ai_dataset_explorer
New-Item -ItemType Directory -Force data, reports, tests
```

**Mac Terminal**

```zsh
cd ~/Documents/python-ai-mastery
source .venv/bin/activate
cd projects
mkdir -p day_11_ai_dataset_explorer/data day_11_ai_dataset_explorer/reports day_11_ai_dataset_explorer/tests
cd day_11_ai_dataset_explorer
```

The Python code is the same on both systems.

---

## Project structure

```text
day_11_ai_dataset_explorer/
├── data/
├── reports/
├── tests/
├── generate_dataset.py
├── analyzer.py
├── main.py
└── README.md
```

## 1. Create sample AI data

Create `generate_dataset.py`:

```python
from pathlib import Path

import pandas as pd


data_file = Path("data") / "ai_requests.csv"

records = [
    [1, "Python Basics", "beginner", "demo-llm", 120, 1.2, 250, 5, "success"],
    [2, "RAG", "intermediate", "demo-llm", 220, 2.1, 420, 5, "success"],
    [3, "AI Agents", "advanced", "demo-llm", 180, 2.8, 390, 4, "success"],
    [4, "Python Basics", "beginner", "demo-llm", 90, 1.0, 200, 4, "success"],
    [5, "Embeddings", "intermediate", "demo-llm", 150, 2.0, 350, 5, "success"],
    [6, "RAG", "beginner", "demo-llm", 200, 3.5, 450, 3, "failed"],
    [7, "AI Agents", "intermediate", "demo-llm", 240, 2.6, 500, 4, "success"],
    [8, "Vector Search", "advanced", "demo-llm", 170, 1.9, 380, 5, "success"],
    [9, "RAG", "intermediate", "demo-llm", 210, 2.3, 430, 4, "success"],
    [10, "Embeddings", "beginner", "demo-llm", 130, 1.4, 300, 5, "success"],
]

columns = [
    "request_id",
    "topic",
    "learner_level",
    "model",
    "prompt_length",
    "response_time_seconds",
    "estimated_tokens",
    "feedback_score",
    "status",
]

data_file.parent.mkdir(parents=True, exist_ok=True)

dataframe = pd.DataFrame(records, columns=columns)
dataframe.to_csv(data_file, index=False)

print(f"Created sample dataset: {data_file}")
print(dataframe.head())
```

Run:

```bash
python generate_dataset.py
```

---

## 2. Build the analyzer

Create `analyzer.py`:

```python
from pathlib import Path

import pandas as pd


REQUIRED_COLUMNS = {
    "request_id",
    "topic",
    "learner_level",
    "model",
    "prompt_length",
    "response_time_seconds",
    "estimated_tokens",
    "feedback_score",
    "status",
}


def load_ai_requests(file_path: Path) -> pd.DataFrame:
    dataframe = pd.read_csv(file_path)

    missing_columns = REQUIRED_COLUMNS - set(dataframe.columns)

    if missing_columns:
        raise ValueError(
            f"Dataset is missing columns: {sorted(missing_columns)}"
        )

    return dataframe


def clean_ai_requests(dataframe: pd.DataFrame) -> pd.DataFrame:
    cleaned = dataframe.copy()

    for column in ["topic", "learner_level", "model", "status"]:
        cleaned[column] = cleaned[column].astype("string").str.strip()

    for column in [
        "prompt_length",
        "response_time_seconds",
        "estimated_tokens",
        "feedback_score",
    ]:
        cleaned[column] = pd.to_numeric(
            cleaned[column],
            errors="coerce",
        )

    cleaned = cleaned.dropna()

    if cleaned.empty:
        raise ValueError("No valid AI request records remain after cleaning.")

    return cleaned


def create_summary(dataframe: pd.DataFrame) -> dict[str, float | int]:
    success_rate = (dataframe["status"] == "success").mean() * 100

    return {
        "total_requests": int(len(dataframe)),
        "success_rate_percent": round(float(success_rate), 2),
        "average_response_time_seconds": round(
            float(dataframe["response_time_seconds"].mean()),
            2,
        ),
        "average_feedback_score": round(
            float(dataframe["feedback_score"].mean()),
            2,
        ),
        "total_estimated_tokens": int(
            dataframe["estimated_tokens"].sum()
        ),
    }


def create_topic_report(dataframe: pd.DataFrame) -> pd.DataFrame:
    report = (
        dataframe.groupby("topic", as_index=False)
        .agg(
            request_count=("request_id", "count"),
            average_response_time=("response_time_seconds", "mean"),
            average_feedback=("feedback_score", "mean"),
            total_tokens=("estimated_tokens", "sum"),
        )
        .sort_values(
            by="request_count",
            ascending=False,
        )
    )

    return report.round(2)
```

---

## 3. Create the project entry point

Create `main.py`:

```python
import json
from pathlib import Path

from analyzer import (
    clean_ai_requests,
    create_summary,
    create_topic_report,
    load_ai_requests,
)


def main() -> None:
    data_file = Path("data") / "ai_requests.csv"
    report_directory = Path("reports")

    report_directory.mkdir(exist_ok=True)

    raw_data = load_ai_requests(data_file)
    cleaned_data = clean_ai_requests(raw_data)

    summary = create_summary(cleaned_data)
    topic_report = create_topic_report(cleaned_data)

    print("AI Dataset Summary")
    print(json.dumps(summary, indent=2))

    print("\nTopic Report")
    print(topic_report.to_string(index=False))

    summary_file = report_directory / "summary.json"
    topic_report_file = report_directory / "topic_report.csv"

    summary_file.write_text(
        json.dumps(summary, indent=2),
        encoding="utf-8",
    )
    topic_report.to_csv(topic_report_file, index=False)

    print(f"\nSaved: {summary_file}")
    print(f"Saved: {topic_report_file}")


if __name__ == "__main__":
    main()
```

Run:

```bash
python main.py
ruff check .
```

Your project generates:

```text
reports/
├── summary.json
└── topic_report.csv
```

---

# What you learned

```text
pd.read_csv()       → load tabular data
DataFrame            → table of rows and columns
head()               → preview data
copy()               → avoid changing original data
dropna()             → remove incomplete rows
groupby()            → analyze categories/topics
agg()                → calculate summary statistics
sort_values()        → rank results
to_csv()             → export a report
```

# 1–3 Hour Delivery Plan

## 1-hour MVP

```text
[ ] Generate CSV data
[ ] Load it with Pandas
[ ] Print summary statistics
[ ] Export one CSV report
```

## 2-hour version

```text
[ ] Clean string/numeric data
[ ] Validate required columns
[ ] Add topic grouping and ranking
[ ] Export JSON + CSV reports
```

## 3-hour corporate version

Create `tests/test_analyzer.py`:

```python
import pandas as pd

from analyzer import clean_ai_requests, create_summary


def test_clean_ai_requests_removes_invalid_rows() -> None:
    dataframe = pd.DataFrame(
        {
            "request_id": [1, 2],
            "topic": ["RAG", "AI Agents"],
            "learner_level": ["beginner", "advanced"],
            "model": ["demo-llm", "demo-llm"],
            "prompt_length": [100, "invalid"],
            "response_time_seconds": [1.5, 2.0],
            "estimated_tokens": [200, 300],
            "feedback_score": [5, 4],
            "status": ["success", "success"],
        }
    )

    cleaned = clean_ai_requests(dataframe)

    assert len(cleaned) == 1
    assert cleaned.iloc[0]["topic"] == "RAG"


def test_create_summary_calculates_success_rate() -> None:
    dataframe = pd.DataFrame(
        {
            "request_id": [1, 2],
            "topic": ["RAG", "RAG"],
            "learner_level": ["beginner", "beginner"],
            "model": ["demo-llm", "demo-llm"],
            "prompt_length": [100, 120],
            "response_time_seconds": [1.0, 2.0],
            "estimated_tokens": [200, 300],
            "feedback_score": [5, 4],
            "status": ["success", "failed"],
        }
    )

    summary = create_summary(dataframe)

    assert summary["total_requests"] == 2
    assert summary["success_rate_percent"] == 50.0
```

Run:

```bash
pytest -q
ruff check .
```

---

# Institute, Corporate, and Client Approach

## Institute submission

```text
[ ] Source code
[ ] Dataset CSV
[ ] Generated JSON and CSV reports
[ ] Test output screenshot
[ ] Explain groupby(), agg(), and data cleaning
```

## Corporate checklist

```text
[ ] Validate input columns
[ ] Handle invalid/missing numeric data
[ ] Preserve original data with copy()
[ ] Produce repeatable reports
[ ] Add tests
[ ] Avoid putting client/private data in Git
```

## Client explanation

> “This dashboard-analysis prototype helps us understand which AI topics are popular, response speed, token usage, feedback quality, and failure rates. It helps the team decide where to improve the AI experience.”

# Day 11 Assignment Extension

Add a function named:

```python
def create_model_report(dataframe: pd.DataFrame) -> pd.DataFrame:
```

It should group data by `model` and show:

```text
request count
average response time
success rate
average feedback score
```

When you finish the MVP, send me your output or any error.
