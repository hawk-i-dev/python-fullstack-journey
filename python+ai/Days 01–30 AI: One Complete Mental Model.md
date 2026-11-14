# Days 01–30 AI: One Complete Mental Model

**You are learning to build a controlled software system around an AI model—not just send prompts to it.**

The model is one component. Python, data, permissions, tools, storage, reliability, and evaluation make it useful.

One honest distinction: these 30 days develop your foundation in **AI application engineering**. Training an LLM from scratch is a separate path involving mathematics, deep learning, training data, and compute. Quiz scores show conceptual understanding; working programs and debugging show practical ability.

## 1. The whole system in one picture

```text
USER REQUEST
     ↓
Validate input + authenticate user
     ↓
Check permissions + usage limits
     ↓
Load relevant conversation history
     ↓
Retrieve permitted knowledge, if needed
     ↓
MODEL proposes an answer or tool call
     ↓
APPLICATION validates and controls execution
     ↓
Return / stream the result + persist appropriate records
     ↓
Measure errors, latency, and token usage
     ↓
Collect feedback → review → regression tests → improve
```

**Python connects the pieces. The model does not replace your application’s responsibilities.**

## 2. Where every day fits

| Days | Concepts | Purpose in the system |
|---|---|---|
| 1–3 | Objects, collections, loops, functions, scope | Represent and transform information |
| 4–5 | Exceptions, logging, tests | Make failures visible and behaviour testable |
| 6–9 | Classes, files, JSON, modules, Git | Organize, persist, and maintain your code |
| 10–13 | NumPy, Pandas, chunking, TF-IDF, similarity | Prepare data and retrieve useful evidence |
| 14–16 | Grounded RAG, model adapter, conversation history | Generate answers using context |
| 17–19 | Tool calling, routing, guardrails, approvals | Add controlled actions |
| 20–23 | Tracing, evaluations, API, auth, database | Make the system inspectable and usable |
| 24–26 | Streaming, frontend, timeouts, retries | Deliver a responsive, more resilient experience |
| 27–28 | Quotas, token usage, latency | Control consumption and observe performance |
| 29–30 | Feedback, reviewed cases, regression runner | Detect problems and check that fixes last |

These are connected layers, not 30 unrelated topics.

## 3. What is common across all 30 days?

Almost everything follows this pattern:

> **Receive → validate → transform → act → verify → record.**

### Example: a Python function

```text
Receive arguments
→ validate values
→ calculate
→ return result
→ test expected behaviour
```

### Example: a tool call

```text
Receive proposed tool arguments
→ validate schema and permissions
→ execute approved operation
→ inspect result
→ record outcome
```

### Example: an evaluation

```text
Receive cases and outputs
→ validate IDs and structure
→ compare with expectations
→ report failures
→ save results
```

The scale changes. The engineering habit stays the same.

## 4. The seven questions to ask about any feature

Before writing code, ask:

1. **Input:** What enters this component?
2. **Contract:** What shape and meaning must that input have?
3. **Permission:** Who is allowed to use it or access its data?
4. **Behaviour:** What should happen?
5. **Failure:** What can go wrong?
6. **Evidence:** How will I observe what happened?
7. **Verification:** How will I prove it works within its intended scope?

Example: feedback submission.

```text
Input: message ID, rating, optional comment
Contract: rating must be 1 or -1
Permission: requester owns the conversation
Behaviour: create or update that user's rating
Failure: missing message, invalid data, database error
Evidence: response status and stored record
Verification: validation, ownership, and duplicate-submission tests
```

This is a reusable design checklist for institute assignments, corporate projects, and client work.

## 5. Understand the differences that matter

### Retrieval versus generation

**Retrieval:** find relevant evidence.  
**Generation:** produce an answer using the supplied information.

If the right policy never reaches the model, changing the wording of the prompt may not fix the underlying problem.

Also remember:

- Similarity is not truth.
- A relevant document can be outdated.
- A citation can be present without supporting the claim.
- RAG supplies context; it does not automatically retrain the model.

### History versus knowledge versus learned parameters

- **History:** previous conversation messages selected for the current request.
- **Knowledge source:** documents or database records retrieved when needed.
- **Model parameters:** patterns learned during training.

Saving a conversation does not automatically teach the model permanently. Your application must load appropriate history when needed.

### Tool proposal versus execution

A model may propose:

```text
delete_document(document_id=42)
```

Your application must still determine:

- Does this tool exist?
- Are its arguments valid?
- Does the user own document 42?
- Is approval required?
- Has this operation already happened?

**Model output is not authorization.**

### Authentication versus authorization

- Authentication: **Who are you?**
- Authorization: **May you access this particular resource?**

A valid login must not give access to every conversation, message, or feedback record.

### Streaming versus completion

Streaming means some output has arrived—not that the entire operation succeeded.

The connection can fail after partial text. That is why blindly replaying a request can duplicate output or actions.

### Tests versus evaluations

- A unit test checks a small function.
- An integration test checks components working together.
- An agent evaluation checks behaviour against task-specific expectations.
- Human review assesses nuance that automated checks may miss.

Our Day 30 runner checked saved fixtures. It did **not** execute the real agent or prove semantic correctness.

## 6. Follow one request end to end

Suppose a user asks:

> “What is my meal allowance?”

### Step 1 — Accept the request

Validate the request shape. Identify the user from trusted authentication—not a user-supplied `user_id`.

### Step 2 — Check access

Verify conversation ownership and retrieve only documents the user may access.

### Step 3 — Gather context

Load bounded, relevant history and retrieve policy passages.

If the organization or policy is unknown, clarification may be better than a confident answer.

### Step 4 — Generate or use a tool

The model receives instructions, the question, and selected context.

If it requests a tool, the application validates and controls execution.

### Step 5 — Return the result

Stream appropriate events, handle failures clearly, and persist the completed result according to your design.

### Step 6 — Observe and improve

Record useful operational metadata without unnecessarily exposing sensitive content.

If the answer receives a 👎:

```text
Review evidence
→ identify the failure
→ define expected behaviour
→ add a sanitized evaluation case
→ fix one cause
→ rerun existing cases
```

OpenAI’s evaluation guidance supports this continuous, task-specific approach and recommends combining automated checks with human judgment. [Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)

## 7. Your troubleshooting strategy

**Find the first incorrect boundary—not merely the final visible symptom.**

| Symptom | First checks |
|---|---|
| Button does nothing | Is the correct component running? Is the handler connected? |
| No request reaches the backend | Browser Network tab, URL, event handler |
| Request rejected | Status code, request body, authentication |
| Wrong user’s data appears | Ownership checks and database filters |
| Answer is incorrect | Retrieved evidence, history, instructions, actual output |
| Tool fails | Tool name, arguments, permissions, service response |
| Partial answer stops | Streaming events, timeout, connection, backend logs |
| Requests become expensive | Context size, output length, repeated calls, retry behaviour |
| Bad answer passes evaluation | Evaluator blind spot or weak expectations |

Use this loop:

> **Reproduce → inspect → form a hypothesis → change one thing → retest → add regression coverage.**

Avoid changing the prompt, model, database, and frontend simultaneously. You lose the ability to tell which change helped.

## 8. What to remember—and what to look up

### Remember deeply

- Data types, mutability, scope, and control flow.
- Function inputs, outputs, and side effects.
- Validation and resource ownership.
- Retrieval and generation are different stages.
- Model-proposed actions need application controls.
- Retries can duplicate work.
- Feedback is a signal, not ground truth.
- Passing checks proves only what those checks cover.

### Look up when needed

- Exact library imports and method names.
- Provider event formats and SDK options.
- Installation commands.
- Model prices and limits.
- Framework configuration.

**Mastery is understanding the behaviour well enough to find, verify, and apply the syntax—not memorizing every command.**

Mac and Windows change some setup commands and paths. These mental models remain the same.

## 9. How to manage work professionally

### Institute level

Explain the design, demonstrate normal and failing cases, and show your tests. Do not submit code you cannot explain.

### Corporate level

Use clear ownership, reviewed changes, reproducible tests, protected secrets, useful logs, and documented failure handling.

### Client level

Agree on:

- Supported tasks.
- Allowed data and actions.
- Acceptance criteria.
- Cost and latency expectations.
- Known limitations.
- Escalation and rollback procedures.

Say:

> “These scenarios passed these checks.”

Do not turn that into:

> “The AI is always accurate.”

## 10. Your 1–3-hour daily approach

| Time | Activity |
|---|---|
| 15 minutes | Recall yesterday’s concepts without notes |
| 25 minutes | Understand one new concept |
| 50 minutes | Build one small working feature |
| 20 minutes | Deliberately break it and investigate |
| 10 minutes | Record lessons and remaining questions |
| Optional extra hour | Add tests, edge cases, and documentation |

A small, tested feature is a valid EOD project increment. You do not need a brand-new application every day.

## Final consolidation assignment

Using your existing project, demonstrate these five scenarios:

1. Relevant policy exists → grounded answer.
2. Policy is missing → uncertainty or clarification.
3. Another user requests a private conversation → access denied.
4. A tool or service fails → clear failure handling.
5. A candidate output breaks an existing expectation → regression reported.

For each, write:

```text
Expected behaviour:
Actual behaviour:
Evidence:
Cause, if failing:
Fix:
Test that prevents recurrence:
```

**The complete mental model to carry forward:**

> Build with clear contracts. Control access and actions. Inspect evidence. Test failures. Measure changes. Learn from reviewed feedback.
