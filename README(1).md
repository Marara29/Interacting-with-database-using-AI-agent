# Building a Database AI Agent with LangChain, Groq, Pandas, and SQLite

This project is a modernized version of the DeepLearning.AI course project **“Building Your Own Database Agent.”**

The original course examples used Azure OpenAI and the older Assistants API. In this version, the project was adapted to use **Groq-hosted LLMs**, modern **LangChain tool calling**, **Pandas**, and **SQLite**, while paying close attention to token usage, rate limits, and agent reliability.

## What This Project Covers

1. **Basic LLM interaction**
2. **Pandas DataFrame Agent**
3. **SQL Database Agent**
4. **Function / Tool Calling**
5. **Stateful Agent Loop**

## Architecture

```text
User
  ↓
LLM
  ↓
Tool Selection
  ↓
Python / SQL Tool
  ↓
SQLite / Pandas
  ↓
Structured Result
  ↓
LLM
  ↓
Final Answer
```

A key principle used throughout the project is:

> Use the LLM for language, reasoning, and tool selection.  
> Use deterministic Python or SQL for calculations and data operations.

## Tech Stack

- Python
- Pandas
- SQLite
- SQLAlchemy
- LangChain
- LangChain Experimental
- LangChain Groq
- Groq API
- Google Colab

## Dataset

The project uses the COVID Tracking Project `all-states-history.csv` dataset.

The CSV is loaded into a SQLite table named:

```text
all_states_history
```

## Example SQL Agent Question

```text
For October 2020, calculate the total hospitalizedIncrease
for New York and for all states combined.
```

The optimized SQL agent generated a single aggregation query and returned:

```text
New York: 0
All states: 53,485
```

## Example Function Tools

Two custom tools were created:

```python
get_hospitalized_increase_for_state_on_date(
    state_abbr,
    specific_date
)

get_positive_cases_for_state_on_date(
    state_abbr,
    specific_date
)
```

The LLM chooses the appropriate tool and arguments, Python executes the tool, SQLite returns the data, and the LLM turns that result into a concise answer.

## Token and Rate-Limit Lessons

One of the most important lessons from the project was:

```text
1 agent.invoke() != 1 LLM API request
```

An agent may internally perform multiple cycles:

```text
LLM → Tool → LLM → Tool → LLM
```

The project was optimized by:

- shortening prompts,
- removing unnecessary examples,
- avoiding repeated verification loops,
- limiting agent iterations,
- reducing `max_tokens`,
- using one SQL aggregation query when possible,
- and leaving deterministic calculations to Python/SQL.

## Reliability Lesson

A Pandas tool correctly returned:

```text
Texas: 0
Nationwide: 63,105
```

but the LLM later produced:

```text
Texas: 1,234
Nationwide: 63,105
```

This demonstrated that **tool use does not automatically eliminate hallucinations**.

For reliable AI systems, numeric outputs should come from deterministic tools and be preserved as structured results rather than recreated freely by the LLM.

## Future Direction

The next goal is to evolve this project into a **Data Quality Agent** with tools such as:

```python
check_nulls()
check_duplicates()
validate_schema()
detect_outliers()
fill_missing_values()
remove_duplicates()
standardize_columns()
```

## Key Takeaway

```text
LLM = reasoning + language + tool selection
Python / SQL = computation + validation + execution
```

That separation makes AI systems more reliable, efficient, and easier to control.
