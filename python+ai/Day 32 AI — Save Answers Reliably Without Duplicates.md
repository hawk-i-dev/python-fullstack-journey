# Day 32 AI — Save Answers Reliably Without Duplicates

**Today’s project:** add a SQLite persistence layer that saves completed answers, records a save event, and handles repeated save requests safely.

Day 31 answered:

> “Did generation finish?”

Day 32 answers:

> “Was that completed answer committed to storage?”

**Time:** 1–3 hours.  
**Requirements:** Python standard library only. No API key.  
**Mac and Windows:** identical code.

## 1. The problem

Imagine this sequence:

1. The agent finishes generating.
2. The backend saves the answer.
3. The response confirming the save gets lost.
4. The application tries saving again.

Without protection, you might get two copies of the same answer.

We need two concepts:

- **Transaction:** related database changes succeed together or roll back together.
- **Idempotency:** repeating the same operation has the same intended effect, without creating duplicates.

For today:

```text
One completed run ID → one saved answer
```

A repeated run ID with **different content** must be rejected, not silently overwrite history.

## 2. Keep generation and saving separate

```text
Generation completed
        ↓
Attempt database transaction
        ├── committed → return save receipt
        └── failed    → report save failure
```

Do not change a successfully generated run to `FAILED` merely because saving failed. They are separate outcomes.

Likewise, do not tell the user “Saved” before the database commit succeeds.

## 3. Continue your existing practice lab

Inside your `day31_run_lifecycle` folder, keep the existing files and add:

```text
storage.py
persistence_demo.py
test_storage.py
```

We are extending the local exercise—not changing your main backend yet.

## 4. Build the persistence layer

Put this in `storage.py`:

```python
import sqlite3
from contextlib import contextmanager
from dataclasses import dataclass
from pathlib import Path

from lifecycle import AgentRun, RunStatus


@dataclass(frozen=True)
class SaveReceipt:
    message_id: int
    run_id: str
    created: bool


class AnswerStore:
    def __init__(self, database_path: Path):
        self.database_path = database_path

        with self._connection() as connection:
            connection.executescript("""
                CREATE TABLE IF NOT EXISTS answers (
                    id INTEGER PRIMARY KEY,
                    run_id TEXT NOT NULL UNIQUE,
                    content TEXT NOT NULL
                );

                CREATE TABLE IF NOT EXISTS save_events (
                    run_id TEXT PRIMARY KEY,
                    event_type TEXT NOT NULL,
                    FOREIGN KEY (run_id) REFERENCES answers(run_id)
                );
            """)

    @contextmanager
    def _connection(self):
        connection = sqlite3.connect(self.database_path)
        connection.row_factory = sqlite3.Row
        connection.execute("PRAGMA foreign_keys = ON")

        try:
            with connection:
                yield connection
        finally:
            connection.close()

    def save(self, run: AgentRun) -> SaveReceipt:
        if run.status != RunStatus.COMPLETED:
            raise ValueError("Only completed runs can be saved")

        content = run.final_answer

        if not isinstance(content, str) or not content.strip():
            raise ValueError("A completed text answer is required")

        with self._connection() as connection:
            cursor = connection.execute(
                """
                INSERT INTO answers (run_id, content)
                VALUES (?, ?)
                ON CONFLICT(run_id) DO NOTHING
                """,
                (run.run_id, content),
            )

            created = cursor.rowcount == 1

            saved = connection.execute(
                """
                SELECT id, content
                FROM answers
                WHERE run_id = ?
                """,
                (run.run_id,),
            ).fetchone()

            if saved is None:
                raise RuntimeError("Could not read the saved answer")

            if saved["content"] != content:
                raise ValueError(
                    "Run ID already exists with different content"
                )

            connection.execute(
                """
                INSERT INTO save_events (run_id, event_type)
                VALUES (?, ?)
                ON CONFLICT(run_id) DO NOTHING
                """,
                (run.run_id, "answer_saved"),
            )

            message_id = saved["id"]

        # The transaction has committed before we return this receipt.
        return SaveReceipt(
            message_id=message_id,
            run_id=run.run_id,
            created=created,
        )

    def counts(self):
        with self._connection() as connection:
            answers = connection.execute(
                "SELECT COUNT(*) FROM answers"
            ).fetchone()[0]

            events = connection.execute(
                "SELECT COUNT(*) FROM save_events"
            ).fetchone()[0]

        return {"answers": answers, "save_events": events}
```

### Understand the important parts

**`UNIQUE run_id`**

The database enforces one answer per run. A Python-only “check then insert” is not sufficient protection against concurrent requests.

**`ON CONFLICT(run_id) DO NOTHING`**

A repeated run ID does not insert another answer.

We then compare the stored content with the supplied content. Reusing the ID for a different answer raises an error.

**Parameterized SQL**

```python
VALUES (?, ?)
```

Values are passed separately from SQL syntax. Do not build SQL by interpolating user text.

**Transaction boundary**

Both operations happen inside one transaction:

```text
Insert answer + insert save event
```

If either fails, neither new change should remain committed.

**Connection lifecycle**

The context manager commits on normal completion, rolls back on an exception, and closes the connection.

**`frozen=True`**

The save receipt cannot normally have its fields reassigned. It represents the result of a completed storage operation.

## 5. Demonstrate duplicate-safe saving

Put this in `persistence_demo.py`:

```python
from pathlib import Path

from lifecycle import AgentRun
from storage import AnswerStore


def main():
    database_path = Path(__file__).resolve().parent / "day32_answers.db"
    store = AnswerStore(database_path)

    run = AgentRun.create()
    run.start()
    run.add_token("The meal allowance is INR 500 per day.")
    run.complete()

    first_receipt = store.save(run)
    second_receipt = store.save(run)

    print("First save:", first_receipt)
    print("Repeated save:", second_receipt)

    print(
        "Same message ID:",
        first_receipt.message_id == second_receipt.message_id,
    )

    print("Database totals:", store.counts())


if __name__ == "__main__":
    main()
```

Run with your virtual environment activated:

```bash
python persistence_demo.py
```

On Mac, outside the environment, use `python3` if `python` is unavailable.

Expected behaviour:

```text
First save: ... created=True
Repeated save: ... created=False
Same message ID: True
Database totals: {'answers': 1, 'save_events': 1}
```

The totals above assume a new database. Each execution creates a new run ID, so later executions add one more answer—not two.

**The key result is that saving the same run twice returns the same message ID.**

## 6. Test the rules

Put this in `test_storage.py`:

```python
import tempfile
import unittest
from pathlib import Path

from lifecycle import AgentRun
from storage import AnswerStore


def completed_run(text="Hello"):
    run = AgentRun.create()
    run.start()
    run.add_token(text)
    run.complete()
    return run


class AnswerStoreTests(unittest.TestCase):
    def setUp(self):
        self.temp_dir = tempfile.TemporaryDirectory()
        self.addCleanup(self.temp_dir.cleanup)

        self.database_path = Path(self.temp_dir.name) / "test.db"
        self.store = AnswerStore(self.database_path)

    def test_completed_answer_is_saved(self):
        receipt = self.store.save(completed_run())

        self.assertTrue(receipt.created)
        self.assertEqual(
            self.store.counts(),
            {"answers": 1, "save_events": 1},
        )

    def test_repeated_save_does_not_duplicate(self):
        run = completed_run()

        first = self.store.save(run)
        second = self.store.save(run)

        self.assertTrue(first.created)
        self.assertFalse(second.created)
        self.assertEqual(first.message_id, second.message_id)

        self.assertEqual(
            self.store.counts(),
            {"answers": 1, "save_events": 1},
        )

    def test_partial_run_is_rejected(self):
        run = AgentRun.create()
        run.start()
        run.add_token("Incomplete")

        with self.assertRaises(ValueError):
            self.store.save(run)

        self.assertEqual(
            self.store.counts(),
            {"answers": 0, "save_events": 0},
        )

    def test_same_id_with_different_content_is_rejected(self):
        original = completed_run("Original answer")
        self.store.save(original)

        conflicting = completed_run("Different answer")
        conflicting.run_id = original.run_id

        with self.assertRaises(ValueError):
            self.store.save(conflicting)

        self.assertEqual(
            self.store.counts(),
            {"answers": 1, "save_events": 1},
        )


if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

**Expected:** these four tests pass, along with your existing discovered tests.

Temporary databases keep tests independent from your demonstration data.

## 7. Important limits of today’s design

This is a local persistence exercise, not a production-ready multi-user service.

### Idempotency is scoped

We prevent duplicate **answer saves**.

This does not prevent duplicate:

- Model requests.
- Emails.
- Payments.
- Tool actions.

Each consequential operation needs its own carefully designed replay protection.

### A UUID is not authorization

Our local lab has no authentication.

When integrating with your real application, verify the authenticated user owns the conversation and run. A supplied run ID alone must not grant access.

### The save event is local

Writing an event row is part of the same database transaction.

Sending that event to an external service is a different problem. A database transaction cannot automatically make an external network operation atomic.

### A save receipt is not proof of frontend delivery

The commit may succeed even if the connection carrying the receipt breaks. That is why repeated saves must be safe.

## 8. Troubleshooting strategy

| Symptom | Investigate |
|---|---|
| Duplicate answers | Stable run IDs, database uniqueness, retry behaviour |
| Same ID contains different text | ID reuse or mutated run data |
| Answer saved without event | Transaction boundaries |
| Generation succeeded but save failed | Database path, permissions, locking, transaction errors |
| User sees “Saved” but record is absent | When the UI status is set; which database is being inspected |
| Retry creates another answer | Whether the retry incorrectly creates a new run ID |

**Retry the failed save with the same run ID—not a newly generated ID.**

That advice applies to this save operation, not blindly to the entire agent workflow.

## 9. Your assignment

Add tests proving:

1. Failed runs cannot be saved.
2. Cancelled runs cannot be saved.
3. Two different completed runs create two answers.
4. A new `AnswerStore` instance can read the same database counts.

**Challenge:** force insertion into `save_events` to fail in a temporary test database. Verify that the answer insertion rolls back too.

That test demonstrates atomicity—not merely duplicate prevention.

### Real-world explanation

- **Institute:** demonstrate a completed save, repeated save, and rejected partial run.
- **Corporate:** define transaction boundaries and document exactly which operation is idempotent.
- **Client:** distinguish “Generated,” “Saving,” “Saved,” and “Could not save.”

## EOD checklist

- [ ] Completed answers persist.
- [ ] Partial, failed, and cancelled runs are rejected.
- [ ] Repeated saves do not create duplicates.
- [ ] Reusing an ID with different content fails.
- [ ] You can explain transaction versus idempotency.
- [ ] You do not confuse a saved record with successful delivery.

**Remember: generation produces the answer; a committed transaction establishes that it was saved.**
