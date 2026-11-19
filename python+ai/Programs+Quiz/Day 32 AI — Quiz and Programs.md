# Day 32 AI — Quiz and Programs

**Topic:** transactions, idempotency, reliable persistence, and save confirmation.

**Time:** 1–3 hours.  
**Mac and Windows:** identical Python code. No API key required.

## Part 1 — MCQ Quiz

Choose one answer per question.

### 1. The agent finishes generating, but the database save fails. What is true?

A. Generation and saving both failed  
B. Generation succeeded; persistence failed  
C. The answer was saved because generation completed  
D. The browser must save the answer automatically  

### 2. What is a database transaction?

A. A group of changes that commit together or roll back together  
B. A mechanism that retries every operation  
C. A copy of the database  
D. A guarantee that external network requests succeed  

### 3. What does idempotency mean for today’s save operation?

A. Every save creates a new answer  
B. Every retry generates a new run ID  
C. Repeating the same save does not create duplicate answers  
D. Failed operations are counted as successful  

### 4. Why place a `UNIQUE` constraint on `run_id`?

A. To authenticate users  
B. To make answer content private  
C. To prevent model hallucinations  
D. To enforce one saved answer per run at the database level  

### 5. The same run ID is submitted again with identical content. What should our store do?

A. Delete the original answer  
B. Return its existing message ID without creating a duplicate  
C. Insert a second answer  
D. Generate another answer using the model  

### 6. The same run ID is submitted with different content. What should happen?

A. Reject the conflicting save  
B. Silently overwrite the original  
C. Store both under the same ID  
D. Ignore the difference and claim success  

### 7. Why use parameterized SQL?

A. It guarantees faster queries  
B. It replaces authorization  
C. It keeps values separate from SQL syntax  
D. It prevents database connection failures  

### 8. Why insert the answer and its save event in one transaction?

A. To send the event to every external service  
B. To avoid validating run status  
C. To guarantee frontend delivery  
D. To prevent only one of those related changes being committed  

### 9. What should happen if inserting the save event fails before commit?

A. Keep the new answer and ignore the event  
B. Roll back the new answer insertion as well  
C. Mark the frontend as saved  
D. Automatically regenerate the answer  

### 10. When should the backend return a successful save receipt?

A. After the transaction commits  
B. When generation starts  
C. Immediately before attempting the database write  
D. When the first token arrives  

### 11. A UUID run ID establishes:

A. That the requester owns the run  
B. That the answer is accurate  
C. An identifier—not authorization  
D. That the run completed successfully  

### 12. Our answer-save idempotency also guarantees:

A. Emails cannot be sent twice  
B. Payments cannot be charged twice  
C. Model requests cannot be repeated  
D. None of these; those operations need their own replay protection  

### 13. The database commit succeeds, but the save receipt is lost in transit. What is a suitable next step?

A. Repeat the save using the same run ID and content  
B. Assume nothing was saved and use a new run ID  
C. Delete the database  
D. Automatically repeat all tool actions  

### 14. Why use temporary databases in tests?

A. To avoid testing persistence  
B. To isolate tests from each other and from demonstration data  
C. To remove the need for assertions  
D. To guarantee production performance  

### 15. What does writing a save-event row in the transaction prove?

A. Every external subscriber received the event  
B. The frontend displayed the answer  
C. The local event record committed alongside the answer  
D. All downstream actions completed  

## Part 2 — Programs

Continue in your existing lifecycle practice folder. Keep your main backend unchanged.

### Program 1 — Extend the persistence tests

Add these four tests to `test_storage.py`:

1. Failed runs cannot be saved.
2. Cancelled runs cannot be saved.
3. Two different completed runs create two answers and two save events.
4. A new `AnswerStore` instance reads the previously committed database records.

Starter:

```python
def test_failed_run_is_rejected(self):
    run = AgentRun.create()
    run.start()
    run.add_token("Partial answer")
    run.fail("Simulated generation failure")

    with self.assertRaises(ValueError):
        self.store.save(run)

    self.assertEqual(
        self.store.counts(),
        {"answers": 0, "save_events": 0},
    )
```

For the fourth test, reuse the database path:

```python
second_store = AnswerStore(self.database_path)
```

**Acceptance criterion:** all eight storage tests pass—the original four plus these four.

Run with your virtual environment activated:

```bash
python -m unittest -v
```

### Program 2 — Prove transaction rollback

**Goal:** verify that failure of the second write rolls back the first write.

Use only the temporary database created by your test.

SQLite can simulate this failure using a trigger:

```python
import sqlite3
```

Inside a new test, install this trigger:

```python
with sqlite3.connect(self.database_path) as connection:
    connection.execute("""
        CREATE TRIGGER reject_save_event
        BEFORE INSERT ON save_events
        BEGIN
            SELECT RAISE(ABORT, 'Simulated event write failure');
        END;
    """)
```

Then complete the test:

```python
with self.assertRaises(sqlite3.IntegrityError):
    self.store.save(completed_run("Test answer"))

self.assertEqual(
    self.store.counts(),
    {"answers": 0, "save_events": 0},
)
```

**Expected:** neither the answer nor its event remains committed.

Explain in your own words:

> Why would this test fail if the answer were committed before inserting the event?

### Program 3 — Simulate a lost save receipt

**Goal:** distinguish “confirmation was lost” from “nothing was saved.”

Create `lost_receipt_demo.py`.

Steps:

1. Create one completed run.
2. Save it successfully.
3. Deliberately ignore the returned receipt.
4. Repeat the save with the **same run object**.
5. Print the second receipt and database counts.

Use a temporary directory so each demonstration starts clean.

Starter:

```python
import tempfile
from pathlib import Path

from lifecycle import AgentRun
from storage import AnswerStore


def main():
    with tempfile.TemporaryDirectory() as directory:
        store = AnswerStore(Path(directory) / "demo.db")

        run = AgentRun.create()
        run.start()
        run.add_token("A completed answer")
        run.complete()

        store.save(run)  # Simulate losing this receipt.

        # Repeat the save and inspect the result.


if __name__ == "__main__":
    main()
```

Expected result:

```text
Repeated save created a new answer: False
Database totals: {'answers': 1, 'save_events': 1}
```

This simulates the application-level consequence of a lost receipt; it does not simulate a real network connection.

**Acceptance criterion:** the retry returns the existing message ID and does not duplicate either record.

## Part 3 — EOD Mini-Project: Persistence Reliability Demo

Create a small console demonstration covering:

| Scenario | Expected outcome |
|---|---|
| Completed run saved once | One answer and one event |
| Same run saved twice | Still one answer and one event |
| Same ID with different content | Conflict rejected |
| Partial run submitted | Save rejected |
| Save-event insertion fails | Entire new transaction rolled back |

Use separate temporary databases where necessary so one scenario does not interfere with another.

For each scenario, print:

```text
Scenario:
Expected:
Actual:
Passed:
```

Do not print “Passed” unconditionally—calculate it from the actual result.

## Part 4 — Corporate and Client Investigation

A client reports:

> “The app showed ‘Could not confirm save.’ I clicked Retry and now see two copies of the answer.”

Write a short investigation note:

1. Did the first database transaction commit?
2. Did the retry reuse the original run ID?
3. Was uniqueness enforced in the database?
4. Did the retry repeat only the save, or the whole agent workflow?
5. Which regression test would prevent recurrence?

A responsible client update would distinguish:

> “Saving failed” from “We could not confirm whether saving succeeded.”

## EOD Deliverables

- Eight basic storage tests.
- One transaction-rollback test.
- One lost-receipt demonstration.
- A five-scenario reliability report.
- A short investigation note.

**Remember:** transactions keep related writes together; idempotency prevents repeated requests from creating repeated effects within a defined scope.

Reply with your quiz answers as `1.B, 2.A, ...`. Share your code or terminal output separately for program review.
