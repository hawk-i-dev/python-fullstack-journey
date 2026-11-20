# Day 33 AI — Safe Retries Without Running the Agent Twice

**Today’s project:** build a request guard that remembers whether an operation is already running or has already completed.

Day 32 prevented duplicate **saved answers**.

But consider this:

```text
Request arrives → model generates → answer saved
Request repeats → model generates AGAIN → duplicate save prevented
```

The database contains one answer, but you still performed generation twice.

**Today we move duplicate protection to the start of the workflow.**

**Time:** 1–3 hours.  
**Requirements:** Python and SQLite only. No API key.  
**Mac and Windows:** identical code.

## 1. The mental model

Give each logical request an **idempotency key**.

For example:

```text
User submits a question → key K1

Network retry of that submission → reuse K1

User intentionally submits a new question → new key K2
```

The server checks a durable request record:

| Existing record | Behaviour |
|---|---|
| No matching key | Claim it and start work |
| Same key, different payload | Reject the conflict |
| Same key, still running | Do not start another execution |
| Same key, completed | Return the stored result |
| Same key, failed | Apply an explicit recovery policy |

Our exercise uses a conservative policy: failed requests are **not automatically replayed**.

## 2. Three identities—not one

Keep these concepts separate:

- **User ID:** who owns the request.
- **Idempotency key:** identifies one logical operation across retries.
- **Run ID:** identifies an execution attempt.

Today’s uniqueness rule is:

```text
(user_id, idempotency_key)
```

Two users can independently use the same key without sharing results.

**Important:** this local exercise accepts a user ID as a function argument. In your real API, derive it from verified authentication—not an editable request field.

## 3. Why store a payload fingerprint?

Suppose the same key is used for:

```text
First request:  "Summarize policy A"
Second request: "Summarize policy B"
```

Returning the first result for the second request would be incorrect.

We compute a fingerprint of the payload to detect this conflict.

A fingerprint is not encryption. It does not make sensitive content safe to publish.

## 4. Create a separate practice folder

Create a new folder named `day33_request_guard`.

Add:

```text
request_guard.py
demo.py
test_request_guard.py
```

Keep Days 31–32 unchanged.

## 5. Build the request guard

Put this in `request_guard.py`:

```python
import hashlib
import json
import sqlite3
from contextlib import contextmanager
from pathlib import Path
from typing import Callable


class RequestConflict(ValueError):
    pass


class RequestUnavailable(RuntimeError):
    pass


def fingerprint(payload: dict) -> str:
    serialized = json.dumps(
        payload,
        sort_keys=True,
        separators=(",", ":"),
        ensure_ascii=False,
        allow_nan=False,
    )

    return hashlib.sha256(serialized.encode("utf-8")).hexdigest()


class RequestGuard:
    def __init__(self, database_path: Path):
        self.database_path = database_path

        with self._connection() as connection:
            connection.execute("""
                CREATE TABLE IF NOT EXISTS requests (
                    user_id TEXT NOT NULL,
                    request_key TEXT NOT NULL,
                    fingerprint TEXT NOT NULL,
                    status TEXT NOT NULL CHECK (
                        status IN ('running', 'completed', 'failed')
                    ),
                    result TEXT,
                    PRIMARY KEY (user_id, request_key)
                )
            """)

    @contextmanager
    def _connection(self):
        connection = sqlite3.connect(self.database_path)
        connection.row_factory = sqlite3.Row

        try:
            with connection:
                yield connection
        finally:
            connection.close()

    def execute(
        self,
        user_id: str,
        request_key: str,
        payload: dict,
        work: Callable[[], str],
    ) -> str:
        if not user_id.strip() or not request_key.strip():
            raise ValueError("User ID and request key are required")

        payload_hash = fingerprint(payload)

        # Claim the request in a short database transaction.
        with self._connection() as connection:
            cursor = connection.execute(
                """
                INSERT INTO requests (
                    user_id, request_key, fingerprint, status
                )
                VALUES (?, ?, ?, 'running')
                ON CONFLICT(user_id, request_key) DO NOTHING
                """,
                (user_id, request_key, payload_hash),
            )

            claimed = cursor.rowcount == 1

            record = connection.execute(
                """
                SELECT fingerprint, status, result
                FROM requests
                WHERE user_id = ? AND request_key = ?
                """,
                (user_id, request_key),
            ).fetchone()

            if record["fingerprint"] != payload_hash:
                raise RequestConflict(
                    "This key was already used with a different payload"
                )

            if not claimed:
                if record["status"] == "completed":
                    return record["result"]

                raise RequestUnavailable(
                    f"Request is {record['status']}; not executing again"
                )

        # The claim is committed before slow work starts.
        try:
            result = work()

            if not isinstance(result, str):
                raise TypeError("This exercise expects a string result")

        except Exception:
            with self._connection() as connection:
                connection.execute(
                    """
                    UPDATE requests
                    SET status = 'failed'
                    WHERE user_id = ? AND request_key = ?
                    """,
                    (user_id, request_key),
                )
            raise

        with self._connection() as connection:
            connection.execute(
                """
                UPDATE requests
                SET status = 'completed', result = ?
                WHERE user_id = ? AND request_key = ?
                """,
                (result, user_id, request_key),
            )

        return result
```

## 6. Understand the important decisions

### Claim before execution

The request record is committed **before** `work()` runs.

Otherwise, two requests could both decide that they should start work.

The database’s composite primary key—not just a Python check—enforces uniqueness.

### Keep database transactions short

Do not hold a database write transaction open while waiting for slow generation.

Our sequence is:

```text
Short claim transaction
→ work outside transaction
→ short completion transaction
```

### Store the result for replay

A completed retry returns the original stored result. It does not call `work()` again.

### Compare normalized payloads

`sort_keys=True` means these dictionaries have the same fingerprint:

```python
{"question": "Hello", "language": "en"}
{"language": "en", "question": "Hello"}
```

This is normalization for our limited JSON exercise—not universal semantic equivalence.

Before fingerprinting real requests, validate them and include all fields that affect the operation.

## 7. Demonstrate the protection

Put this in `demo.py`:

```python
from pathlib import Path
from tempfile import TemporaryDirectory

from request_guard import RequestGuard


def main():
    executions = 0

    def simulated_generation():
        nonlocal executions
        executions += 1
        return "The meal allowance is INR 500 per day."

    with TemporaryDirectory() as directory:
        guard = RequestGuard(Path(directory) / "requests.db")
        payload = {"question": "What is the meal allowance?"}

        first = guard.execute(
            "alice",
            "submission-001",
            payload,
            simulated_generation,
        )

        repeated = guard.execute(
            "alice",
            "submission-001",
            payload,
            simulated_generation,
        )

        print("First result:", first)
        print("Repeated result:", repeated)
        print("Generation executions:", executions)

        assert first == repeated
        assert executions == 1


if __name__ == "__main__":
    main()
```

Run with your virtual environment activated:

```bash
python demo.py
```

On Mac, outside the environment, use `python3` if needed.

Expected:

```text
First result: The meal allowance is INR 500 per day.
Repeated result: The meal allowance is INR 500 per day.
Generation executions: 1
```

**The important evidence is the execution counter—not merely identical answer text.**

Two executions could coincidentally produce the same answer.

## 8. Write the first test

Put this in `test_request_guard.py`:

```python
import tempfile
import unittest
from pathlib import Path
from unittest.mock import Mock

from request_guard import RequestConflict, RequestGuard


class RequestGuardTests(unittest.TestCase):
    def setUp(self):
        self.directory = tempfile.TemporaryDirectory()
        self.addCleanup(self.directory.cleanup)

        self.guard = RequestGuard(
            Path(self.directory.name) / "test.db"
        )

    def test_completed_request_is_replayed(self):
        work = Mock(return_value="Answer")
        payload = {"question": "Hello"}

        first = self.guard.execute("alice", "key-1", payload, work)
        second = self.guard.execute("alice", "key-1", payload, work)

        self.assertEqual(first, second)
        work.assert_called_once_with()

    def test_changed_payload_is_rejected(self):
        work = Mock(return_value="Answer")

        self.guard.execute(
            "alice", "key-1", {"question": "First"}, work
        )

        with self.assertRaises(RequestConflict):
            self.guard.execute(
                "alice", "key-1", {"question": "Different"}, work
            )

        work.assert_called_once_with()


if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

`Mock` lets you verify whether the protected operation was actually called.

## 9. The hardest boundary: a process crash

Suppose:

```text
Claim committed
→ work executes
→ process crashes before result is recorded
```

The database still says `running`.

**Do not assume “running for a long time” means nothing happened.**

Our exercise deliberately leaves this request blocked. Production recovery needs additional design, such as timestamps, ownership/leases, reconciliation, and operation-specific replay protection.

Also:

> This guard does not provide universal “exactly once” execution.

If `work()` sends an email and then raises an exception, the email may already have been sent. Marking the request failed does not reverse it.

Today’s demonstration has no external side effects.

## 10. Your assignment

Add tests proving:

1. Different keys execute work twice.
2. Different users using the same key get independent executions.
3. Reordered dictionary keys produce the same fingerprint.
4. A failed request is not automatically executed again.
5. A new guard instance using the same database replays a completed result.

For the failed-request test:

```text
First call → work raises an exception
Second call, same key → RequestUnavailable
Work execution count → 1
```

### Real-world troubleshooting

A client says:

> “Clicking Retry generated the answer twice.”

Investigate:

- Did the retry reuse the original key?
- Was the key scoped to the authenticated user?
- Was the request claimed before generation?
- Did the application check an existing result before starting work?
- Did a crash leave an uncertain operation behind?

Do not fix this only by disabling the frontend button. Backend protection is still needed.

## EOD checklist

- [ ] Same user + same key + same payload executes once and replays.
- [ ] Changed payload under the same key is rejected.
- [ ] Different users remain isolated.
- [ ] Failed requests are not blindly repeated.
- [ ] Request records survive a new guard instance.
- [ ] You can explain the crash window and the limits of this design.

**Remember: a retry is another delivery attempt—not necessarily a new logical operation.**
