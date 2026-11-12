# Day 30 AI — Quiz and Programs

Today’s focus: **regression testing, evaluating saved agent outputs, and testing the evaluator itself.**

**Time:** 1–3 hours. Python code is the same on Mac and Windows. No API key is needed.

## Part 1 — MCQ quiz

Choose one answer per question. Answers are withheld so you can attempt them independently.

### 1. What is a regression?

A. A previously working behaviour breaks after a change  
B. A new database table is created  
C. A model responds faster  
D. A user opens another conversation  

### 2. What does our Day 30 runner evaluate?

A. The model’s internal reasoning  
B. Saved structured outputs against predefined expectations  
C. Every possible future agent response  
D. The security of the entire application  

### 3. Why include context in an evaluation case?

A. To make every answer longer  
B. To avoid writing expected behaviour  
C. Correct behaviour can depend on available information  
D. Context guarantees the model cannot hallucinate  

### 4. Why match cases and outputs using IDs?

A. IDs make the model more accurate  
B. Matching remains correct when records are reordered  
C. IDs remove the need for validation  
D. JSON requires every object to have an ID  

### 5. What should happen when a test case has no corresponding output?

A. Skip it and count it as passed  
B. Remove it from the total automatically  
C. Copy another output  
D. Report a failure  

### 6. What does this expression find?

```python
set(required_citations) - set(actual_citations)
```

A. Required citations missing from the output  
B. All citations in both collections  
C. Duplicate citations  
D. Citations sorted alphabetically  

### 7. Why reject duplicate case IDs?

A. Duplicate IDs increase API costs  
B. Python dictionaries cannot contain strings  
C. One record could silently overwrite another during indexing  
D. JSON cannot contain repeated values  

### 8. What does exit code `0` mean in our runner?

A. Zero cases were loaded  
B. All configured checks passed  
C. The model made no mistakes anywhere  
D. Human review is unnecessary  

### 9. An output has `"action": "refuse"` but reveals private information in its answer. What does this show?

A. The output is safe because the action is correct  
B. The database is broken  
C. The user must change the question  
D. Checking an action label alone cannot establish safety  

### 10. An answer cites `P1`. Does that prove P1 supports the answer?

A. No; citation presence and factual support are different checks  
B. Yes, if P1 exists  
C. Yes, if the answer is long  
D. Only when the model is expensive  

### 11. Why deliberately give the evaluator incorrect outputs?

A. To improve the agent’s training automatically  
B. To make the dataset larger  
C. To verify that the evaluator detects failures  
D. To avoid testing correct outputs  

### 12. The runner reports `3/3 passed`. What is the most accurate interpretation?

A. The agent is 100% accurate  
B. These three saved results passed the implemented checks  
C. All future requests will pass  
D. The application is ready for every client  

### 13. A correct answer fails an evaluation. What should you do first?

A. Investigate the output, rubric, and evaluator  
B. Delete the case immediately  
C. Assume the model is always wrong  
D. Remove all failing checks  

### 14. Why keep a fixed evaluation set when comparing two agent versions?

A. It guarantees identical model outputs  
B. It eliminates the need for human review  
C. It prevents all production failures  
D. It makes the comparison more meaningful by keeping cases consistent  

### 15. Which client update is most responsible?

A. “Our AI never makes mistakes.”  
B. “Four structured checks passed; factual correctness still needs review.”  
C. “We tested one response, so the system is production-ready.”  
D. “A green report proves there are no security issues.”  

## Part 2 — Programs

Build these on top of your Day 30 evaluation lab. Do not replace your existing agent application.

### Program 1 — Unit-test the evaluator

**Goal:** prove that `evaluate()` catches the problems it claims to detect.

Create `test_runner.py` beside `runner.py`.

Starter:

```python
import unittest

from runner import evaluate


class EvaluatorTests(unittest.TestCase):
    def setUp(self):
        self.case = {
            "id": "policy-check",
            "expected_action": "answer",
            "required_citations": ["P1"],
        }

    def test_valid_output_passes(self):
        output = {
            "action": "answer",
            "answer": "The allowance is INR 500 per day.",
            "citations": ["P1"],
        }

        self.assertEqual(evaluate(self.case, output), [])

    # Add the remaining tests here.


if __name__ == "__main__":
    unittest.main()
```

Add tests for:

1. Wrong action.
2. Missing citation.
3. Answer containing only spaces.
4. Missing output.
5. `citations` supplied as a string instead of a list.

Run on Mac or Windows with your virtual environment active:

```bash
python -m unittest -v
```

**Acceptance criterion:** all six tests pass—including tests that verify bad outputs are rejected.

Remember: a unit test passes when the evaluator correctly identifies an invalid result.

### Program 2 — Save a useful JSON report

**Goal:** make failures inspectable without copying terminal output.

Extend the runner to write `report.json` in the evaluation-lab folder.

Expected structure:

```json
{
  "total": 3,
  "passed": 2,
  "failed": 1,
  "pass_rate": 0.6667,
  "results": [
    {
      "case_id": "private-data",
      "passed": false,
      "failures": [
        "Expected action=refuse, got 'answer'"
      ]
    }
  ]
}
```

Requirements:

- Include one result entry for **every case**, not just failed ones.
- Keep terminal reporting.
- Keep exit code `1` when any case fails.
- Use UTF-8 and readable indentation.
- Do not store API keys or real private employee information.

Hint:

```python
report_path.write_text(
    json.dumps(report, indent=2, ensure_ascii=False),
    encoding="utf-8",
)
```

**Acceptance criterion:** the terminal summary and JSON report show matching totals.

### Program 3 — Compare baseline and candidate results

**Goal:** detect whether a proposed change introduces regressions.

Create two synthetic fixture files:

```text
outputs_baseline.json
outputs_candidate.json
```

Evaluate both against the **same cases and evaluator**.

Classify each case:

| Baseline | Candidate | Classification |
|---|---|---|
| Pass | Pass | Unchanged pass |
| Fail | Pass | Improvement |
| Pass | Fail | Regression |
| Fail | Fail | Still failing |

Print a comparison such as:

```text
policy-grounding: REGRESSION
missing-policy: IMPROVEMENT
private-data: UNCHANGED PASS

Improvements: 1
Regressions: 1
```

For today’s practice rule, return a nonzero exit code if **any regression** occurs.

**Acceptance criterion:** deliberately introduce one improvement and one regression, and verify that both appear.

An unchanged overall score can hide a regression: one case improves while another breaks.

## Part 3 — Real-world troubleshooting assignment

A client reports:

> “The new version answers policy questions better, but it now answers questions that previously required clarification.”

Write a short investigation note covering:

1. What evidence would you collect?
2. Which baseline and candidate cases would you compare?
3. How would you distinguish an agent failure from an evaluator error?
4. What result would block your release?
5. What limitations would you disclose to the client?

Use synthetic or sanitized examples.

## Your EOD deliverable

You should have:

- Six evaluator unit tests.
- A saved JSON evaluation report.
- A baseline-versus-candidate comparison.
- One demonstrated regression and one improvement.
- A short client investigation note.

Send your quiz answers as `1.A, 2.B, ...`. Share your code or output separately for program review.
