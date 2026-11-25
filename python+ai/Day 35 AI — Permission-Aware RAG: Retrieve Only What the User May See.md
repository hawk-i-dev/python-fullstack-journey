# Day 35 AI — Permission-Aware RAG: Retrieve Only What the User May See

**Today’s project:** build a small document retriever that enforces organization and role permissions **before ranking documents**.

This reinforces your Day 34 revision point:

> Permission comes from trusted application policy—not from the model.

**Time:** 1–3 hours.  
**Requirements:** Python standard library only. No API key.  
**Mac and Windows:** identical code.

Today’s search uses simple word overlap so you can inspect every decision. It is **not an embedding-based retriever or a live LLM integration**.

## 1. The problem

An employee asks:

> “What is the meal allowance?”

Your search index contains:

- Their organization’s employee policy.
- Their organization’s restricted finance policy.
- Another organization’s policy.

A search that returns the most similar document without checking access could expose restricted information.

A polite final answer cannot undo that exposure if the restricted text already reached the model.

### The correct boundary

```text
Authenticated user
       ↓
Trusted organization and role information
       ↓
Filter documents by access
       ↓
Rank authorized documents
       ↓
Select relevant passages
       ↓
Send only those passages to the model
```

**The model must not decide which organization or role the user belongs to.**

## 2. Understand three separate checks

| Check | Question |
|---|---|
| Organization boundary | Does this document belong to the user’s organization? |
| Document permission | May this user’s role access it? |
| Relevance | Does it help answer this question? |

An accessible document is not necessarily relevant.

A relevant document is not necessarily accessible.

## 3. Create a practice folder

Create a new folder named `day35_permission_aware_rag`.

Add:

```text
retrieval.py
demo.py
test_retrieval.py
```

Keep your existing projects unchanged.

## 4. Implement trusted identity and document metadata

Put this in `retrieval.py`:

```python
import re
from dataclasses import dataclass


@dataclass(frozen=True)
class Principal:
    user_id: str
    organization_id: str
    roles: frozenset[str]


@dataclass(frozen=True)
class Document:
    document_id: str
    organization_id: str
    allowed_roles: frozenset[str]
    text: str


@dataclass(frozen=True)
class SearchHit:
    document_id: str
    text: str
    score: int


def tokenize(text: str) -> set[str]:
    return set(re.findall(r"[a-z0-9]+", text.lower()))


def can_read(principal: Principal, document: Document) -> bool:
    same_organization = (
        principal.organization_id == document.organization_id
    )

    has_allowed_role = bool(
        principal.roles & document.allowed_roles
    )

    return same_organization and has_allowed_role


def search_documents(
    principal: Principal,
    query: str,
    documents: list[Document],
    *,
    limit: int = 3,
) -> list[SearchHit]:
    if not isinstance(query, str):
        raise ValueError("Query must be a string")

    if not query.strip() or len(query) > 200:
        raise ValueError("Query must contain 1–200 characters")

    if type(limit) is not int or not 1 <= limit <= 10:
        raise ValueError("Limit must be an integer from 1 to 10")

    query_terms = tokenize(query)

    if not query_terms:
        return []

    hits = []

    for document in documents:
        # Authorization happens before relevance scoring.
        if not can_read(principal, document):
            continue

        score = len(query_terms & tokenize(document.text))

        if score > 0:
            hits.append(
                SearchHit(
                    document_id=document.document_id,
                    text=document.text,
                    score=score,
                )
            )

    hits.sort(key=lambda hit: (-hit.score, hit.document_id))

    return hits[:limit]
```

### Understand the Python

**`frozenset`**

An immutable set. Useful here because identity roles and permission metadata should not be casually modified during retrieval.

**Set intersection**

```python
principal.roles & document.allowed_roles
```

Finds shared roles.

Our exercise uses **any-role matching**: at least one shared role permits access within the same organization.

**Default denial**

An empty `allowed_roles` set grants nobody access.

We do not interpret missing permission as “public.”

**Filtering before ranking**

Unauthorized documents never enter the scoring or result-selection stage.

**Deterministic sorting**

```python
(-hit.score, hit.document_id)
```

Sorts by descending score, then document ID to resolve ties predictably.

### Important limitation

`frozen=True` does not authenticate a `Principal`.

For this exercise, we construct identities manually. In your real application, the backend must build them from verified authentication and trusted membership records.

## 5. Demonstrate organization and role isolation

Put this in `demo.py`:

```python
from retrieval import Document, Principal, search_documents


def sample_documents():
    return [
        Document(
            document_id="A-EMPLOYEE",
            organization_id="org-a",
            allowed_roles=frozenset({"employee", "finance"}),
            text="The meal allowance is INR 500 per day.",
        ),
        Document(
            document_id="A-FINANCE",
            organization_id="org-a",
            allowed_roles=frozenset({"finance"}),
            text=(
                "Restricted finance meal allowance exception "
                "approval procedure."
            ),
        ),
        Document(
            document_id="B-EMPLOYEE",
            organization_id="org-b",
            allowed_roles=frozenset({"employee"}),
            text="The meal allowance is INR 900 per day.",
        ),
    ]


def main():
    alice = Principal(
        user_id="alice",
        organization_id="org-a",
        roles=frozenset({"employee"}),
    )

    hits = search_documents(
        alice,
        "meal allowance",
        sample_documents(),
    )

    print("Visible search results:")

    for hit in hits:
        print(f"{hit.document_id}: {hit.text}")

    assert {hit.document_id for hit in hits} == {"A-EMPLOYEE"}


if __name__ == "__main__":
    main()
```

Run with your virtual environment activated:

```bash
python demo.py
```

On Mac, outside your environment, use `python3` if needed.

Expected:

```text
Visible search results:
A-EMPLOYEE: The meal allowance is INR 500 per day.
```

Alice must not receive either the finance document or Organization B’s policy—even though both match the question.

## 6. Test the permission boundary

Put this in `test_retrieval.py`:

```python
import unittest

from demo import sample_documents
from retrieval import Principal, search_documents


class RetrievalTests(unittest.TestCase):
    def setUp(self):
        self.documents = sample_documents()

    def test_employee_gets_only_permitted_document(self):
        alice = Principal(
            "alice",
            "org-a",
            frozenset({"employee"}),
        )

        hits = search_documents(
            alice,
            "meal allowance",
            self.documents,
        )

        self.assertEqual(
            {hit.document_id for hit in hits},
            {"A-EMPLOYEE"},
        )

    def test_finance_role_gets_permitted_finance_document(self):
        finance_user = Principal(
            "fiona",
            "org-a",
            frozenset({"finance"}),
        )

        hits = search_documents(
            finance_user,
            "meal allowance",
            self.documents,
        )

        self.assertEqual(
            {hit.document_id for hit in hits},
            {"A-EMPLOYEE", "A-FINANCE"},
        )

    def test_other_organization_is_isolated(self):
        bob = Principal(
            "bob",
            "org-b",
            frozenset({"employee"}),
        )

        hits = search_documents(
            bob,
            "meal allowance",
            self.documents,
        )

        self.assertEqual(
            {hit.document_id for hit in hits},
            {"B-EMPLOYEE"},
        )

    def test_no_roles_means_no_results(self):
        user = Principal(
            "charlie",
            "org-a",
            frozenset(),
        )

        hits = search_documents(
            user,
            "meal allowance",
            self.documents,
        )

        self.assertEqual(hits, [])


if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

**Expected:** four passing tests.

## 7. Connect this to Day 34

Day 34 validated a proposed action:

```json
{
  "action": "search_policy",
  "message": "",
  "query": "meal allowance"
}
```

Day 35 supplies the next boundary:

```text
Validated search proposal
       ↓
Check whether search is permitted
       ↓
Obtain trusted principal
       ↓
Retrieve only accessible documents
```

The model supplies the proposed query.

**It does not supply authoritative roles or organization membership.**

When integrating, a trusted backend wrapper should supply `principal` to `search_documents()`. Do not expose identity selection as a model-controlled tool argument.

## 8. Two subtle production mistakes

### Mistake 1: Permission metadata disappears during chunking

Suppose you split a restricted document into ten chunks but store only:

```text
chunk_id, text, vector
```

You have lost the access-control information needed during retrieval.

Every retrievable chunk must retain—or reliably resolve to—its source organization and permissions.

### Mistake 2: Cached answers bypass current permissions

Days 32–33 introduced saved results and replay.

Suppose a user had finance access yesterday but lost it today.

A cached answer may still contain restricted information.

**Authentication and current authorization must be considered before replaying protected results.** A stored result is not a permanent access grant.

## 9. Your assignment

Add six tests:

1. An empty `allowed_roles` set denies access.
2. A blank query is rejected.
3. A limit of `0` is rejected.
4. An accessible document with no matching words is excluded.
5. Equal scores are ordered by document ID.
6. A higher-scoring unauthorized document is excluded and does not crowd out an authorized result when `limit=1`.

**Expected total:** ten tests.

### EOD mini-project

Create a local policy-search console demonstration with:

- Two organizations.
- Employee and finance roles.
- One document with no permitted roles.
- The same question asked by different trusted test identities.
- Assertions verifying each identity’s result IDs.

Print only the documents that identity may access. Do not “demonstrate filtering” by exposing restricted document text first.

## 10. Troubleshooting approach

A client reports:

> “An employee received another organization’s policy.”

Investigate the boundary in order:

1. Where did organization identity come from?
2. Was document/chunk permission metadata correct?
3. Was filtering applied before result selection?
4. Did a tool accept model-supplied identity?
5. Did a cache or saved answer bypass access checks?
6. What actually entered the model’s context?

Do not start by changing the answer prompt.

### Institute, corporate, and client approach

- **Institute:** explain relevance versus authorization and demonstrate denied cases.
- **Corporate:** test organization isolation, role changes, chunk metadata, and cached results.
- **Client:** document who can access which sources and how permission changes take effect.

## EOD checklist

- [ ] Organization boundaries are enforced.
- [ ] Role permissions are checked before ranking.
- [ ] Missing permissions deny access.
- [ ] Unauthorized results never reach the answer context.
- [ ] Ten tests pass.
- [ ] You can explain why model-generated permissions are untrusted.

**Remember: retrieve the most relevant information the user may access—not the most relevant information in the entire system.**
