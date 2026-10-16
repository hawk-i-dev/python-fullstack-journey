## Day 20 AI — Programs and Quiz

A trace helps you inspect one agent run; evaluation cases let you measure behavior repeatedly and catch regressions after code or prompt changes. [Official OpenAI agent-evaluation guide](https://developers.openai.com/api/docs/guides/agent-evals)

### Program 1: Trace summary reader

Create `trace_summary.py`:

```python
import json
from pathlib import Path


def show_trace_summary(file_path: Path) -> None:
    trace = json.loads(file_path.read_text(encoding="utf-8"))

    print("Trace ID:", trace["trace_id"])
    print("Workflow:", trace["workflow_name"])
    print("Duration:", trace["duration_ms"], "ms")
    print("Events:")

    for event in trace["events"]:
        print(f"- {event['name']}: {event['data']}")


trace_files = sorted(Path("data/traces").glob("*.json"))

if not trace_files:
    print("No trace files found. Run main_observed_demo.py first.")
else:
    show_trace_summary(trace_files[-1])
```

Run:

```bash
python trace_summary.py
```

### Program 2: Add an evaluation score percentage

Update the end of `run_evals.py`:

```python
total = len(EVAL_CASES)
score_percent = (passed / total) * 100 if total else 0

print(f"\nScore: {passed}/{total}")
print(f"Percentage: {score_percent:.1f}%")

if score_percent == 100:
    print("Status: Ready for this small local test suite.")
else:
    print("Status: Investigate failed cases before changing production behavior.")
```

### Program 3: Test trace file contents

Create `tests/test_agent_tracing.py`:

```python
import json
from pathlib import Path

from agent_tracing import finish_trace, start_trace


def test_trace_file_contains_start_and_finish_events(tmp_path: Path):
    trace, started_at = start_trace("test_workflow")

    trace.add_event("workflow_started", {"question_length": 10})
    trace.add_event("workflow_finished", {"ok": True})

    output_file = tmp_path / "trace.json"

    finish_trace(
        trace=trace,
        started_at=started_at,
        file_path=output_file,
    )

    saved_trace = json.loads(output_file.read_text(encoding="utf-8"))

    event_names = [
        event["name"]
        for event in saved_trace["events"]
    ]

    assert "workflow_started" in event_names
    assert "workflow_finished" in event_names
    assert saved_trace["duration_ms"] is not None
```

Run:

```bash
pytest
```

### Program 4: Add the Day 20 assignment cases

Add these to `EVAL_CASES`:

```python
{
    "name": "grocery is food",
    "question": "Categorize this expense.",
    "expense_title": "Grocery shopping",
    "expected_intent": "categorize_expense",
    "expected_ok": True,
},
{
    "name": "negative spending is rejected",
    "question": "What is my remaining budget?",
    "monthly_limit": 30000,
    "spent": -100,
    "expected_intent": "calculate_budget",
    "expected_ok": False,
},
{
    "name": "deletion needs approval",
    "question": "Delete expense number 42.",
    "expected_intent": "unsupported",
    "expected_ok": False,
},
```

---

## Day 20 AI Quiz

Reply like: `1.B, 2.A, ...`

1. What is an agent trace?

   A. A password file  
   B. A record of one workflow run and its steps  
   C. A Python package  
   D. A database deletion  

2. What is the main purpose of observability?

   A. Understand, debug, and improve workflow behavior  
   B. Hide failures  
   C. Remove all tests  
   D. Store secrets  

3. What is an evaluation case?

   A. A random user message  
   B. Input plus expected agent behavior  
   C. An API key  
   D. A frontend route  

4. What is a regression?

   A. A previously working behavior that breaks after a change  
   B. A successful test  
   C. A new Python file  
   D. A user approval  

5. What should a trace record?

   A. API keys  
   B. Passwords  
   C. Events, safe metadata, duration, and outcome  
   D. Bank PINs  

6. Why should sensitive data be redacted from traces?

   A. To protect users and reduce data exposure  
   B. To make logs invalid  
   C. To slow debugging  
   D. To avoid tests  

7. When debugging a new agent issue, what should you inspect first?

   A. A representative trace  
   B. Random CSS  
   C. A deleted log  
   D. An API key  

8. Why use an evaluation dataset?

   A. To repeatedly test expected behavior across cases  
   B. To remove all unit tests  
   C. To avoid routing  
   D. To give tools unlimited access  

9. Which metric is useful for agent quality?

   A. Evaluation pass rate  
   B. Number of folders  
   C. Screen brightness  
   D. Keyboard model  

10. What should happen after a real production failure is understood?

   A. Ignore it  
   B. Add it as a regression evaluation case  
   C. Delete all traces  
   D. Disable testing  

11. What can traces help you discover?

   A. Wrong tool selection or failed guardrails  
   B. Only font size  
   C. Only git branch name  
   D. Nothing about workflows  

12. Why should evals run before deployment?

   A. To catch behavior changes before users are affected  
   B. To increase errors  
   C. To avoid code review  
   D. To remove logs  

13. What should an evaluation case check for a sensitive action?

   A. That it executes silently  
   B. That it requires approval or is safely rejected  
   C. That secrets are printed  
   D. That validation is skipped  

14. What is the correct improvement cycle?

   A. Change code → deploy without testing  
   B. Inspect trace → identify failure → add eval → fix → rerun evals  
   C. Delete failures  
   D. Trust one successful response  

15. Why are local deterministic evals useful?

   A. They are fast, repeatable, and do not require a paid API call  
   B. They make all AI calls unnecessary  
   C. They remove the need for production monitoring  
   D. They store user secrets
