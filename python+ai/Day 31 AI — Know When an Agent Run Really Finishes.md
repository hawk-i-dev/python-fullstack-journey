# Day 31 AI — Know When an Agent Run Really Finishes

**Today’s project:** build an agent-run controller that distinguishes **streaming, success, failure, and cancellation**.

This addresses one of your master-quiz revision areas:

> Receiving some output does not mean the operation completed successfully.

**Time:** 1–3 hours.  
**Requirements:** Python only; no API key.  
**Mac and Windows:** identical code.

We’ll use simulated events first. This lets you understand and test the control logic without paying for model calls.

## 1. The problem we are solving

Imagine your assistant streams:

```text
Your meal allowance is…
```

Then the connection breaks.

Should the application:

- Display “Completed”?
- Save that fragment as the final answer?
- Enable a rating button?
- Automatically repeat a potentially consequential tool action?

Not automatically.

**Partial output is evidence of progress—not evidence of success.**

## 2. Think in states

A *state* describes where an operation currently stands.

Today’s controller uses:

```text
PENDING
   ↓ start
STREAMING
   ├── complete → COMPLETED
   ├── error    → FAILED
   └── cancel   → CANCELLED
```

`COMPLETED`, `FAILED`, and `CANCELLED` are **terminal states**: this run cannot continue accepting events.

To try again, create a new run.

### Our rules

1. Output tokens are accepted only while streaming.
2. Only explicit completion creates a final answer.
3. Failure preserves partial text for inspection, but not as a final answer.
4. Cancellation is different from success.
5. An unexpected stream ending counts as failure.
6. Events after a terminal state are rejected.

These rules are the **contract** of today’s component.

## 3. Create an isolated practice folder

Create a new folder named `day31_run_lifecycle` in your learning project.

Add:

```text
day31_run_lifecycle/
    lifecycle.py
    demo.py
    test_lifecycle.py
```

Do not replace your existing backend or streaming endpoint.

## 4. Build the controller

Put this in `lifecycle.py`:

```python
from dataclasses import dataclass
from enum import Enum
from uuid import uuid4


class RunStatus(str, Enum):
    PENDING = "pending"
    STREAMING = "streaming"
    COMPLETED = "completed"
    FAILED = "failed"
    CANCELLED = "cancelled"


TERMINAL_STATUSES = {
    RunStatus.COMPLETED,
    RunStatus.FAILED,
    RunStatus.CANCELLED,
}


@dataclass
class AgentRun:
    run_id: str
    status: RunStatus = RunStatus.PENDING
    partial_text: str = ""
    final_answer: str | None = None
    error_message: str | None = None

    @classmethod
    def create(cls):
        return cls(run_id=str(uuid4()))

    def _require_streaming(self):
        if self.status != RunStatus.STREAMING:
            raise ValueError(
                f"Expected streaming state, got {self.status.value}"
            )

    def start(self):
        if self.status != RunStatus.PENDING:
            raise ValueError("Only a pending run can start")

        self.status = RunStatus.STREAMING

    def add_token(self, text: str):
        self._require_streaming()

        if not isinstance(text, str):
            raise TypeError("Token text must be a string")

        self.partial_text += text

    def complete(self):
        self._require_streaming()

        if not self.partial_text.strip():
            raise ValueError("This text-only run needs a non-empty answer")

        self.final_answer = self.partial_text
        self.status = RunStatus.COMPLETED

    def fail(self, message: str):
        self._require_streaming()

        self.error_message = message
        self.status = RunStatus.FAILED

    def cancel(self):
        self._require_streaming()
        self.status = RunStatus.CANCELLED

    def finish_stream(self):
        if self.status == RunStatus.STREAMING:
            self.fail("Stream ended without a completion event")

    @property
    def can_publish_final_answer(self):
        return self.status == RunStatus.COMPLETED
```

### Understand the design

**`Enum`**

Restricts status values to named choices. You avoid inconsistent strings such as `"done"`, `"finished"`, and `"success"` scattered across your code.

**`dataclass`**

Groups the run’s data and generates routine initialization code.

**`_require_streaming()`**

Centralizes a rule used by several methods. The leading underscore signals an internal helper; it is not a security boundary.

**`partial_text` versus `final_answer`**

They represent different facts:

```text
partial_text → what has arrived so far
final_answer → the answer accepted after explicit completion
```

**`finish_stream()`**

Detects a subtle failure: the stream ends without reporting success or failure.

**Non-empty answer rule**

This is specific to our **text-only exercise**. A future tool-only or artifact-producing run may legitimately complete without a text answer and will need a different contract.

## 5. Simulate events

Put this in `demo.py`:

```python
from lifecycle import AgentRun


def run_simulation(events):
    run = AgentRun.create()
    run.start()

    try:
        for event in events:
            kind = event["type"]

            if kind == "token":
                run.add_token(event["text"])

            elif kind == "complete":
                run.complete()

            elif kind == "error":
                run.fail(event["message"])

            elif kind == "cancel":
                run.cancel()

            else:
                raise ValueError(f"Unknown event type: {kind}")

    finally:
        run.finish_stream()

    return run


def display_run(label, run):
    print(f"\n{label}")
    print(f"Status: {run.status.value}")
    print(f"Partial: {run.partial_text!r}")
    print(f"Final: {run.final_answer!r}")
    print(f"Error: {run.error_message!r}")
    print(f"Publish final: {run.can_publish_final_answer}")


if __name__ == "__main__":
    successful = run_simulation([
        {"type": "token", "text": "The allowance is "},
        {"type": "token", "text": "INR 500 per day."},
        {"type": "complete"},
    ])

    interrupted = run_simulation([
        {"type": "token", "text": "The allowance is "},
    ])

    cancelled = run_simulation([
        {"type": "token", "text": "Checking the policy…"},
        {"type": "cancel"},
    ])

    display_run("SUCCESSFUL RUN", successful)
    display_run("INTERRUPTED RUN", interrupted)
    display_run("CANCELLED RUN", cancelled)
```

Run from the practice folder with your environment active:

```bash
python demo.py
```

On Mac, outside the virtual environment, use `python3` if `python` is unavailable.

Expected key results:

```text
SUCCESSFUL RUN
Status: completed
Final: 'The allowance is INR 500 per day.'
Publish final: True

INTERRUPTED RUN
Status: failed
Final: None
Error: 'Stream ended without a completion event'
Publish final: False

CANCELLED RUN
Status: cancelled
Final: None
Publish final: False
```

The simulator is not running an LLM. It tests how your application handles events.

Also note: `finally` performs cleanup; it does **not** swallow exceptions. Malformed events still raise an error for the caller to handle.

## 6. Test the boundaries

Put this in `test_lifecycle.py`:

```python
import unittest

from lifecycle import AgentRun, RunStatus


class AgentRunTests(unittest.TestCase):
    def test_success_creates_final_answer(self):
        run = AgentRun.create()
        run.start()
        run.add_token("Hello")
        run.complete()

        self.assertEqual(run.status, RunStatus.COMPLETED)
        self.assertEqual(run.final_answer, "Hello")
        self.assertTrue(run.can_publish_final_answer)

    def test_partial_text_is_not_final(self):
        run = AgentRun.create()
        run.start()
        run.add_token("Half an answer")

        self.assertIsNone(run.final_answer)
        self.assertFalse(run.can_publish_final_answer)

    def test_unexpected_end_fails(self):
        run = AgentRun.create()
        run.start()
        run.add_token("Partial")
        run.finish_stream()

        self.assertEqual(run.status, RunStatus.FAILED)
        self.assertIsNone(run.final_answer)

    def test_cancellation_is_not_completion(self):
        run = AgentRun.create()
        run.start()
        run.cancel()

        self.assertEqual(run.status, RunStatus.CANCELLED)
        self.assertFalse(run.can_publish_final_answer)

    def test_tokens_after_completion_are_rejected(self):
        run = AgentRun.create()
        run.start()
        run.add_token("Finished")
        run.complete()

        with self.assertRaises(ValueError):
            run.add_token("Unexpected extra text")

    def test_cannot_start_twice(self):
        run = AgentRun.create()
        run.start()

        with self.assertRaises(ValueError):
            run.start()


if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

**Expected:** six passing tests.

## 7. How this connects to your existing app

Do not wire this directly into your backend yet. First understand the mapping:

| Application event | Controller action |
|---|---|
| Run accepted and generation starts | `start()` |
| Text fragment arrives | `add_token(text)` |
| Adapter confirms successful generation | `complete()` |
| Generation fails | `fail(message)` |
| Cancellation is confirmed | `cancel()` |
| Stream closes unexpectedly | `finish_stream()` |

Two important boundaries remain:

### Generation completed ≠ database save succeeded

Our controller tracks generation only.

Your application must separately handle persistence failures. If the frontend’s `done` event promises “saved successfully,” send it only after the save succeeds.

### Browser disconnected ≠ backend cancelled

The backend or provider may still be working. Clicking Stop also does not prove a consequential tool action was undone.

Cancellation must be implemented and confirmed across the relevant components.

## 8. Troubleshooting mindset

When a user says “the answer stopped,” investigate:

1. Did generation finish?
2. Did the backend receive the completion event?
3. Did saving succeed?
4. Did the frontend receive the terminal event?
5. Did the UI update its state?

These are separate questions. Avoid treating them as one generic “AI error.”

For sensitive systems, show users a safe message and record sanitized diagnostic details internally.

## 9. Your assignment

Add four more tests:

- A token before `start()` is rejected.
- An explicit error produces `FAILED`.
- Empty text cannot complete this text-only run.
- A new run has a different ID from the previous run.

Then demonstrate:

```text
start → token → error
start → token → complete
start → token → unexpected end
start → token → cancel
```

For each, explain whether a final answer should be published and why.

### Institute, corporate, and client approach

- **Institute:** explain the transitions and demonstrate invalid ones.
- **Corporate:** define what “completed” means across generation, storage, tools, and delivery.
- **Client:** display honest statuses—“Generating,” “Completed,” “Interrupted,” or “Cancelled”—without claiming unfinished work succeeded.

## EOD checklist

- [ ] Three simulated outcomes work.
- [ ] Ten unit tests pass.
- [ ] Partial output never becomes final without completion.
- [ ] Invalid transitions are rejected.
- [ ] You can distinguish generation, persistence, delivery, and cancellation.

**Remember: progress, completion, and successful delivery are different facts.**
