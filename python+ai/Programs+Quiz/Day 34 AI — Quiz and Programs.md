# Day 34 AI — Quiz and Programs

**Topic:** structured decisions, validation, authorization, and semantic correctness.

**Time:** 1–3 hours.  
**Mac and Windows:** identical Python code. No API key required.

## Part 1 — MCQ Quiz

Choose one answer per question.

### 1. What does successfully parsing JSON establish?

A. The answer is factually correct  
B. The user may execute the action  
C. The text can be interpreted as JSON data  
D. All required application fields are present  

### 2. What is the purpose of our decision contract?

A. Define allowed fields, types, actions, and relationships  
B. Replace authentication  
C. Guarantee factual accuracy  
D. Prevent every runtime failure  

### 3. Which decision is valid under our Day 34 contract?

A. `{"action":"search_policy","message":"","query":null}`  
B. `{"action":"answer","message":"","query":null}`  
C. `{"action":"delete_database","message":"Done","query":null}`  
D. `{"action":"clarify","message":"Which policy?","query":null}`  

### 4. Why reject unexpected fields?

A. JSON objects cannot contain extra fields  
B. Unrecognized fields must not accidentally influence application behaviour  
C. Extra fields always contain malware  
D. It makes authorization unnecessary  

### 5. What does the `Action` enum provide?

A. An allowlist of supported action names  
B. Proof that the selected action is appropriate  
C. User authentication  
D. Encryption of tool arguments  

### 6. A search decision contains `"query": null`. What kind of problem is this?

A. Database persistence failure  
B. Authentication failure  
C. Cross-field validation failure  
D. Network timeout  

### 7. A structurally valid search proposal is made by a user without search permission. What should happen?

A. Execute it because validation passed  
B. Ask the model to authorize the user  
C. Treat the action name as permission  
D. Reject execution at the authorization boundary  

### 8. Where should real search permission come from?

A. A model-generated `"can_search": true` field  
B. Trusted application policy applied to the authenticated user  
C. A client-supplied checkbox alone  
D. The length of the query  

### 9. Why is `frozen=True` useful for the validated decision object?

A. It makes normal field reassignment unavailable after construction  
B. It creates a security sandbox  
C. It validates every future tool call  
D. It prevents the model from hallucinating  

### 10. A well-structured answer invents an allowance amount. Which check is missing?

A. JSON parsing  
B. Field-name validation  
C. Semantic or factual evaluation  
D. Enum construction  

### 11. The action is `"refuse"`, but the message reveals confidential data. Which statement is correct?

A. The response is safe because the label says refuse  
B. Valid JSON makes disclosure acceptable  
C. The problem is only formatting  
D. An action label does not establish that the content is safe  

### 12. Why does our search proposal require an empty message?

A. Search tools can never produce text  
B. Our contract separates a search request from a final user-facing message  
C. Python dictionaries cannot contain two non-empty strings  
D. It guarantees retrieval accuracy  

### 13. What should you investigate when a valid, permitted search call fails during execution?

A. The tool implementation, inputs, and runtime dependencies  
B. Only the model’s writing style  
C. The enum definition first, regardless of evidence  
D. The browser’s font size  

### 14. Why use `json.dumps()` when constructing test payloads?

A. It proves the answer is true  
B. It authorizes the request  
C. It converts Python data to JSON without manual escaping  
D. It runs the proposed tool  

### 15. Which sequence best describes a controlled execution boundary?

A. Execute → parse → authorize → validate  
B. Parse → validate → authorize → execute  
C. Authorize every model proposal automatically  
D. Parse → execute → inspect permissions afterward  

## Part 2 — Programs

Continue in your `day34_structured_decisions` folder. Keep your main backend unchanged.

### Program 1 — Complete the validation tests

Add these six tests to `test_decision.py`:

1. Unknown action is rejected.
2. Unexpected field is rejected.
3. A JSON array is rejected.
4. A whitespace-only answer is rejected.
5. A search query longer than 200 characters is rejected.
6. An `answer` containing a query is rejected.

Starter:

```python
import json


def test_unknown_action_is_rejected(self):
    raw = json.dumps({
        "action": "delete_database",
        "message": "Done",
        "query": None,
    })

    with self.assertRaises(DecisionError):
        parse_decision(raw)
```

**Acceptance criterion:** ten tests pass—the original four plus these six.

Run with your virtual environment activated:

```bash
python -m unittest -v
```

### Program 2 — Prove denied actions never reach the tool

**Goal:** verify behaviour, not merely an error message.

Extend the executor to accept an injected search function:

```python
def execute_raw_output(raw, *, can_search, search):
    ...
```

Requirements:

- Parse and validate first.
- Check search permission before calling `search`.
- Call `search(decision.query)` only for an authorized search proposal.
- Return message decisions without calling `search`.

Use a mock:

```python
from unittest.mock import Mock

search = Mock(return_value=["Synthetic policy passage"])
```

Write four tests:

| Scenario | Expected tool calls |
|---|---|
| Valid search, allowed | Once with the validated query |
| Valid search, denied | Never |
| Invalid search proposal | Never |
| Valid clarification | Never |

Example assertion:

```python
search.assert_called_once_with("meal allowance")
```

Or:

```python
search.assert_not_called()
```

**Acceptance criterion:** invalid and unauthorized proposals produce zero tool calls.

Update your demo calls to match the new executor signature. Keep imports free of automatically executed demonstrations.

### Program 3 — Detect duplicate JSON field names

**Goal:** close the limitation discussed in Day 34.

This JSON contains a duplicate field:

```json
{
  "action": "clarify",
  "action": "search_policy",
  "message": "",
  "query": "salary"
}
```

Our original parser did not explicitly reject duplicate names before constructing the dictionary.

Add a helper:

```python
def reject_duplicate_fields(pairs):
    result = {}

    for key, value in pairs:
        if key in result:
            raise DecisionError(f"Duplicate JSON field: {key}")

        result[key] = value

    return result
```

Connect it to the existing parser:

```python
data = json.loads(
    raw,
    parse_constant=reject_nonstandard_constant,
    object_pairs_hook=reject_duplicate_fields,
)
```

Retain the existing validation checks.

Add tests proving:

1. Duplicate `action` is rejected.
2. Duplicate `query` is rejected.
3. A valid object still passes.

**Acceptance criterion:** ambiguous duplicate fields cannot silently become a different decision.

## Part 3 — EOD Mini-Project: Decision Boundary Report

Create a console report covering:

| Case | Expected |
|---|---|
| Valid answer | Message returned |
| Valid clarification | Message returned |
| Valid search, allowed | Search called once |
| Valid search, denied | Permission error; no search |
| Missing query | Validation error; no search |
| Unknown action | Validation error; no search |
| Duplicate action field | Validation error; no search |
| Wrong fact in valid structure | Structural acceptance only |

For each case, report:

```text
Case:
Expected:
Observed:
Tool call count:
Passed:
```

Use actual function calls, exceptions, and assertions to determine results.

For the factually wrong answer, explicitly print:

```text
Structure accepted.
Factual correctness was not established.
```

Do not label it a verified correct answer.

## Part 4 — Corporate and Client Investigation

A client reports:

> “The AI returned valid JSON and requested a policy search, but it retrieved documents the employee should not access.”

Write a short investigation note:

1. Was the failure parsing, validation, authorization, or retrieval filtering?
2. Did permission come from trusted application policy?
3. Did retrieval restrict documents to the authenticated user’s access?
4. Why was allowing the `search_policy` action insufficient?
5. Which tests would prevent recurrence?

Remember that **permission to search is not permission to access every document**.

## EOD Deliverables

- Ten basic validation tests.
- Four tool-execution boundary tests.
- Three duplicate-field tests.
- A decision-boundary report.
- A short access-control investigation note.

**Remember: valid structure is not permission, and permission is not proof of correctness.**

Reply with your quiz answers as `1.C, 2.A, ...`. Share your code or terminal output separately for practical review.
