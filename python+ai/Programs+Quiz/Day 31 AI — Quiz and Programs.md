# Day 31 AI — Quiz and Programs

**Topic:** agent-run states, streaming completion, failure, and cancellation.

**Time:** 1–3 hours.  
**Mac and Windows:** same Python code. No API key required.

## Part 1 — MCQ Quiz

Choose one answer per question.

### 1. The frontend receives the first text fragment. What does this prove?

A. The complete answer is saved  
B. Some output has arrived  
C. All tool actions succeeded  
D. The run has completed  

### 2. Which transition is valid in our Day 31 controller?

A. `COMPLETED → STREAMING`  
B. `FAILED → COMPLETED`  
C. `PENDING → STREAMING`  
D. `CANCELLED → PENDING`  

### 3. What is a terminal state?

A. A state after which this run accepts no further transitions  
B. A state used only in the VS Code terminal  
C. A state that automatically retries  
D. A state containing partial text  

### 4. Why separate `partial_text` from `final_answer`?

A. Python cannot store both in one class  
B. Partial text always contains errors  
C. Final answers do not need validation  
D. Received text and successfully completed output represent different facts  

### 5. The stream closes without a completion event. What should our controller do?

A. Mark the run completed  
B. Mark it failed  
C. Delete all evidence  
D. Start another run automatically  

### 6. What does `_require_streaming()` provide?

A. A security barrier that prevents external access  
B. Provider authentication  
C. A reusable check enforcing the required state  
D. Automatic database persistence  

### 7. A leading underscore in `_require_streaming` primarily means:

A. The method is intended for internal use by convention  
B. Other Python code cannot call it  
C. Its output is encrypted  
D. It can run only once  

### 8. What happens if `add_token()` is called after completion in our controller?

A. A new run starts  
B. The final answer is overwritten  
C. The token is silently accepted  
D. A `ValueError` is raised  

### 9. Generation completed, but saving the answer fails. Which statement is correct?

A. Completion proves the database save succeeded  
B. Generation and persistence are separate outcomes  
C. The browser must already have saved the answer  
D. The answer should be reported as saved anyway  

### 10. The browser disconnects. What can we conclude?

A. Every backend operation stopped immediately  
B. Every tool action was undone  
C. We must investigate; backend or provider work may still continue  
D. The request definitely completed  

### 11. Why does our text-only controller reject an empty completion?

A. Its specific contract requires a non-empty text answer  
B. All AI workflows require text output  
C. Python enums reject empty strings automatically  
D. Tool-only runs cannot exist  

### 12. What does `finally` do in the simulator?

A. Converts every exception into success  
B. Prevents exceptions from reaching the caller  
C. Runs only when the simulation succeeds  
D. Calls cleanup whether processing succeeds or raises an exception  

### 13. What should happen when retrying a failed run in this design?

A. Reopen the failed run and append tokens  
B. Create a new run with its own identity  
C. Change its status directly to completed  
D. Reuse another user’s run  

### 14. What does clicking “Stop” establish by itself?

A. All external actions have been reversed  
B. The final answer was saved  
C. The user requested cancellation; actual cancellation still needs implementation and confirmation  
D. The provider will charge nothing  

### 15. Which client-facing message is most responsible after a partial stream fails?

A. “The response was interrupted before completion.”  
B. “Everything completed successfully.”  
C. “Your action was definitely undone.”  
D. “The answer was saved,” without checking persistence  

## Part 2 — Programs

Use your existing `day31_run_lifecycle` practice folder. Do not replace your main backend.

### Program 1 — Complete the state-boundary tests

Add these tests to `test_lifecycle.py`:

1. A token before `start()` raises `ValueError`.
2. An explicit error produces `FAILED`.
3. Empty text cannot complete the text-only run.
4. Two newly created runs have different IDs.
5. Completion after cancellation is rejected.
6. Calling `finish_stream()` after successful completion leaves the state unchanged.

Starter:

```python
def test_token_before_start_is_rejected(self):
    run = AgentRun.create()

    with self.assertRaises(ValueError):
        run.add_token("Too early")
```

Run with your virtual environment activated:

```bash
python -m unittest -v
```

**Acceptance criterion:** the original six tests plus these six tests pass.

Remember: a test expecting rejection passes when the controller correctly raises the exception.

### Program 2 — Add a user-facing status formatter

Create `presentation.py`.

Implement:

```python
def get_status_message(run):
    ...
```

Required mappings:

| Status | Message |
|---|---|
| `PENDING` | Waiting to start |
| `STREAMING` | Generating response |
| `COMPLETED` | Response generation completed |
| `FAILED` | Response failed or was interrupted |
| `CANCELLED` | Response generation cancelled |

Requirements:

- Use the enum rather than unrelated string values.
- Do not display `error_message` directly to the user.
- Do not say “saved successfully”—this controller does not track database persistence.
- Test all five mappings.

**Acceptance criterion:** each state produces the correct message without exposing diagnostic details.

### Program 3 — Build a run-summary function

Create `reporting.py`.

Implement:

```python
def summarize_runs(runs):
    ...
```

Example output:

```python
{
    "total": 4,
    "completed": 1,
    "failed": 1,
    "cancelled": 1,
    "pending": 0,
    "streaming": 1,
    "completion_rate": 0.25,
}
```

Requirements:

- Count every state.
- Define `completion_rate` as `completed / total`.
- For an empty list, return a completion rate of `0.0`.
- Do not count partial text as completion.
- Do not label this metric “answer accuracy.”

Test:

```text
✓ Empty input
✓ All completed
✓ Mixed states
✓ Failed run containing partial text
```

**Acceptance criterion:** counts add up to `total`, and the completion rate matches your defined denominator.

Corporate consideration: including in-progress runs affects this rate. Document your denominator rather than leaving its meaning ambiguous.

## Part 3 — EOD Mini-Project: Run Lifecycle Monitor

Combine the simulator, formatter, and summary function.

Simulate four runs:

```text
Run 1: start → token → complete
Run 2: start → token → error
Run 3: start → token → cancel
Run 4: start → token → unexpected stream end
```

Print a report similar to:

```text
Run 1: Response generation completed
Run 2: Response failed or was interrupted
Run 3: Response generation cancelled
Run 4: Response failed or was interrupted

Total: 4
Completed: 1
Failed: 2
Cancelled: 1
Completion rate: 25.0%
```

Also show whether each run can publish a final answer.

**Expected:** only Run 1 can.

## Part 4 — Real-world investigation

A client reports:

> “The answer appeared completely on screen, but after refreshing, it disappeared.”

Write a short investigation note answering:

1. Does visible text prove persistence?
2. What evidence would you inspect in the browser and backend?
3. What if generation completed but the database transaction failed?
4. When should the frontend display “Saved”?
5. How would you reproduce this with a simulated save failure?

Keep the distinction clear:

> **Generated ≠ saved ≠ delivered. Cancellation requested ≠ cancellation confirmed.**

Reply with your quiz answers as `1.B, 2.C, ...`. Share program output or code separately for review.
