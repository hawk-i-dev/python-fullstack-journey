# Day 26 AI — Reliable Agent APIs: Timeouts, Retries, and Rate Limits

Today you learn how to handle real failures without crashing the app or repeatedly charging/calling an AI provider.

A production agent must distinguish:

```text
Temporary failure → retry carefully
Permanent failure → return a clear safe error
Stream already started → do not blindly replay it
```

For temporary `429` or overload errors, follow `Retry-After` when provided; otherwise use bounded exponential backoff with jitter. Do not retry billing, quota, authentication, or invalid-input errors. [Official OpenAI rate-limit guidance](https://developers.openai.com/api/docs/guides/rate-limits)

## What should retry?

| Failure | Retry? | Reason |
|---|---:|---|
| Network connection error | Yes | Usually temporary |
| Request timeout | Yes, limited | May be temporary |
| `429` temporary rate limit | Yes | Respect `Retry-After` |
| `503` service overload | Yes | Provider may recover |
| `400` invalid request | No | Fix code/input |
| `401` / `403` | No | Fix credentials/permissions |
| Billing or quota error | No | Needs account action |
| Stream already produced text | No automatic replay | Could duplicate output/tool actions |

Important: after an SSE stream starts, an error can arrive midway. Do not silently restart the whole request because the agent might repeat actions or duplicate text.

## 1. Create `app/resilience.py`

```python
import random


def retry_delay(
    attempt_number: int,
    retry_after: float | None = None,
    jitter: float | None = None,
) -> float:
    if retry_after is not None:
        return retry_after

    base_delay = 0.5 * (2 ** attempt_number)
    random_jitter = jitter if jitter is not None else random.uniform(0, 0.25)

    return base_delay + random_jitter
```

Example delays:

```text
Attempt 0 → around 0.5 seconds
Attempt 1 → around 1 second
Attempt 2 → around 2 seconds
```

The random jitter prevents many clients from retrying again at exactly the same moment.

## 2. Add a retry-safe stream starter

Create `app/llm_client.py`:

```python
import asyncio
import os

from openai import (
    APIConnectionError,
    APITimeoutError,
    AsyncOpenAI,
    InternalServerError,
    RateLimitError,
)

from app.resilience import retry_delay


class AgentTemporarilyUnavailable(Exception):
    pass


RETRYABLE_ERRORS = (
    APIConnectionError,
    APITimeoutError,
    InternalServerError,
    RateLimitError,
)


def get_retry_after(error: Exception) -> float | None:
    response = getattr(error, "response", None)

    if response is None:
        return None

    raw_value = response.headers.get("retry-after")

    try:
        return float(raw_value)
    except (TypeError, ValueError):
        return None


async def start_response_stream(
    input_messages: list[dict[str, str]],
):
    client = AsyncOpenAI(
        max_retries=0,
        timeout=20.0,
    )

    max_attempts = 3

    for attempt in range(max_attempts):
        try:
            return await client.responses.create(
                model=os.environ["OPENAI_MODEL"],
                input=input_messages,
                stream=True,
            )

        except RETRYABLE_ERRORS as error:
            is_last_attempt = attempt == max_attempts - 1

            if is_last_attempt:
                raise AgentTemporarilyUnavailable(
                    "AI service is temporarily unavailable"
                ) from error

            delay = retry_delay(
                attempt_number=attempt,
                retry_after=get_retry_after(error),
            )

            await asyncio.sleep(delay)
```

Why `max_retries=0`?

Your code is managing retries itself. If the SDK and your code both retry, you can accidentally create nested retries and send too many requests.

## 3. Stream only after a successful start

Add this function in `app/llm_client.py`:

```python
from collections.abc import AsyncGenerator


async def openai_agent_stream(
    input_messages: list[dict[str, str]],
) -> AsyncGenerator[str, None]:
    stream = await start_response_stream(input_messages)

    async for event in stream:
        if event.type == "response.output_text.delta":
            yield event.delta
```

Important rule:

```text
Retry only while starting the request.
After text begins streaming, report errors safely instead of replaying.
```

## 4. Return a safe frontend error event

Inside your `generate_events()` function in `app/main.py`:

```python
from app.llm_client import AgentTemporarilyUnavailable
```

Then add:

```python
except AgentTemporarilyUnavailable:
    yield sse_event(
        "error",
        {
            "message": (
                "The AI service is busy. "
                "Please try again in a moment."
            ),
        },
    )
```

The browser receives a useful message. Your server logs should contain the detailed exception and request ID.

## 5. Test retry-delay logic

Create `tests/test_resilience.py`:

```python
from app.resilience import retry_delay


def test_retry_after_has_priority():
    delay = retry_delay(
        attempt_number=2,
        retry_after=4.0,
        jitter=0,
    )

    assert delay == 4.0


def test_exponential_backoff_increases_delay():
    first_delay = retry_delay(
        attempt_number=0,
        jitter=0,
    )

    second_delay = retry_delay(
        attempt_number=1,
        jitter=0,
    )

    third_delay = retry_delay(
        attempt_number=2,
        jitter=0,
    )

    assert first_delay == 0.5
    assert second_delay == 1.0
    assert third_delay == 2.0
```

Run:

```bash
pytest -q
```

## Day 26 Assignment

Complete these before Day 27:

1. Add `resilience.py` and its tests.
2. Replace the local demo stream with your retry-safe OpenAI stream adapter.
3. Verify invalid API credentials do not retry repeatedly.
4. Temporarily set an invalid model name and confirm the frontend shows a safe error.
5. Add this section to your README:

```text
Reliability policy:
- Retry temporary connection, timeout, rate-limit, and overload failures.
- Use a maximum of three attempts.
- Respect Retry-After when available.
- Never retry invalid requests, authentication failures, or billing/quota errors.
- Never automatically replay a stream after output begins.
```

This is how you move from “the API works on my laptop” to “the API behaves safely when real systems fail.”
