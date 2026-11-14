# Days 01–30 AI — Master Quiz

**40 MCQs · One correct answer each · Suggested time: 40–50 minutes**

Attempt without notes first. These questions test understanding and troubleshooting—not syntax memorization.

## Python foundations — Days 1–9

### 1. What does this print?

```python
original = ["policy"]
copied = original
copied.append("invoice")

print(original)
```

A. `["policy"]`  
B. `["policy", "invoice"]`  
C. `["invoice"]`  
D. An error  

### 2. What does this function return?

```python
def clean_title(title):
    title.strip()

result = clean_title("  Travel  ")
```

A. `"Travel"`  
B. `"  Travel  "`  
C. `None`  
D. `True`  

### 3. You need to process every item in a list of documents. Which is most appropriate?

A. A `for` loop  
B. A single `if` statement  
C. A `return` outside a function  
D. An exception handler only  

### 4. Why is this function risky?

```python
def add_message(message, history=[]):
    history.append(message)
    return history
```

A. Lists cannot be function arguments  
B. `append()` creates a new list every time  
C. Default arguments are evaluated after each call  
D. The same default list can be shared across calls  

### 5. A JSON file contains invalid syntax. What is the best response?

A. Silently replace it with an empty dataset  
B. Report the parsing problem with useful context  
C. Retry reading it forever  
D. Treat the entire file as a successful result  

### 6. Which log entry is safest and most useful?

A. Full API key and request headers  
B. Every user’s private conversation  
C. Request ID, operation, duration, and sanitized error details  
D. Password and database connection secret  

### 7. What is the main purpose of a unit test?

A. Check a small, isolated piece of behaviour  
B. Guarantee the whole application is secure  
C. Replace all manual investigation  
D. Train the model  

### 8. Why separate authentication, retrieval, and model calls into modules?

A. Python requires one function per file  
B. It guarantees faster model responses  
C. It eliminates runtime errors  
D. It makes responsibilities easier to understand, test, and change  

## Data and RAG — Days 10–16

### 9. What does a high similarity score primarily suggest?

A. The document is guaranteed to be true  
B. The user is authorized to access it  
C. The document is similar to the query under that representation  
D. The document is the latest version  

### 10. Which approach best identifies missing values in a Pandas column?

A. Use an explicit missing-value check such as `isna()`  
B. Assume every empty-looking value equals zero  
C. Convert the entire dataset to strings  
D. Remove all rows without inspecting them  

### 11. Why split long documents into chunks?

A. To permanently train the model on each paragraph  
B. To remove the need for retrieval  
C. To guarantee every answer is correct  
D. To retrieve manageable passages relevant to a question  

### 12. A retrieval system returns an unrelated document. What should you investigate first?

A. The frontend button colour  
B. Query processing, chunking, representation, and ranking  
C. The user’s screen resolution  
D. The answer’s font size  

### 13. What does RAG normally do?

A. Permanently updates model parameters for each document  
B. Replaces all database queries  
C. Retrieves external information and supplies it as generation context  
D. Guarantees answers never contain unsupported claims  

### 14. The relevant policy is unavailable. What is the best behaviour?

A. Explain the limitation and request the needed information  
B. Invent a plausible policy  
C. Cite an unrelated document  
D. Use a confident tone to hide uncertainty  

### 15. An answer includes citation `P1`. What does that establish by itself?

A. Every claim is supported by P1  
B. P1 is current  
C. The answer is factually correct  
D. The output references P1; support still needs checking  

### 16. Why use a model adapter?

A. To put provider API keys in the browser  
B. To isolate provider-specific calls from application logic  
C. To avoid handling errors  
D. To give the model direct database ownership  

### 17. Saving messages in SQLite means:

A. The model automatically learns them permanently  
B. Every future request automatically includes them  
C. The application can load appropriate history when needed  
D. Token limits no longer apply  

### 18. Why bound the conversation history sent to a model?

A. To manage context size, relevance, latency, and consumption  
B. Because databases cannot store long conversations  
C. To remove authentication  
D. To guarantee deterministic answers  

## Agents and safety — Days 17–20

### 19. A model proposes deleting document 42. What must happen before execution?

A. Delete it immediately because the model requested it  
B. Ask the model whether the user is authorized  
C. Assume document IDs are secret enough  
D. Validate arguments, permissions, and any required approval  

### 20. What is routing in an agent application?

A. Changing the browser’s colour theme  
B. Selecting the appropriate workflow or capability for a request  
C. Saving every message in one file  
D. Retrying every failed request  

### 21. A tool call has valid JSON but requests another user’s record. What should happen?

A. Execute it because the JSON is valid  
B. Execute it if the model sounds confident  
C. Reject it because valid structure does not establish authorization  
D. Remove logging and continue  

### 22. What is the main purpose of a trace?

A. Record workflow events so you can investigate what happened  
B. Expose hidden model reasoning  
C. Replace access control  
D. Guarantee the answer is accurate  

### 23. Which action most clearly calls for an approval mechanism?

A. Formatting a local string  
B. Counting words in a supplied paragraph  
C. Sorting an in-memory list  
D. Sending a consequential external message on a user’s behalf  

### 24. Why evaluate tool selection separately from final answer wording?

A. A fluent answer proves the right tool was used  
B. An agent may produce polished text while using the wrong tool or arguments  
C. Tool behaviour cannot be tested  
D. Tool results never affect answer quality  

## Application engineering — Days 21–28

### 25. What is the difference between authentication and authorization?

A. They are identical  
B. Authorization identifies the user; authentication checks document ownership  
C. Authentication identifies the requester; authorization checks permitted access  
D. Both can safely be delegated to a user-supplied ID  

### 26. Where should the backend normally obtain the authenticated user identity?

A. A trusted authentication mechanism  
B. An editable `user_id` field in the request body  
C. The conversation title  
D. The browser’s display name alone  

### 27. Why must message queries check conversation ownership?

A. Message IDs are always impossible to guess  
B. A login grants access to every message  
C. Frontend navigation already enforces security  
D. Authentication alone does not establish access to a particular conversation  

### 28. What does streaming establish?

A. The complete operation has succeeded  
B. Some events or output can arrive before completion  
C. The answer is already saved successfully  
D. Retrying the request cannot duplicate work  

### 29. Where should the provider API key be kept?

A. In frontend source code  
B. In a public repository  
C. In protected backend configuration  
D. In every outgoing browser request  

### 30. Which failure is generally a reasonable candidate for a bounded retry?

A. A transient connection failure, when replaying the operation is safe  
B. Invalid request fields  
C. Missing permission  
D. An incorrect API credential  

### 31. Why avoid blindly retrying after a stream has partially completed?

A. Streams cannot fail  
B. Python prevents retries  
C. Streaming responses contain no state  
D. It can duplicate output, requests, or consequential actions  

### 32. Why implement application-level per-user quotas?

A. Provider limits always enforce your exact user-level policy  
B. Your application needs its own fairness and consumption controls  
C. Quotas replace authentication  
D. Token usage cannot be measured  

### 33. Which statement about usage and latency is correct?

A. Fast responses are always correct  
B. Short visible answers always have negligible usage  
C. Measure usage and duration; do not infer quality or cost solely from appearance  
D. Measuring latency replaces evaluation  

## Feedback and evaluations — Days 29–30

### 34. What does a negative user rating mean?

A. A signal requiring investigation—not automatic proof of an error  
B. The user’s preferred answer must become ground truth  
C. The model must immediately be retrained  
D. The conversation should be deleted  

### 35. What should happen before private production feedback becomes an evaluation case?

A. Publish it unchanged  
B. Automatically accept every complaint  
C. Remove the expected behaviour  
D. Review, sanitize, and define the intended behaviour  

### 36. Why match evaluation cases and outputs by stable IDs?

A. IDs guarantee factual accuracy  
B. Matching remains meaningful when record order changes  
C. IDs remove the need for schema validation  
D. IDs make missing outputs pass  

### 37. A test case has no corresponding output. What should the runner do?

A. Count it as passed  
B. Copy another case’s output  
C. Report it as missing and fail that case  
D. Ignore it without reporting  

### 38. A candidate improves one case but breaks another, leaving the overall score unchanged. What should you conclude?

A. Inspect case-level changes; an unchanged score can hide a regression  
B. The candidate is identical to the baseline  
C. No regression occurred  
D. Delete the broken case  

### 39. Why test the evaluator with deliberately bad outputs?

A. To train the agent automatically  
B. To increase model token usage  
C. To eliminate human review forever  
D. To verify the evaluator detects the failures it claims to detect  

### 40. Your local runner reports `10/10 passed` on handwritten fixtures. What can you responsibly claim?

A. The real agent is fully safe and accurate  
B. Those fixtures passed the implemented checks; real-agent behaviour needs separate testing  
C. All future users will receive correct answers  
D. The application needs no further tests  

---

Reply with:

```text
1.B
2.C
...
40.B
```

I’ll score it, explain any corrections, and identify which areas need practical revision.
