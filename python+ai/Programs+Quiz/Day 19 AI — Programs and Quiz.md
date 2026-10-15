## Day 19 AI — Programs and Quiz

Guardrails automatically allow, deny, or validate. Approvals pause sensitive side effects until a person explicitly confirms them. [Official OpenAI guardrails and approvals guide](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals)

### Program 1: Approval-token store

Create `approval_store.py`:

```python
import secrets
from dataclasses import dataclass
from datetime import datetime, timedelta, timezone


@dataclass(frozen=True)
class PendingApproval:
    token: str
    user_id: str
    tool_name: str
    expires_at: datetime


approvals: dict[str, PendingApproval] = {}


def create_approval(
    *,
    user_id: str,
    tool_name: str,
    expires_in_minutes: int = 10,
) -> PendingApproval:
    token = secrets.token_urlsafe(16)

    approval = PendingApproval(
        token=token,
        user_id=user_id,
        tool_name=tool_name,
        expires_at=datetime.now(timezone.utc)
        + timedelta(minutes=expires_in_minutes),
    )

    approvals[token] = approval
    return approval


def validate_approval(
    *,
    token: str,
    user_id: str,
    tool_name: str,
) -> bool:
    approval = approvals.get(token)

    if approval is None:
        return False

    if approval.user_id != user_id:
        return False

    if approval.tool_name != tool_name:
        return False

    if datetime.now(timezone.utc) > approval.expires_at:
        del approvals[token]
        return False

    del approvals[token]  # One-time-use token
    return True
```

### Program 2: Confirmation demo

Create `approval_demo.py`:

```python
from approval_store import create_approval, validate_approval

user_id = "user-101"
tool_name = "delete_expense"

approval = create_approval(
    user_id=user_id,
    tool_name=tool_name,
)

print("Sensitive action needs approval.")
print("Confirmation token:", approval.token)

entered_token = input("Enter token to confirm: ").strip()

is_approved = validate_approval(
    token=entered_token,
    user_id=user_id,
    tool_name=tool_name,
)

if is_approved:
    print("Approved: deletion may now continue.")
else:
    print("Denied: token is invalid, expired, or does not match.")
```

Run:

```bash
python approval_demo.py
```

### Program 3: Redact secrets before logging

Add this helper to `audit_log.py`:

```python
from typing import Any


SENSITIVE_FIELDS = {
    "password",
    "api_key",
    "token",
    "authorization",
}


def redact_sensitive_fields(arguments: dict[str, Any]) -> dict[str, Any]:
    safe_arguments = {}

    for key, value in arguments.items():
        if key.lower() in SENSITIVE_FIELDS:
            safe_arguments[key] = "[REDACTED]"
        else:
            safe_arguments[key] = value

    return safe_arguments
```

Then update the audit event:

```python
"arguments": redact_sensitive_fields(arguments),
```

Never log passwords, API keys, access tokens, or bank-account details.

### Program 4: Test approval tokens

Create `tests/test_approval_store.py`:

```python
from approval_store import create_approval, validate_approval


def test_valid_token_is_approved_once():
    approval = create_approval(
        user_id="user-101",
        tool_name="delete_expense",
    )

    assert validate_approval(
        token=approval.token,
        user_id="user-101",
        tool_name="delete_expense",
    ) is True

    assert validate_approval(
        token=approval.token,
        user_id="user-101",
        tool_name="delete_expense",
    ) is False


def test_token_cannot_be_used_by_another_user():
    approval = create_approval(
        user_id="user-101",
        tool_name="delete_expense",
    )

    assert validate_approval(
        token=approval.token,
        user_id="user-999",
        tool_name="delete_expense",
    ) is False


def test_wrong_tool_cannot_use_token():
    approval = create_approval(
        user_id="user-101",
        tool_name="delete_expense",
    )

    assert validate_approval(
        token=approval.token,
        user_id="user-101",
        tool_name="transfer_money",
    ) is False
```

Run:

```bash
pytest
```

---

## Day 19 AI Quiz

Reply like: `1.B, 2.A, ...`

1. What is a guardrail?

   A. A database backup  
   B. An automatic validation or safety check  
   C. An API key  
   D. A frontend component  

2. What is human approval used for?

   A. Pausing a sensitive action until it is approved or rejected  
   B. Formatting Python code  
   C. Creating embeddings  
   D. Installing packages  

3. Which is normally a read-only action?

   A. Transfer money  
   B. Delete an expense  
   C. Calculate remaining budget  
   D. Update a category  

4. Which action should usually require approval?

   A. Search local documents  
   B. Categorize “Uber ride”  
   C. Delete an expense  
   D. Calculate savings rate  

5. What should happen to an unknown tool request?

   A. Execute it  
   B. Deny it safely  
   C. Give it administrator access  
   D. Delete audit logs  

6. Why should approval tokens expire?

   A. To make them safer if leaked or unused  
   B. To make tests fail  
   C. To avoid logging  
   D. To increase API cost  

7. Why should a confirmation token be one-time use?

   A. To prevent replaying a previous approval  
   B. To make it easier to guess  
   C. To avoid validation  
   D. To remove user identity  

8. Which data should never be written into logs?

   A. Tool name  
   B. Timestamp  
   C. API key  
   D. Decision status  

9. What should a tool guardrail validate?

   A. Tool arguments and tool results around execution  
   B. Only page color  
   C. Only Python version  
   D. Only the final greeting  

10. What does “fail closed” mean?

   A. If safety validation fails, deny or pause the action  
   B. Run every tool anyway  
   C. Delete the project  
   D. Ignore errors  

11. Why keep audit logs?

   A. To understand, debug, and review agent decisions  
   B. To store every password  
   C. To replace tests  
   D. To bypass approvals  

12. Which is the correct high-risk workflow?

   A. Execute → ask later  
   B. Plan → guardrail → approval → execute → log  
   C. Delete → log  
   D. LLM decides everything alone  

13. If a user approval token belongs to another user, what should happen?

   A. Allow it  
   B. Deny it  
   C. Change its owner automatically  
   D. Ignore the user ID  

14. What is the difference between a guardrail and approval?

   A. They are exactly the same  
   B. Guardrail is automatic; approval requires a person or policy decision  
   C. Guardrails require passwords  
   D. Approvals never pause execution  

15. What is the best production rule for sensitive actions?

   A. Require explicit approval, validate inputs, log safely, and test  
   B. Trust every model tool call  
   C. Skip logging  
   D. Store all secrets in JSON
