# Day 33 AI — Quiz and Programs

**Topic:** request idempotency, safe retries, payload fingerprints, and crash recovery limits.

**Time:** 1–3 hours.  
**Mac and Windows:** identical Python code. No API key needed.

## Part 1 — MCQ Quiz

Choose one answer per question.

### 1. What does an idempotency key identify?

A. A database connection  
B. One logical operation across retry attempts  
C. A user’s password  
D. Every request from every user  

### 2. A network retry repeats the same submission. Which key should it use?

A. A new random key every time  
B. Another user’s key  
C. The original submission’s key  
D. No key  

### 3. Why is our database key composed of `(user_id, request_key)`?

A. To scope request identity to a user  
B. To replace authentication  
C. To encrypt stored answers  
D. To guarantee model accuracy  

### 4. The same user submits the same key with different content. What should happen?

A. Return the previous answer without checking  
B. Execute both requests  
C. Silently overwrite the first request  
D. Reject the conflicting payload  

### 5. What is the purpose of a payload fingerprint?

A. To hide the user’s identity  
B. To detect changes to the payload associated with a key  
C. To authorize tool execution  
D. To guarantee that the payload contains no private data  

### 6. Why claim the request before starting work?

A. To establish which request is allowed to start the operation  
B. To make the model generate more quickly  
C. To avoid storing results  
D. To guarantee delivery to the browser  

### 7. What should a completed duplicate request do?

A. Generate another answer  
B. Delete the original result  
C. Return the stored result without executing work again  
D. Change the completed request to running  

### 8. Why keep the database claim transaction short?

A. SQLite does not support transactions  
B. Short transactions remove the need for uniqueness  
C. Generation cannot fail outside a transaction  
D. Slow work should not unnecessarily hold a database write transaction open  

### 9. Two requests return identical answer text. Does that prove work executed once?

A. Yes, identical text proves identical execution  
B. No; inspect execution counts or other execution evidence  
C. Yes, if both requests used Python  
D. Only if the answer contains citations  

### 10. What does `work.assert_called_once_with()` check in our test?

A. The mocked operation was called exactly once with no arguments  
B. The result was saved exactly twice  
C. The user was authenticated  
D. The generated answer is factually correct  

### 11. The process crashes after work executes but before completion is recorded. What may remain?

A. A guaranteed completed record  
B. A guaranteed failed record  
C. No possibility of side effects  
D. A `running` record despite work possibly having happened  

### 12. Why not automatically rerun every request that has been running for a long time?

A. Requests never become stuck  
B. Every retry is forbidden  
C. The original operation may already have produced effects  
D. A timestamp establishes successful cancellation  

### 13. A tool sends an email, then raises an exception. What does marking the request failed do?

A. Recall the email automatically  
B. Record failure; it does not reverse the email  
C. Prove no email was sent  
D. Make repeating the tool automatically safe  

### 14. Is a supplied user ID sufficient authorization in a real API?

A. No; derive identity from trusted authentication and check access  
B. Yes, if it is a long string  
C. Yes, if it accompanies an idempotency key  
D. Yes, if the browser button is disabled  

### 15. Which statement best describes our request guard?

A. It guarantees exactly-once execution across all external systems  
B. It prevents every possible concurrency and crash problem  
C. It makes all failed operations safe to retry  
D. It provides scoped duplicate protection with documented recovery limits  

## Part 2 — Programs

Continue in your `day33_request_guard` practice folder. Do not replace your main backend.

### Program 1 — Complete the request-guard tests

Add these five tests to `test_request_guard.py`:

1. Different keys execute work twice.
2. Different users using the same key execute independently.
3. Reordered dictionary keys produce the same fingerprint.
4. Failed requests are not automatically executed again.
5. A new guard instance replays a result from the same database.

For the fingerprint test:

```python
from request_guard import fingerprint
```

```python
def test_dictionary_order_does_not_change_fingerprint(self):
    first = {"question": "Hello", "language": "en"}
    second = {"language": "en", "question": "Hello"}

    self.assertEqual(fingerprint(first), fingerprint(second))
```

For the failed-request test, use:

```python
work = Mock(side_effect=RuntimeError("Simulated failure"))
```

Expected behaviour:

```text
First call → RuntimeError
Second call with the same user, key, and payload → RequestUnavailable
Work call count → 1
```

**Acceptance criterion:** all seven tests pass—the original two plus these five.

Run with your environment activated:

```bash
python -m unittest -v
```

### Program 2 — Prove a running duplicate cannot start work

**Goal:** test the running state without threads, sleeps, or timing assumptions.

While the first request’s `work()` is executing, have it attempt another call using the same user, key, and payload.

Starter:

```python
from request_guard import RequestUnavailable


def test_running_duplicate_is_rejected(self):
    payload = {"question": "Hello"}
    duplicate_work = Mock(return_value="Should not execute")

    def original_work():
        with self.assertRaises(RequestUnavailable):
            self.guard.execute(
                "alice",
                "key-running",
                payload,
                duplicate_work,
            )

        return "Original answer"

    result = self.guard.execute(
        "alice",
        "key-running",
        payload,
        original_work,
    )

    self.assertEqual(result, "Original answer")
    duplicate_work.assert_not_called()
```

Explain why this works:

> The original request’s claim must already be committed before `original_work()` begins.

**Acceptance criterion:** the duplicate is rejected while the original finishes successfully.

This verifies the application’s running-state behaviour. It is not a comprehensive multi-process concurrency test.

### Program 3 — Build a retry-versus-new-submission demo

Create `submission_demo.py`.

Use a `Mock` or counter to track actual executions.

Demonstrate:

| Submission | User | Key | Payload | Expected |
|---|---|---|---|---|
| Original | Alice | K1 | Question A | Execute |
| Network retry | Alice | K1 | Question A | Replay |
| Intentional new submission | Alice | K2 | Question A | Execute |
| Conflicting reuse | Alice | K1 | Question B | Reject |
| Independent user | Bob | K1 | Question A | Execute |

Use a temporary database.

Expected final execution count:

```text
Actual work executions: 3
```

Requirements:

- Catch `RequestConflict` for the conflicting submission.
- Verify that the conflict does not execute work.
- Use assertions—not only printed success messages.
- Give each mock result neutral wording; do not imply that this synthetic demo queried an actual model.

**Acceptance criterion:** all five scenarios behave as specified.

## Part 3 — EOD Mini-Project: Request Reliability Report

Combine your demonstrations into a console report:

```text
Completed duplicate: replayed
Changed payload: rejected
Running duplicate: rejected
Failed duplicate: blocked
Different user: independent execution
Persistence across guard instances: verified
```

Calculate each result from observed behaviour.

For each scenario, report:

```text
Expected:
Actual:
Work execution count:
Passed:
```

**Do not claim “exactly once across all systems.”** State the scope:

> “The guard prevents duplicate execution for recognized requests under this exercise’s rules. Crash recovery and external side effects need additional design.”

## Part 4 — Corporate and Client Investigation

A client reports:

> “The request timed out. We clicked Retry, and two emails were sent.”

Write a short investigation note answering:

1. Did the retry reuse the original idempotency key?
2. Was the request claimed before sending?
3. Could the first email have been sent before its result was recorded?
4. Did the email-sending operation itself have duplicate protection?
5. What evidence is needed before deciding whether another retry is safe?

Do not assume a timeout means the email was never sent.

## EOD Deliverables

- Seven basic request-guard tests.
- One running-duplicate test.
- A five-submission demo with exactly three work executions.
- A reliability report.
- A short duplicate-email investigation note.

**Remember: same logical operation → same key. New intentional operation → new key. Unknown outcome → investigate before replaying consequential work.**

Reply with your quiz answers as `1.B, 2.C, ...`. Share code or output separately for practical review.
