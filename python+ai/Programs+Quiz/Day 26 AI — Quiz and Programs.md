# Day 26 AI — Quiz and Programs

Retry only temporary failures, keep retries bounded, and never replay a stream after visible output has started. A `Retry-After` value is a minimum wait time; otherwise use exponential backoff with jitter. [Official OpenAI rate-limit guidance](https://developers.openai.com/api/docs/guides/rate-limits)

## Program 1 — Classify retryable errors

Add this to `app/resilience.py`:

```python
NON_RETRYABLE_CODES = {
    "invalid_api_key",
    "insufficient_quota",
    "billing_hard_limit_reached",
    "project_spend_limit_exceeded",
}


def should_retry(
    status_code: int | None,
    error_code: str | None = None,
) -> bool:
    if error_code in NON_RETRYABLE_CODES:
        return False

    return status_code in {
        408,
        429,
        500,
        502,
        503,
        504,
    }
```

Create `tests/test_retry_rules.py`:

```python
from app.resilience import should_retry


def test_rate_limit_is_retryable():
    assert should_retry(429, "slow_down") is True


def test_server_overload_is_retryable():
    assert should_retry(503, "server_is_overloaded") is True


def test_invalid_api_key_is_not_retryable():
    assert should_retry(401, "invalid_api_key") is False


def test_billing_limit_is_not_retryable():
    assert should_retry(429, "billing_hard_limit_reached") is False


def test_invalid_request_is_not_retryable():
    assert should_retry(400, "invalid_request_error") is False
```

## Program 2 — Test a retrying async operation

Add this to `app/resilience.py`:

```python
import asyncio
from collections.abc import Awaitable, Callable
from typing import TypeVar


T = TypeVar("T")


async def retry_async(
    operation: Callable[[], Awaitable[T]],
    max_attempts: int = 3,
) -> T:
    last_error: Exception | None = None

    for attempt in range(max_attempts):
        try:
            return await operation()

        except ConnectionError as error:
            last_error = error

            if attempt == max_attempts - 1:
                break

            await asyncio.sleep(
                retry_delay(attempt_number=attempt)
            )

    assert last_error is not None
    raise last_error
```

Create `tests/test_retry_async.py`:

```python
import asyncio

from app.resilience import retry_async


def test_retry_async_succeeds_after_temporary_failures():
    attempts = 0

    async def unreliable_operation() -> str:
        nonlocal attempts

        attempts += 1

        if attempts < 3:
            raise ConnectionError("Temporary network failure")

        return "success"

    result = asyncio.run(
        retry_async(
            unreliable_operation,
            max_attempts=3,
        )
    )

    assert result == "success"
    assert attempts == 3
```

## Program 3 — Add a frontend “Try again” option

In `App.tsx`, store the latest question:

```tsx
const [lastQuestion, setLastQuestion] = useState("");
```

Before calling `streamAssistant()`:

```tsx
setLastQuestion(question);
```

Add a helper:

```tsx
function tryAgain() {
  if (!lastQuestion || streaming) {
    return;
  }

  setDraft(lastQuestion);
  setError("");
}
```

Show a retry button after an error:

```tsx
{error && (
  <section role="alert">
    <p>{error}</p>

    <button
      type="button"
      onClick={tryAgain}
      disabled={streaming}
    >
      Try again
    </button>
  </section>
)}
```

The user should explicitly click **Try again**. Do not automatically retry after partial text was displayed.

# Day 26 Quiz

1. Which failure is normally safe to retry?  
   A. Invalid API key  
   B. Temporary network failure  
   C. Invalid request body  
   D. Billing hard limit  

2. What does HTTP `429` commonly indicate?  
   A. Successful response  
   B. Rate limit or throttling  
   C. Missing API route  
   D. Invalid JSON  

3. What should you do when `Retry-After` is present?  
   A. Retry immediately  
   B. Wait at least the stated amount  
   C. Ignore the response  
   D. Delete the conversation  

4. What is exponential backoff?  
   A. Retrying with an increasing delay  
   B. Retrying forever instantly  
   C. Deleting failed requests  
   D. Increasing database size  

5. Why add random jitter to retry delays?  
   A. To make errors harder to debug  
   B. To prevent many clients retrying at the same instant  
   C. To bypass rate limits  
   D. To expose hidden errors  

6. Which error should not be retried automatically?  
   A. `503` temporary overload  
   B. Connection timeout  
   C. Invalid credentials  
   D. Temporary rate limit  

7. Why set a maximum number of retry attempts?  
   A. To avoid infinite waiting and extra requests  
   B. To remove authentication  
   C. To make models faster  
   D. To avoid tests  

8. Why can nested SDK and application retries be dangerous?  
   A. They can multiply requests unexpectedly  
   B. They remove all errors  
   C. They disable SQLite  
   D. They block React rendering permanently  

9. What should happen after a stream has already shown text and then fails?  
   A. Automatically replay the whole request  
   B. Return a safe error and avoid blind replay  
   C. Return the API key  
   D. Delete all chat history  

10. Which status can indicate temporary provider overload?  
   A. 200  
   B. 401  
   C. 503  
   D. 422  

11. What should a user see when the AI provider is temporarily unavailable?  
   A. Internal stack trace  
   B. Safe message asking them to try later  
   C. Database connection string  
   D. Another user’s data  

12. What does a timeout mean?  
   A. An operation took longer than allowed  
   B. User authentication succeeded  
   C. Database data is deleted  
   D. Streaming completed successfully  

13. Which code should be logged server-side for support investigation?  
   A. Request ID and safe error details  
   B. API key  
   C. User password  
   D. Full `.env` content  

14. What should happen for a billing or quota failure?  
   A. Retry rapidly  
   B. Return a clear error; account action is required  
   C. Ignore it  
   D. Delete the project  

15. What is the safe resilience flow?  
   A. Retry every failure forever  
   B. Classify error → bounded retry only temporary failures → safe client message → log details  
   C. Show all internal errors to users  
   D. Replay every failed stream automatically  

Reply like this:

```text
1.B
2.B
3.B
...
15.B
```
