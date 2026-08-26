# AI Data Analyst Agent

## Project Overview

AI Data Analyst Agent is a multi-agent business analytics system designed to answer business questions by routing them to specialized analytical agents. The workflow combines specialist agents for Sales, Customer, Inventory, and Marketing with analytical tools, structured outputs, aggregation, short-term conversation context, and LangSmith observability.

The complete implementation and captured execution outputs are provided in `ai_data_analyst_agent.ipynb`.

## Project Objectives

The system is designed to:

- Route a business question to the relevant specialist agents.
- Analyze Sales, Customer, Inventory, and Marketing data using dedicated tools.
- Combine specialist findings into one structured final analysis.
- Provide facts, findings, possible causes, recommendations, and confidence/limitations.
- Preserve short-term conversational context through the workflow checkpointer.
- Trace workflow and LLM activity using LangSmith.

## System Architecture

The main workflow follows this pattern:

```text
User Question
     |
     v
Question Router
     |
     +-------------------+-------------------+-------------------+
     |                   |                   |                   |
     v                   v                   v                   v
 Sales Specialist   Customer Specialist  Inventory Specialist  Marketing Specialist
     |                   |                   |                   |
     +-------------------+-------------------+-------------------+
                             |
                             v
                     Aggregator Agent
                             |
                             v
                   Structured Final Analysis
```

The final analysis is returned as structured data containing:

- `facts`
- `findings`
- `possible_causes`
- `recommendations`
- `confidence_and_limitations`

## Main Components

### Router

Determines which specialist agents are relevant to the user's question.

### Specialist Agents

The project includes dedicated specialist workflows for:

- Sales
- Customer
- Inventory
- Marketing

Each specialist uses the tools and data relevant to its domain.

### Aggregator

Combines the specialist outputs into one structured business analysis.

### Memory and Workflow State

The workflow uses a checkpointer and conversation context so that a previous question and final analysis can be injected into a subsequent turn.

### LangSmith Observability

LangSmith tracing is enabled for the workflow and LLM activity. Phase 14 includes an actual workflow execution and inspection of LangSmith traces, including successful LLM, tool, parser, and workflow runs.

## Example Result

One captured analysis correctly identified that the premise of a sales decrease was not supported by the available data. The analysis reported:

- May 2026 revenue: $281,925.52
- June 2026 revenue: $292,105.93
- May-to-June growth: +3.61%
- Repeat-customer rate: 99.33%
- Average order value: $404.34

The same analysis identified six products below their reorder thresholds and noted that month-specific marketing and customer data were not available for establishing a causal explanation.

Another captured analysis identified Campaign B as having the highest available conversion-per-spend metric at 65.55 conversions per $1,000 spent, while explicitly limiting the conclusion because monetary ROI and customer-level electronics purchasing data were unavailable.

## Observability

LangSmith was configured for the project using the project name:

```text
ai-data-analyst-agent-capstone
```

The captured LangSmith traces included successful runs for components such as:

- `aggregator_task`
- `marketing_task`
- `customer_task`
- `campaign_performance_tool`
- `ChatGroq`
- `PydanticToolsParser`
- `RunnableSequence`

The notebook also includes the trace-upload synchronization step used before querying LangSmith.

## Technologies

The project uses the technologies and libraries implemented in the notebook, including:

- Python
- LangChain
- LangGraph
- Groq
- Pydantic
- Pandas
- LangSmith
- Google Colab

## Repository Structure

```text
ai-data-analyst-agent/
├── ai_data_analyst_agent.ipynb
├── README.md
├── CAPSTONE_WRITEUP.md
├── TECHNICAL_DOCUMENTATION.md
├── SUBMISSION_CHECKLIST.md
├── requirements.txt
└── .gitignore
```

## How to Run

### 1. Open the notebook

Open:

```text
ai_data_analyst_agent.ipynb
```

in Google Colab or a compatible Jupyter environment.

### 2. Configure credentials

The notebook reads API credentials from the execution environment/Colab user data rather than storing secret keys directly in the repository.

Required services include the model provider used by the notebook and LangSmith for observability.

Do not place API keys directly inside the notebook or repository.

### 3. Install dependencies

Install the packages listed in:

```text
requirements.txt
```

### 4. Run the notebook

Run the notebook cells in order. The notebook contains the implementation, execution results, and captured outputs for the project phases.

Students: Leen Altayyar

Training program: Building AI Agent Systems

Delivered by: https://github.com/SDAIAAcademy

Trainer: Mohammed Albeladi

Cohort/session dates: August 23–27, 2026

The final analyses reflect the data available to the specialist agents. In particular, some questions cannot be answered with high confidence when month-specific or customer-level data are unavailable. The notebook records these limitations in the structured `confidence_and_limitations` output rather than presenting unsupported conclusions as facts.

## Project Documentation

- `CAPSTONE_WRITEUP.md` — section-by-section project write-up.
- `TECHNICAL_DOCUMENTATION.md` — technical implementation and architecture documentation.
- `SUBMISSION_CHECKLIST.md` — final submission checklist.
- `requirements.txt` — Python dependencies.
- `.gitignore` — excludes environment files, secrets, temporary files, and generated artifacts.

## Repository


