# Day 35 AI — Quiz and Programs

**Topic:** permission-aware retrieval, organization isolation, role checks, and protected context.

**Time:** 1–3 hours.  
**Mac and Windows:** identical Python code. No API key required.

## Part 1 — MCQ Quiz

Choose one answer per question.

### 1. Which order matches our retrieval design?

A. Rank all documents → send to model → check permissions  
B. Filter by access → rank authorized documents → select results  
C. Ask the model to assign the user’s role → retrieve  
D. Retrieve everything → hide restricted citations  

### 2. Where should the user’s organization and roles come from?

A. Model-generated tool arguments  
B. An editable request body alone  
C. Trusted authentication and membership records  
D. The wording of the question  

### 3. A highly relevant document belongs to another organization. What should happen?

A. Exclude it from the user’s retrieval results  
B. Return it because relevance is high  
C. Include it without a citation  
D. Let the model decide whether to expose it  

### 4. What does this expression find?

```python
principal.roles & document.allowed_roles
```

A. Every role in either set  
B. Roles missing from the document  
C. The user’s organization  
D. Roles shared by the user and the document’s permission set  

### 5. Under our exercise’s policy, what does an empty `allowed_roles` set mean?

A. Everyone can access the document  
B. Nobody is granted access through that set  
C. Only the model can access the document  
D. Organization checks are skipped  

### 6. Which statement is correct?

A. Accessible documents are always relevant  
B. Relevant documents are always accessible  
C. Access and relevance are separate properties  
D. A high similarity score grants permission  

### 7. Why filter before selecting the top results?

A. Unauthorized documents must not occupy the result slots or enter context  
B. Sorting cannot process restricted documents  
C. It removes the need for document metadata  
D. It guarantees factual accuracy  

### 8. What does `frozen=True` establish for our `Principal` dataclass?

A. The identity is authenticated  
B. The roles came from a trusted database  
C. The user cannot impersonate anyone  
D. Normal field reassignment is blocked; authentication remains separate  

### 9. A model proposes `"organization_id": "org-b"` during a search. What should the application do?

A. Switch organizations automatically  
B. Use trusted server-side identity rather than treating that proposal as authority  
C. Search every organization  
D. Accept it if the JSON is valid  

### 10. What access information must remain available after document chunking?

A. Source organization and permissions  
B. Only the chunk’s word count  
C. Only the embedding vector  
D. Only the generated summary  

### 11. A user loses access to a restricted source. What is a risk when replaying an old cached answer?

A. Cached text cannot contain restricted information  
B. Saving an answer permanently grants access  
C. The cached answer may expose information the user may no longer access  
D. Idempotency automatically updates permissions  

### 12. Why use document ID as a tie-breaker when scores are equal?

A. It authenticates the document  
B. It makes the score more accurate  
C. It proves the document is current  
D. It makes result ordering deterministic  

### 13. Our word-overlap search returns a policy. What can we conclude?

A. The policy must answer the question correctly  
B. It passed our access filter and matched query terms under this scoring method  
C. It is the latest policy  
D. The model has permanently learned it  

### 14. A client receives another organization’s policy. What should you investigate first?

A. Trusted identity, metadata, filtering, and cache boundaries  
B. Only the answer’s writing style  
C. Only the temperature setting  
D. The dashboard colour theme  

### 15. Which result demonstrates correct behaviour when `limit=1`?

A. Return the highest-scoring document even if unauthorized  
B. Return no results whenever any unauthorized document matches  
C. Return the highest-ranked authorized matching document  
D. Return restricted text but omit its document ID  

## Part 2 — Programs

Continue in your `day35_permission_aware_rag` practice folder. Keep your main backend unchanged.

### Program 1 — Complete the retrieval tests

Add these six tests to `test_retrieval.py`:

1. Empty `allowed_roles` denies access.
2. A blank query raises `ValueError`.
3. A limit of `0` raises `ValueError`.
4. An accessible document with no matching words is excluded.
5. Equal scores are ordered by document ID.
6. A higher-scoring unauthorized document does not crowd out an authorized result when `limit=1`.

For test 6, construct documents like:

```python
documents = [
    Document(
        document_id="RESTRICTED",
        organization_id="org-a",
        allowed_roles=frozenset({"finance"}),
        text="meal allowance approval procedure",
    ),
    Document(
        document_id="EMPLOYEE",
        organization_id="org-a",
        allowed_roles=frozenset({"employee"}),
        text="meal allowance",
    ),
]
```

Search as an employee:

```python
hits = search_documents(
    employee,
    "meal allowance approval procedure",
    documents,
    limit=1,
)
```

Expected result IDs:

```python
["EMPLOYEE"]
```

**Acceptance criterion:** ten tests pass—the original four plus these six.

Run with your virtual environment activated:

```bash
python -m unittest -v
```

### Program 2 — Prove unauthorized text is never scored

**Goal:** verify that filtering happens before scoring—not merely before returning results.

Patch `tokenize()` while allowing its original behaviour:

```python
from unittest.mock import patch
```

Inside your test:

```python
with patch(
    "retrieval.tokenize",
    wraps=tokenize,
) as tracked_tokenize:
    search_documents(employee, "meal allowance", documents)

    scored_inputs = [
        call.args[0]
        for call in tracked_tokenize.call_args_list
    ]
```

Add assertions proving:

- The query was tokenized.
- An authorized document’s text was tokenized.
- An unauthorized document’s text was **not** tokenized.

Import the original function for `wraps`:

```python
from retrieval import tokenize
```

**Acceptance criterion:** the test fails if authorization is moved after document scoring.

This checks our local implementation’s ordering. It is not proof that every component of a production search service handles data securely.

### Program 3 — Build a trusted search wrapper

**Goal:** prevent model output from selecting the user’s identity.

Implement:

```python
def make_policy_search(principal, documents):
    def search(query):
        return search_documents(principal, query, documents)

    return search
```

Create separate wrappers:

```python
alice_search = make_policy_search(alice, documents)
bob_search = make_policy_search(bob, documents)
```

Requirements:

- The returned function accepts a query—not a user ID, organization, or roles.
- Alice’s wrapper returns only Alice’s permitted results.
- Bob’s wrapper returns only Bob’s permitted results.
- Both wrappers can receive the same query and return different authorized results.

**Acceptance criterion:** identity is supplied by the trusted application boundary, not by the search proposal.

In this local lab, you construct the principals manually. A production backend must construct them from verified identity and membership information.

## Part 3 — EOD Mini-Project: Policy Access Matrix

Create synthetic documents and test these scenarios:

| Identity | Organization | Roles | Expected access |
|---|---|---|---|
| Alice | A | Employee | A’s employee policy |
| Fiona | A | Finance | A’s employee and finance policies, under our configured role sets |
| Bob | B | Employee | B’s employee policy |
| Charlie | A | No roles | No documents |
| Alice after role removal | A | No roles | No documents on a fresh search |

Use the same query for every identity.

For each scenario, print:

```text
Identity:
Expected document IDs:
Actual document IDs:
Passed:
```

Determine `Passed` using assertions or comparisons. Do not print restricted document text for demonstration purposes.

### Permission-change requirement

Create a **new trusted principal** representing Alice’s current role membership. Do not reuse an old principal and assume its roles refresh automatically.

This exercise tests fresh retrieval. Cache invalidation and protected saved-answer replay require additional controls.

## Part 4 — Corporate and Client Investigation

A client reports:

> “The source document had correct restrictions, but its chunks appeared in another team’s AI answers.”

Write a short investigation note:

1. Were permissions preserved during chunk creation?
2. Did each chunk retain a reliable link to its source?
3. Were organization and role filters applied during retrieval?
4. Did any cache reuse results across users or organizations?
5. Which regression test would reproduce the exposure safely?

Use synthetic content, not actual confidential documents.

## EOD Deliverables

- Ten basic retrieval tests.
- One filter-before-scoring test.
- A trusted search-wrapper demonstration.
- A five-scenario access matrix.
- A short chunk-permission investigation note.

**Remember: access checks belong before protected information enters the model’s context.**

Reply with your quiz answers as `1.B, 2.C, ...`. Share your code or terminal output separately for practical review.
