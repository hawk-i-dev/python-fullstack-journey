## Day 18 AI — Programs and Quiz

Today’s practice strengthens routing, planning, clarification, and safe multi-tool execution. A manager keeps final-answer ownership; a handoff transfers ownership to a specialist. [Official OpenAI orchestration guide](https://developers.openai.com/api/docs/guides/agents/orchestration)

### Program 1: Display an agent plan clearly

Create `plan_display.py`:

```python
from agent_planner import create_plan


def show_plan(question: str, **details: object) -> None:
    plan = create_plan(question=question, **details)

    print(f"\nIntent: {plan.intent}")

    if plan.clarification_needed:
        print(f"Clarification needed: {plan.clarification_needed}")
        return

    print("Plan:")
    for number, step in enumerate(plan.steps, start=1):
        print(f"{number}. Tool: {step.tool_name}")
        print(f"   Arguments: {step.arguments}")
        print(f"   Reason: {step.reason}")


show_plan(
    "Categorize this expense and check my remaining budget.",
    expense_title="Uber ride to office",
    monthly_limit=30000,
    spent=18450,
)

show_plan("What is my remaining budget?")
```

Run:

```bash
python plan_display.py
```

### Program 2: Add savings-rate routing

Add this condition to `create_plan()` in `agent_planner.py`:

```python
wants_savings_rate = any(
    word in normalized
    for word in ("savings", "save", "savings rate")
)

if wants_savings_rate:
    income = locals().get("income")

    if income is None or spent is None:
        return AgentPlan(
            intent="savings_rate",
            steps=steps,
            clarification_needed="Please provide both income and spent amounts.",
        )

    steps.append(
        PlanStep(
            tool_name="calculate_savings_rate",
            arguments={
                "income": income,
                "spent": spent,
            },
            reason="The user asked about savings or savings rate.",
        )
    )
```

Also add `income: float | None = None` to the `create_plan()` parameters.

### Program 3: Workflow acceptance test

Add to `tests/test_agent_workflow.py`:

```python
from agent_workflow import run_finance_workflow


def test_workflow_runs_category_and_budget_tools():
    result = run_finance_workflow(
        question="Categorize this expense and show remaining budget.",
        expense_title="Uber ride",
        monthly_limit=30000,
        spent=18450,
        chunks=[],
    )

    assert result["ok"] is True
    assert result["intent"] == "multi_tool"
    assert len(result["results"]) == 2
    assert result["results"][0]["tool"] == "categorize_expense"
    assert result["results"][1]["tool"] == "calculate_budget"
```

Run:

```bash
pytest
```

---

## Day 18 AI Quiz

Reply like: `1.B, 2.A, ...`

1. What is agent routing?

   A. Deleting old chat messages  
   B. Choosing the correct workflow, tool, or specialist for a request  
   C. Creating an API key  
   D. Installing Python  

2. What does an agent plan contain?

   A. Ordered steps needed to handle a request safely  
   B. Only the user password  
   C. CSS styles  
   D. Random tool names  

3. A manager agent is best when:

   A. A specialist must own the complete conversation  
   B. The manager should keep control of the final answer  
   C. No tools are available  
   D. The application has no users  

4. A handoff is best when:

   A. The manager needs a small helper result  
   B. The specialist should take over the conversation branch  
   C. You want to skip validation  
   D. You want to delete history  

5. What should happen when required budget details are missing?

   A. Invent the numbers  
   B. Run the tool with zero values  
   C. Ask a focused clarification question  
   D. Transfer money  

6. Why is a multi-tool plan useful?

   A. One request may require several safe steps  
   B. It removes testing  
   C. It makes tools unrestricted  
   D. It avoids validation  

7. In a budget-and-category request, which order is reasonable?

   A. Delete expenses, then calculate  
   B. Categorize expense and calculate budget through approved tools  
   C. Send data to an unknown service  
   D. Skip all tools  

8. What should an unsupported request return?

   A. A safe limitation message  
   B. A fake successful result  
   C. An unapproved tool call  
   D. A deleted database  

9. Why should every plan step include a reason?

   A. To make execution explainable and easier to audit  
   B. To make code slower  
   C. To hide the tool name  
   D. To avoid logs  

10. Which request should require confirmation before execution?

   A. Calculate savings rate  
   B. Categorize “Grocery shopping”  
   C. Transfer money  
   D. Search RAG documents  

11. When should you split one agent into multiple specialists?

   A. Immediately for every project  
   B. Only when tools, policy, ownership, or instructions materially differ  
   C. Never  
   D. Only when Python fails  

12. Which is an example of a bounded specialist task?

   A. Summarizing retrieved documents  
   B. Accessing every company system  
   C. Running arbitrary shell commands  
   D. Deleting user accounts  

13. Why test the planner separately from the live LLM?

   A. Planning logic should be deterministic, quick, and testable  
   B. LLMs cannot use text  
   C. Tests require API keys  
   D. JSON cannot be tested  

14. What should `intent="unsupported"` normally contain?

   A. A destructive fallback tool  
   B. A clarification or limitation message and no execution steps  
   C. An API key  
   D. A random tool request  

15. What is the safest agent design principle?

   A. Plan, validate, allowlist, execute, and report  
   B. Let the model run anything  
   C. Skip tests for speed  
   D. Always assume user details
