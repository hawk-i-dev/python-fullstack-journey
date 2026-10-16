# Day 20 AI — Observability, Traces, and Evals

Today you will learn how professionals answer:

> “Why did the agent give this answer, choose this tool, or fail this request?”

The answer is not guessing. It is **traces + evaluation cases + repeatable tests**.

A trace records one workflow run: routing, tool calls, guardrails, approvals, duration, and outcome. Once you know what “good” means, use a dataset of test cases to detect regressions after changing prompts, tools, or routing logic. [Official OpenAI agent-evaluation guide](https://developers.openai.com/api/docs/guides/agent-evals)

## Today’s project

Add local tracing and a small evaluation suite to the Finance Assistant Agent.

```text
User request
   ↓
Trace starts
   ↓
Route → plan → guardrail → tool execution
   ↓
Trace saved as JSON
   ↓
Evaluation checks expected behavior
```

No Mac/Windows code difference.

## 1. Create `agent_tracing.py`

```python
import json
import time
import uuid
from dataclasses import asdict, dataclass, field
from datetime import datetime, timezone
from pathlib import Path
from typing import Any


@dataclass
class TraceEvent:
    name: str
    data: dict[str, Any]
    timestamp: str = field(
        default_factory=lambda: datetime.now(timezone.utc).isoformat()
    )


@dataclass
class WorkflowTrace:
    trace_id: str
    workflow_name: str
    started_at: str
    events: list[TraceEvent] = field(default_factory=list)
    duration_ms: int | None = None

    def add_event(self, name: str, data: dict[str, Any]) -> None:
        self.events.append(TraceEvent(name=name, data=data))


def start_trace(workflow_name: str) -> tuple[WorkflowTrace, float]:
    trace = WorkflowTrace(
        trace_id=str(uuid.uuid4()),
        workflow_name=workflow_name,
        started_at=datetime.now(timezone.utc).isoformat(),
    )

    return trace, time.perf_counter()


def finish_trace(
    trace: WorkflowTrace,
    started_at: float,
    file_path: Path,
) -> None:
    trace.duration_ms = round((time.perf_counter() - started_at) * 1000)

    file_path.parent.mkdir(parents=True, exist_ok=True)

    with file_path.open("w", encoding="utf-8") as file:
        json.dump(asdict(trace), file, indent=2)
```

## 2. Create `observed_workflow.py`

```python
from pathlib import Path
from typing import Any

from agent_tracing import finish_trace, start_trace
from agent_workflow import run_finance_workflow


def run_observed_workflow(
    *,
    question: str,
    chunks: list[dict[str, Any]],
    expense_title: str | None = None,
    monthly_limit: float | None = None,
    spent: float | None = None,
) -> dict[str, Any]:
    trace, started_at = start_trace("finance_assistant")

    trace.add_event(
        "workflow_started",
        {
            "question_length": len(question),
        },
    )

    result = run_finance_workflow(
        question=question,
        chunks=chunks,
        expense_title=expense_title,
        monthly_limit=monthly_limit,
        spent=spent,
    )

    trace.add_event(
        "workflow_finished",
        {
            "ok": result["ok"],
            "intent": result["intent"],
            "result_count": len(result["results"]),
        },
    )

    trace_file = Path("data/traces") / f"{trace.trace_id}.json"

    finish_trace(
        trace=trace,
        started_at=started_at,
        file_path=trace_file,
    )

    result["trace_file"] = str(trace_file)
    return result
```

Notice: we log `question_length`, not the complete user question. In production, avoid storing secrets, sensitive financial details, personal information, or tokens unless there is a clear approved policy.

## 3. Create `main_observed_demo.py`

```python
from observed_workflow import run_observed_workflow


result = run_observed_workflow(
    question="Categorize this expense and check my remaining budget.",
    expense_title="Uber ride to office",
    monthly_limit=30000,
    spent=18450,
    chunks=[],
)

print(result)
print("Trace was saved to:", result["trace_file"])
```

Run:

```bash
python main_observed_demo.py
```

Open the generated JSON file in `data/traces/` and check:

- unique `trace_id`
- workflow name
- start and finish events
- duration
- safe summary data

## 4. Create an evaluation dataset

Create `eval_cases.py`:

```python
EVAL_CASES = [
    {
        "name": "budget calculation",
        "question": "What is my remaining budget?",
        "monthly_limit": 30000,
        "spent": 18450,
        "expected_intent": "calculate_budget",
        "expected_ok": True,
    },
    {
        "name": "missing budget details",
        "question": "What is my remaining budget?",
        "expected_intent": "budget_check",
        "expected_ok": False,
    },
    {
        "name": "unsupported money transfer",
        "question": "Transfer ₹5000 to my friend.",
        "expected_intent": "unsupported",
        "expected_ok": False,
    },
]
```

## 5. Create `run_evals.py`

```python
from eval_cases import EVAL_CASES
from agent_workflow import run_finance_workflow


def run_evaluations() -> None:
    passed = 0

    for case in EVAL_CASES:
        result = run_finance_workflow(
            question=case["question"],
            chunks=[],
            monthly_limit=case.get("monthly_limit"),
            spent=case.get("spent"),
        )

        success = (
            result["intent"] == case["expected_intent"]
            and result["ok"] == case["expected_ok"]
        )

        status = "PASS" if success else "FAIL"

        print(f"{status}: {case['name']}")
        print(f"  Expected intent: {case['expected_intent']}")
        print(f"  Actual intent:   {result['intent']}")

        if success:
            passed += 1

    print(f"\nScore: {passed}/{len(EVAL_CASES)}")


if __name__ == "__main__":
    run_evaluations()
```

Run:

```bash
python run_evals.py
```

## What you should learn

| Concept | Meaning |
|---|---|
| Trace | Record of one real workflow run |
| Event | A meaningful step inside that run |
| Evaluation case | Input plus expected behavior |
| Regression | Something that worked before but fails after a change |
| Eval score | Passed cases ÷ total cases |

## Corporate and client mindset

- Inspect traces when debugging a failure.
- Turn real failures into permanent evaluation cases.
- Run evaluations before merging or deploying.
- Measure routing accuracy, tool correctness, refusal correctness, approval behavior, latency, and cost.
- Log safely—never use observability as an excuse to collect secrets.

## Day 20 assignment

Add three evaluation cases:

1. `Grocery shopping` is categorized as `Food`.
2. A negative `spent` amount is rejected.
3. A request to delete an expense returns `pending_approval`, not execution.

Then add one test that confirms a trace file contains both:

```python
"workflow_started"
"workflow_finished"
```

By the end of Day 20, you should understand that a production AI agent is continuously improved through evidence: inspect traces, define failures, add eval cases, and prevent regressions.
