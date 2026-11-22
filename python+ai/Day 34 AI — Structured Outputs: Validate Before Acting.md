# Day 34 AI — Structured Outputs: Validate Before Acting

**Today’s project:** build a boundary that converts raw AI output into a validated decision your application can safely interpret.

Days 31–33 taught us to manage runs, save results, and prevent duplicate execution.

Now we address another problem:

> What if the agent completes successfully—but returns malformed or inappropriate instructions?

**Time:** 1–3 hours.  
**Requirements:** Python standard library only. No API key.  
**Mac and Windows:** identical code.

## 1. The mental model

Do not connect raw model text directly to application actions.

```text
Raw model output
       ↓
Parse JSON
       ↓
Validate fields, types, and relationships
       ↓
Check permissions and application policy
       ↓
Execute an allowed action
```

These checks answer different questions:

- **Parsing:** Is this valid JSON?
- **Validation:** Does it follow our contract?
- **Authorization:** Is this user allowed to perform the operation?
- **Evaluation:** Was the decision appropriate and the answer correct?

**Valid JSON does not establish correctness or authorization.**

## 2. Define a small decision contract

Our practice agent can propose four actions:

| Action | Meaning | Required content |
|---|---|---|
| `answer` | Provide an answer | Non-empty message |
| `clarify` | Ask for missing information | Non-empty message |
| `refuse` | Decline the request | Non-empty message |
| `search_policy` | Request a policy search | Non-empty query |

Every result must contain exactly these fields:

```json
{
  "action": "clarify",
  "message": "Which organization's policy should I use?",
  "query": null
}
```

A search proposal looks like:

```json
{
  "action": "search_policy",
  "message": "",
  "query": "meal allowance"
}
```

We deliberately keep search results separate from final answers. A request to search is not itself an answer.

## 3. Create a separate practice folder

Create `day34_structured_decisions` with:

```text
decision.py
demo.py
test_decision.py
```

Keep your earlier projects unchanged.

## 4. Implement the parser and validator

Put this in `decision.py`:

```python
import json
from dataclasses import dataclass
from enum import Enum


class DecisionError(ValueError):
    pass


class Action(str, Enum):
    ANSWER = "answer"
    CLARIFY = "clarify"
    REFUSE = "refuse"
    SEARCH_POLICY = "search_policy"


@dataclass(frozen=True)
class Decision:
    action: Action
    message: str
    query: str | None


def reject_nonstandard_constant(value):
    raise DecisionError(f"Non-standard JSON constant: {value}")


def parse_decision(raw: str) -> Decision:
    if not isinstance(raw, str):
        raise DecisionError("Output must be a string")

    if len(raw) > 10_000:
        raise DecisionError("Output exceeds the lab's size limit")

    try:
        data = json.loads(
            raw,
            parse_constant=reject_nonstandard_constant,
        )
    except json.JSONDecodeError as exc:
        raise DecisionError("Output is not valid JSON") from exc

    if not isinstance(data, dict):
        raise DecisionError("Output must be a JSON object")

    expected_fields = {"action", "message", "query"}

    if set(data) != expected_fields:
        raise DecisionError(
            "Output must contain exactly: action, message, query"
        )

    if not isinstance(data["action"], str):
        raise DecisionError("Action must be a string")

    try:
        action = Action(data["action"])
    except ValueError as exc:
        raise DecisionError("Unknown action") from exc

    if not isinstance(data["message"], str):
        raise DecisionError("Message must be a string")

    message = data["message"].strip()

    if len(message) > 1_000:
        raise DecisionError("Message exceeds 1,000 characters")

    query = data["query"]

    if query is not None and not isinstance(query, str):
        raise DecisionError("Query must be a string or null")

    if isinstance(query, str):
        query = query.strip()

    if action == Action.SEARCH_POLICY:
        if not query:
            raise DecisionError("Search requires a non-empty query")

        if len(query) > 200:
            raise DecisionError("Query exceeds 200 characters")

        if message:
            raise DecisionError(
                "A search proposal must not include a final message"
            )

    else:
        if not message:
            raise DecisionError(
                "Answer, clarify, and refuse require a message"
            )

        if query is not None:
            raise DecisionError(
                "Only search_policy may include a query"
            )

    return Decision(
        action=action,
        message=message,
        query=query,
    )
```

### Understand what this code does

**`json.loads()`**

Converts JSON text into Python values. It does not enforce our decision contract.

**Exact fields**

We reject missing or unexpected fields. An extra field such as `"skip_authorization": true` must not gain meaning accidentally.

**`Action` enum**

Defines the complete action allowlist. An output cannot introduce an action such as `delete_database`.

**Cross-field validation**

Checking individual types is insufficient.

For example:

```json
{
  "action": "search_policy",
  "message": "",
  "query": null
}
```

The types are plausible, but the combination is invalid: a search needs a query.

**Frozen dataclass**

Produces an object whose fields cannot normally be reassigned after validation.

This is a convenience—not a security sandbox. Keep execution behind a controlled application boundary.

## 5. Demonstrate controlled execution

Put this in `demo.py`:

```python
from decision import Action, DecisionError, parse_decision


def execute_raw_output(raw, *, can_search):
    decision = parse_decision(raw)

    if decision.action == Action.SEARCH_POLICY:
        if not can_search:
            raise PermissionError("Policy search is not permitted")

        # Local demonstration only: no real retrieval happens here.
        return {
            "kind": "search_requested",
            "query": decision.query,
        }

    return {
        "kind": "message",
        "action": decision.action.value,
        "text": decision.message,
    }


if __name__ == "__main__":
    clarify = """
    {
        "action": "clarify",
        "message": "Which policy should I use?",
        "query": null
    }
    """

    search = """
    {
        "action": "search_policy",
        "message": "",
        "query": "meal allowance"
    }
    """

    malformed = """
    {
        "action": "search_policy",
        "message": "",
        "query": null
    }
    """

    print(execute_raw_output(clarify, can_search=False))
    print(execute_raw_output(search, can_search=True))

    try:
        execute_raw_output(search, can_search=False)
    except PermissionError:
        print("Search blocked by application permissions")

    try:
        execute_raw_output(malformed, can_search=True)
    except DecisionError as exc:
        print("Invalid decision rejected:", exc)
```

Run with your virtual environment activated:

```bash
python demo.py
```

On Mac, outside the environment, use `python3` if needed.

Expected key results:

```text
... 'action': 'clarify' ...
... 'kind': 'search_requested' ...
Search blocked by application permissions
Invalid decision rejected: Search requires a non-empty query
```

### Notice the separation

The search proposal can be:

```text
Valid structure + insufficient permission → blocked
```

In a real API, `can_search` must come from trusted application policy—not the model or a client-supplied boolean. Actual retrieval must also restrict which documents the user may access.

## 6. Write boundary tests

Put this in `test_decision.py`:

```python
import unittest

from decision import Action, DecisionError, parse_decision
from demo import execute_raw_output


class DecisionTests(unittest.TestCase):
    def test_valid_answer(self):
        raw = """
        {
            "action": "answer",
            "message": "Hello",
            "query": null
        }
        """

        decision = parse_decision(raw)

        self.assertEqual(decision.action, Action.ANSWER)
        self.assertEqual(decision.message, "Hello")

    def test_invalid_json_is_rejected(self):
        with self.assertRaises(DecisionError):
            parse_decision("This is not JSON")

    def test_search_requires_query(self):
        raw = """
        {
            "action": "search_policy",
            "message": "",
            "query": null
        }
        """

        with self.assertRaises(DecisionError):
            parse_decision(raw)

    def test_valid_search_still_requires_permission(self):
        raw = """
        {
            "action": "search_policy",
            "message": "",
            "query": "meal allowance"
        }
        """

        with self.assertRaises(PermissionError):
            execute_raw_output(raw, can_search=False)


if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

**Expected:** four passing tests.

## 7. What this does—and does not—prove

Consider:

```json
{
  "action": "answer",
  "message": "Every employee receives INR 50,000 per meal.",
  "query": null
}
```

Our validator accepts its structure.

It cannot establish whether the allowance is real.

Likewise:

```json
{
  "action": "refuse",
  "message": "Here is the confidential information...",
  "query": null
}
```

The action label does not make the content safe.

This connects directly to Day 30:

> Structural checks are useful, but semantic behaviour needs separate evaluation.

Also, this validator detects duplicate-free field sets after parsing, but does not reject duplicate JSON member names before parsing. Treat duplicate-key detection as an optional hardening exercise—not a capability already implemented.

## 8. Your assignment

Add six tests:

1. Unknown action is rejected.
2. Unexpected field is rejected.
3. A JSON array is rejected.
4. An answer containing only spaces is rejected.
5. A search query longer than 200 characters is rejected.
6. An `answer` containing a query is rejected.

Use `json.dumps()` to construct test inputs without manually escaping JSON.

Example:

```python
import json

raw = json.dumps({
    "action": "search_policy",
    "message": "",
    "query": "x" * 201,
})
```

**Expected total:** ten passing tests.

### EOD mini-project

Create a decision-validation report for these scenarios:

| Scenario | Expected result |
|---|---|
| Valid clarification | Accepted as a message |
| Valid search, permission granted | Search request produced |
| Valid search, permission denied | Permission error |
| Missing search query | Validation error |
| Unknown action | Validation error |
| Factually wrong but well-structured answer | Structure accepted; requires semantic evaluation |

Calculate outcomes using actual function calls and assertions—not hardcoded “PASS” messages.

## 9. Real-world troubleshooting approach

When an AI response causes trouble, classify the failure first:

```text
Cannot parse it?
    → JSON/transport problem

Can parse it, but fields are invalid?
    → Contract problem

Valid proposal, but user lacks permission?
    → Authorization problem

Valid and permitted, but wrong decision or answer?
    → Behaviour/quality problem

Valid tool request, but execution fails?
    → Tool/runtime problem
```

Do not solve every category by rewriting the prompt.

### Institute, corporate, and client mindset

- **Institute:** explain why valid JSON can still be an invalid decision.
- **Corporate:** version contracts and test every allowed action and rejected boundary.
- **Client:** document supported actions, permission rules, and what validation does not guarantee.

## EOD checklist

- [ ] Raw text is parsed and validated before execution.
- [ ] Unknown actions and unexpected fields are rejected.
- [ ] Cross-field rules are enforced.
- [ ] Permissions are checked separately.
- [ ] Ten tests pass.
- [ ] You can explain why a validated answer can still be wrong.

**Remember: parsing gives you data; validation gives you a contract; authorization gives permission. None of them alone establishes truth.**
