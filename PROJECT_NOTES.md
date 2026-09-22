# Project Notes — Database AI Agent

## Goal

Build and understand an AI agent that can interact with tabular data and a SQL database using natural language.

The original course environment used Azure OpenAI. The project was adapted to current tooling and a Groq-hosted model.

## What We Built

### 1. Basic LLM Call

```text
HumanMessage → LLM → response.content
```

### 2. Pandas Agent

The LLM could interpret natural-language questions about a DataFrame and decide what Pandas code to execute.

```text
LLM decides what to do
Pandas performs the calculation
```

### 3. SQL Agent

The COVID CSV was loaded into SQLite:

```text
CSV → Pandas → SQLite → all_states_history
```

The SQL agent could generate, validate, and execute SQL, then return a natural-language answer.

### 4. Function Calling

Two Python functions were exposed as tools:

```python
get_hospitalized_increase_for_state_on_date(...)
get_positive_cases_for_state_on_date(...)
```

The LLM selected the tool and arguments; Python executed the function against SQLite.

### 5. Agent Loop with Conversation State

```text
User
 ↓
LLM
 ↓
tool call?
 ↓ yes
Python function
 ↓
database
 ↓
ToolMessage
 ↓
LLM
 ↓
final answer
```

Conversation history also allowed follow-up questions without repeating all parameters.

## Important Lessons

### Agents can consume multiple API requests

```text
User request
 ↓
LLM call
 ↓
tool
 ↓
LLM call
 ↓
tool
 ↓
LLM call
```

This matters for cost, latency, rate limits, and reliability.

### Tokens and requests are different

- **Request** = one API call to the model.
- **Tokens** = chunks of text processed or generated inside those requests.

### Keep deterministic work outside the LLM

Better pattern:

```text
LLM → chooses operation
Python / SQL → computes result
LLM → explains result
```

### Tool outputs should be the source of truth

A tool returned:

```text
Texas: 0
Nationwide: 63,105
```

but the model later stated Texas was `1,234`.

This showed that a correct tool result can still be misreported by the LLM.

## Modernization Decisions

- Groq was used as the LLM provider.
- LangChain handled tool calling.
- Conversation history replaced the old thread concept.
- A custom Python loop handled tool execution.
- The older Assistants API implementation was treated as a conceptual lesson rather than copied directly.

## Planned Extension

Build a Data Quality Agent with tools for:

```text
null checks
duplicate detection
schema validation
outlier detection
missing-value handling
deduplication
standardization
quality reporting
```

Long-term architecture:

```text
User
 ↓
Data Quality Agent
 ↓
LLM decides which tools are needed
 ↓
Python / SQL performs checks or transformations
 ↓
Structured results
 ↓
LLM explains findings
```
