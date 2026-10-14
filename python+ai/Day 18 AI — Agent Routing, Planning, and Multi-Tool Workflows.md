# Day 18 AI — Agent Routing, Planning, and Multi-Tool Workflows

Today you will make your agent decide:

1. Which specialist capability is needed.
2. Which safe tools are needed.
3. In what order tools should run.
4. When it should refuse or ask for missing information.

This is called orchestration. Use one manager agent when it should own the final reply; use handoffs only when a specialist should take over the conversation. Start with one agent where possible—extra agents create extra prompts, traces, approvals, and complexity. [Official OpenAI orchestration guide](https://developers.openai.com/api/docs/guides/agents/orchestration)

## Today’s project

Build a deterministic **Finance Request Router** on top of Day 17.

```text
User request
   ↓
Router identifies intent
   ↓
Planner creates safe steps
   ↓
Executor calls approved tools
   ↓
Manager formats final result
```

No Mac/Windows code difference.

## 1. Create `agent_planner.py`

```python
from dataclasses import dataclass
from typing import Any


@dataclass(frozen=True)
class PlanStep:
    tool_name: str
    arguments: dict[str, Any]
    reason: str


@dataclass(frozen=True)
class AgentPlan:
    intent: str
    steps: list[PlanStep]
    clarification_needed: str | None = None


def create_plan(
    *,
    question: str,
    expense_title: str | None = None,
    monthly_limit: float | None = None,
    spent: float | None = None,
) -> AgentPlan:
    normalized = question.lower()

    steps: list[PlanStep] = []

    wants_category = any(
        word in normalized
        for word in ("category", "categorize", "classify")
    )

    wants_budget = any(
        word in normalized
        for word in ("budget", "remaining", "left", "afford")
    )

    wants_knowledge_search = any(
        word in normalized
        for word in ("what is", "explain", "how does", "rag", "embedding")
    )

    if wants_category:
        if not expense_title:
            return AgentPlan(
                intent="categorize_expense",
                steps=[],
                clarification_needed="Please provide an expense title to categorize.",
            )

        steps.append(
            PlanStep(
                tool_name="categorize_expense",
                arguments={"title": expense_title},
                reason="The user asked to categorize an expense.",
            )
        )

    if wants_budget:
        if monthly_limit is None or spent is None:
            return AgentPlan(
                intent="budget_check",
                steps=steps,
                clarification_needed=(
                    "Please provide both monthly_limit and spent amounts."
                ),
            )

        steps.append(
            PlanStep(
                tool_name="calculate_budget",
                arguments={
                    "monthly_limit": monthly_limit,
                    "spent": spent,
                },
                reason="The user asked about budget status.",
            )
        )

    if wants_knowledge_search:
        steps.append(
            PlanStep(
                tool_name="search_knowledge",
                arguments={"query": question},
                reason="The user asked a knowledge-base question.",
            )
        )

    if not steps:
        return AgentPlan(
            intent="unsupported",
            steps=[],
            clarification_needed=(
                "I can help with expense categories, budgets, and AI knowledge-base questions."
            ),
        )

    return AgentPlan(
        intent="multi_tool" if len(steps) > 1 else steps[0].tool_name,
        steps=steps,
    )
```

## 2. Create `agent_workflow.py`

```python
from typing import Any

from agent_planner import AgentPlan, create_plan
from agent_tools import execute_tool


def run_finance_workflow(
    *,
    question: str,
    chunks: list[dict[str, Any]],
    expense_title: str | None = None,
    monthly_limit: float | None = None,
    spent: float | None = None,
) -> dict[str, Any]:
    plan: AgentPlan = create_plan(
        question=question,
        expense_title=expense_title,
        monthly_limit=monthly_limit,
        spent=spent,
    )

    if plan.clarification_needed:
        return {
            "ok": False,
            "intent": plan.intent,
            "message": plan.clarification_needed,
            "results": [],
        }

    results = []

    for step in plan.steps:
        result = execute_tool(
            step.tool_name,
            step.arguments,
            chunks=chunks,
        )

        results.append(
            {
                "tool": step.tool_name,
                "reason": step.reason,
                "result": result,
            }
        )

    return {
        "ok": True,
        "intent": plan.intent,
        "message": "Workflow completed safely.",
        "results": results,
    }
```

## 3. Create `main_workflow_demo.py`

```python
from agent_workflow import run_finance_workflow

demo_chunks = []

result = run_finance_workflow(
    question="Categorize my expense and check my remaining budget.",
    expense_title="Uber ride to office",
    monthly_limit=30000,
    spent=18450,
    chunks=demo_chunks,
)

print(result)
```

Run:

```bash
python main_workflow_demo.py
```

Expected learning: one request can produce a plan with two approved tool calls:

1. Categorize `Uber ride to office` as Transport.
2. Calculate the remaining budget as ₹11,550.

## 4. Add tests

Create `tests/test_agent_planner.py`:

```python
from agent_planner import create_plan


def test_plan_for_category_and_budget_has_two_steps():
    plan = create_plan(
        question="Categorize this expense and show my remaining budget.",
        expense_title="Uber ride",
        monthly_limit=30000,
        spent=18450,
    )

    assert plan.clarification_needed is None
    assert len(plan.steps) == 2
    assert plan.steps[0].tool_name == "categorize_expense"
    assert plan.steps[1].tool_name == "calculate_budget"


def test_budget_plan_requests_missing_data():
    plan = create_plan(question="What is my remaining budget?")

    assert plan.clarification_needed is not None
    assert "monthly_limit" in plan.clarification_needed


def test_unsupported_request_is_not_executed():
    plan = create_plan(question="Transfer ₹5000 to my friend.")

    assert plan.intent == "unsupported"
    assert plan.steps == []
```

Run all tests:

```bash
pytest
```

## Real-world approach

| Situation | Correct design |
|---|---|
| Simple request | One agent + one tool |
| One request needs multiple steps | Manager creates an ordered plan |
| A specialist must own the reply | Use a handoff |
| High-risk action | Ask for confirmation before execution |
| Missing details | Ask a focused clarification question |
| Unknown request | Refuse safely; never guess or run a random tool |

A real finance agent must never infer bank details, make transfers, delete records, or modify data merely because a model requests it.

## Day 18 assignment

Add `calculate_savings_rate` from Day 17 into the planner.

Requirements:

- Route questions containing `savings`, `save`, or `savings rate`.
- Require `income` and `spent`.
- Return a clarification message when either amount is missing.
- Write three tests:
  - valid savings-rate plan
  - missing income
  - combined budget + savings request

By the end of Day 18, you should be able to explain: “A reliable agent is a controlled workflow—route, plan, validate, execute, and report—not an unrestricted model.”
