# Day 27 AI — Quiz and Programs

Your product quota is different from the provider’s rate limit:

```text
Provider limits → organization/project
App quota       → each user in your application
```

A client-facing AI app should enforce its own user limits to control misuse and cost. [Official OpenAI rate-limit guidance](https://developers.openai.com/api/docs/guides/rate-limits)

## Program 1 — Test user quota isolation

Add this test to `tests/test_usage.py`:

```python
from app.usage import (
    consume_daily_request_quota,
    get_daily_request_count,
)


def test_each_user_has_independent_quota(db):
    consume_daily_request_quota(
        db=db,
        user_id="alice",
        limit=2,
    )
    consume_daily_request_quota(
        db=db,
        user_id="alice",
        limit=2,
    )

    consume_daily_request_quota(
        db=db,
        user_id="bob",
        limit=2,
    )

    assert get_daily_request_count(db, "alice") == 2
    assert get_daily_request_count(db, "bob") == 1
```

Alice reaching her limit must not block Bob.

## Program 2 — Show usage in React

Add this to `frontend/src/api.ts`:

```typescript
export type UsageSummary = {
  daily_limit: number;
  used_today: number;
  remaining_today: number;
};

export async function getMyUsage(
  apiToken: string,
): Promise<UsageSummary> {
  const response = await fetch(`${API_URL}/usage/me`, {
    headers: {
      Authorization: `Bearer ${apiToken}`,
    },
  });

  if (!response.ok) {
    throw new Error("Could not load usage");
  }

  return response.json() as Promise<UsageSummary>;
}
```

In `App.tsx`, import `UsageSummary` and `getMyUsage`:

```tsx
import {
  getMyUsage,
  type UsageSummary,
} from "./api";
```

Add state:

```tsx
const [usage, setUsage] = useState<UsageSummary | null>(null);
```

Load usage after a successful stream:

```tsx
const latestUsage = await getMyUsage(apiToken);
setUsage(latestUsage);
```

Display it:

```tsx
{usage && (
  <p>
    AI requests remaining today:{" "}
    {usage.remaining_today} / {usage.daily_limit}
  </p>
)}
```

## Program 3 — Handle `429` cleanly in React

Update the non-success response code in `streamAssistant()`:

```typescript
if (!response.ok || !response.body) {
  if (response.status === 429) {
    throw new Error(
      "You have reached your daily AI request limit. Try again tomorrow.",
    );
  }

  if (response.status === 401) {
    throw new Error("Your login token is invalid.");
  }

  throw new Error("Could not start AI stream");
}
```

Important: a user quota `429` is not a temporary provider error. Do not automatically retry it.

# Day 27 Quiz

1. Why should an app have its own per-user AI quota?  
   A. To control cost and prevent misuse  
   B. To remove the database  
   C. To avoid authentication  
   D. To replace provider limits  

2. Provider API rate limits are usually applied at which level?  
   A. Browser-tab level  
   B. Organization/project level  
   C. Individual React component level  
   D. Conversation-title level  

3. What status code should your app return when a user exceeds a daily quota?  
   A. 200  
   B. 401  
   C. 429  
   D. 500  

4. What does a `UsageEvent` record?  
   A. A user’s tracked application usage  
   B. A password  
   C. Frontend CSS  
   D. API-key value  

5. Why should the quota query filter by `user_id`?  
   A. To give each user an independent limit  
   B. To remove database indexes  
   C. To share all users’ usage  
   D. To bypass authentication  

6. Why use UTC for a daily reset?  
   A. It provides one consistent server-side time reference  
   B. It makes streaming faster  
   C. It removes rate limits  
   D. It encrypts messages  

7. When should quota be checked?  
   A. Before starting the expensive agent workflow  
   B. Only after the model completes  
   C. Only in React  
   D. Never for streamed requests  

8. What should a frontend do when it receives an app quota `429`?  
   A. Retry continuously  
   B. Show a clear limit-reached message  
   C. Reveal the API key  
   D. Delete user messages  

9. Which is useful to track for future AI cost analysis?  
   A. Input and output token usage  
   B. User passwords  
   C. CSS class names only  
   D. Laptop battery level  

10. What does `remaining_today` represent?  
   A. Requests still allowed for the user today  
   B. Provider API key balance only  
   C. Total conversations  
   D. Database file size  

11. If Alice has used her full quota, what should happen to Bob?  
   A. Bob should still have his own independent quota  
   B. Bob must be blocked too  
   C. The database must reset  
   D. All conversations are deleted  

12. Why should an app avoid showing exact internal provider errors to users?  
   A. They can expose sensitive system details  
   B. They make React faster  
   C. They remove SSE events  
   D. They create extra models  

13. What is the correct order?  
   A. Agent stream → quota check → authentication  
   B. Authentication → ownership check → quota check → agent stream  
   C. Database delete → agent stream  
   D. Frontend checks quota → trust it fully  

14. Does a provider rate limit replace an application’s user quota?  
   A. Yes, always  
   B. No, they solve different problems  
   C. Only for SQLite  
   D. Only for React  

15. What is the safest response to a user quota limit?  
   A. Automatically create another user account  
   B. Return `429`, show a safe message, and wait for quota reset or approved upgrade  
   C. Retry every second forever  
   D. Disable authentication  

Reply in this format:

```text
1.A
2.B
3.C
...
15.B
```
