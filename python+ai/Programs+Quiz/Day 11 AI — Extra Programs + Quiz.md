# Day 11 AI — Extra Programs + Quiz

**Pandas code and commands are the same on Mac and Windows** after activating `.venv`.

From the Day 11 project folder, run:

```bash
python main.py
pytest -q
ruff check .
```

## Program 1 — Filter successful AI requests

Create `filter_requests.py`:

```python
from pathlib import Path

import pandas as pd


data_file = Path("data") / "ai_requests.csv"

dataframe = pd.read_csv(data_file)

successful_requests = dataframe[
    dataframe["status"] == "success"
]

fast_requests = successful_requests[
    successful_requests["response_time_seconds"] < 2
]

print("Successful requests:")
print(successful_requests[["request_id", "topic", "response_time_seconds"]])

print("\nSuccessful requests under 2 seconds:")
print(fast_requests[["request_id", "topic", "response_time_seconds"]])
```

Run:

```bash
python filter_requests.py
```

## Program 2 — Model performance report

Add this function to `analyzer.py`:

```python
def create_model_report(dataframe: pd.DataFrame) -> pd.DataFrame:
    report_data = dataframe.copy()

    report_data["is_success"] = (
        report_data["status"] == "success"
    )

    report = (
        report_data.groupby("model", as_index=False)
        .agg(
            request_count=("request_id", "count"),
            average_response_time=("response_time_seconds", "mean"),
            success_rate_percent=("is_success", "mean"),
            average_feedback=("feedback_score", "mean"),
        )
        .sort_values(
            by="average_feedback",
            ascending=False,
        )
    )

    report["success_rate_percent"] = (
        report["success_rate_percent"] * 100
    )

    return report.round(2)
```

Add this import to `main.py`:

```python
from analyzer import create_model_report
```

Then, after `topic_report` is created, add:

```python
model_report = create_model_report(cleaned_data)

print("\nModel Report")
print(model_report.to_string(index=False))

model_report.to_csv(
    report_directory / "model_report.csv",
    index=False,
)
```

## Program 3 — Data-quality report

Create `data_quality.py`:

```python
from pathlib import Path

import pandas as pd


def create_data_quality_report(
    dataframe: pd.DataFrame,
) -> dict[str, int]:
    return {
        "total_rows": len(dataframe),
        "missing_values": int(dataframe.isna().sum().sum()),
        "duplicate_rows": int(dataframe.duplicated().sum()),
        "negative_response_times": int(
            (dataframe["response_time_seconds"] < 0).sum()
        ),
        "invalid_feedback_scores": int(
            (
                (dataframe["feedback_score"] < 1)
                | (dataframe["feedback_score"] > 5)
            ).sum()
        ),
    }


data_file = Path("data") / "ai_requests.csv"
dataframe = pd.read_csv(data_file)

report = create_data_quality_report(dataframe)

print("Data Quality Report")
for key, value in report.items():
    print(f"{key}: {value}")
```

## Program 4 — Test the model report

Create `tests/test_model_report.py`:

```python
import pandas as pd

from analyzer import create_model_report


def test_create_model_report_calculates_metrics() -> None:
    dataframe = pd.DataFrame(
        {
            "request_id": [1, 2, 3],
            "topic": ["RAG", "RAG", "AI Agents"],
            "learner_level": ["beginner", "beginner", "advanced"],
            "model": ["model-a", "model-a", "model-b"],
            "prompt_length": [100, 120, 150],
            "response_time_seconds": [1.0, 3.0, 2.0],
            "estimated_tokens": [200, 300, 400],
            "feedback_score": [5, 3, 4],
            "status": ["success", "failed", "success"],
        }
    )

    report = create_model_report(dataframe)

    model_a = report[report["model"] == "model-a"].iloc[0]

    assert model_a["request_count"] == 2
    assert model_a["average_response_time"] == 2.0
    assert model_a["success_rate_percent"] == 50.0
    assert model_a["average_feedback"] == 4.0
```

Run:

```bash
pytest -q
ruff check .
```

# Day 11 AI MCQ Quiz

Reply like:

```text
1.B 2.A 3.C ...
```

1. What is Pandas mainly used for?

   A. Designing web pages  
   B. Loading, cleaning, analyzing, and reporting tabular data  
   C. Managing Git branches  
   D. Running LLM APIs only  

2. What is a Pandas `DataFrame`?

   A. A table of rows and columns  
   B. A Python error  
   C. A virtual environment  
   D. A Git commit  

3. Which function commonly loads a CSV file?

   A. `pd.load_csv()`  
   B. `pd.open_csv()`  
   C. `pd.read_csv()`  
   D. `pd.import_csv()`  

4. What does `dataframe.head()` usually show?

   A. The first few rows  
   B. The final row only  
   C. Only numeric columns  
   D. A chart  

5. Why use `dataframe.copy()` before modifying data?

   A. To protect the original DataFrame from unintended changes  
   B. To delete duplicate rows  
   C. To convert data into JSON  
   D. To install Pandas  

6. What does `pd.to_numeric(..., errors="coerce")` do with invalid numeric text?

   A. Converts it to zero  
   B. Converts it to missing data (`NaN`)  
   C. Deletes the full file  
   D. Raises no matter what  

7. What does `dropna()` commonly do?

   A. Removes rows or columns with missing values  
   B. Adds missing values  
   C. Sorts the DataFrame  
   D. Creates a new virtual environment  

8. What does `groupby("topic")` allow you to do?

   A. Group records by topic for analysis  
   B. Delete the `topic` column  
   C. Call an LLM  
   D. Encrypt topics  

9. What does `.agg()` do after `groupby()`?

   A. Performs summary calculations such as count, mean, and sum  
   B. Removes every row  
   C. Starts a Git branch  
   D. Converts CSV into Python code  

10. Which calculation gives an AI request success rate?

   A. `dataframe["status"].mean()`  
   B. `(dataframe["status"] == "success").mean() * 100`  
   C. `dataframe["status"].sum()`  
   D. `len(dataframe["status"]) * 100`  

11. What does `sort_values(by="request_count", ascending=False)` do?

   A. Sorts from highest request count to lowest  
   B. Deletes the count column  
   C. Creates random ordering  
   D. Sorts alphabetically only  

12. Which method exports a DataFrame to CSV?

   A. `to_json()`  
   B. `save_csv()`  
   C. `to_csv()`  
   D. `export_table()`  

13. Why check duplicate rows?

   A. Duplicate records can distort analysis and reports  
   B. They always improve accuracy  
   C. They make CSV files smaller  
   D. They replace missing values  

14. In a corporate AI analytics project, what should be checked before analyzing client data?

   A. Only the file name  
   B. Data quality, privacy, permissions, and required columns  
   C. Only the number of rows  
   D. The color of the spreadsheet  

15. Which client-facing statement is most accurate?

   A. “The data proves the AI is always correct.”  
   B. “This report helps identify usage trends, failures, speed, and feedback patterns.”  
   C. “The model does not need monitoring.”  
   D. “Testing is unnecessary for analytics.”
