# AI Data Analyst Agent

## Project description

The **AI Data Analyst Agent** is a multi-agent e-commerce analytics
system built with Python, Pandas, LangChain, LangGraph, Groq, Chroma,
and LangSmith.

The system analyzes synthetic e-commerce data through four specialist
agents:

-   **Sales Agent** --- revenue, growth, products, categories, and
    regions
-   **Customer Agent** --- customer segments, repeat-purchase behavior,
    and policy questions
-   **Inventory Agent** --- low-stock and out-of-stock risk
-   **Marketing Agent** --- campaign spend, conversions, and campaign
    efficiency

A manager/router uses structured LLM output to decide which specialists
are relevant. Selected specialists then run in parallel, and an
aggregator produces a structured final analysis separating facts,
findings, hypotheses, recommendations, and limitations.

## Key capabilities

-   Real Pandas-backed analytical tools
-   Pydantic structured results
-   LLM tool calling
-   LLM-based multi-agent routing
-   LangGraph Functional API
-   Parallel specialist execution
-   Structured aggregation
-   Policy RAG
-   Short-term conversation state
-   Cross-thread long-term memory
-   Human-in-the-loop approval
-   Retry and validation/error handling
-   LangSmith observability

## Data

The notebook generates synthetic data for 30 products, 300 customers,
4,160 orders, 30 inventory records, and 30 marketing records covering
January--June 2026. The data deliberately contains discoverable signals,
including an Electronics sales dip in March, low-stock products, and an
intentionally weak Campaign D.

The notebook verifies that the datasets contain no nulls and that
order/customer/product relationships are valid.

## Architecture

``` text
User Question
      |
      v
Manager / Router
      |
      +----------+-----------+-----------+
      |          |           |           |
    Sales    Customer    Inventory   Marketing
      |          |           |           |
      +----------+-----------+-----------+
                     |
                     v
                Aggregator
                     |
                     v
             Structured Analysis
```

After routing, selected specialist tasks are launched concurrently. The
aggregator runs after their results are available.

## RAG

The Customer Agent uses a policy RAG pipeline:

``` text
Policy .txt files
      -> DirectoryLoader
      -> RecursiveCharacterTextSplitter
      -> HuggingFace embeddings
      -> Chroma
      -> Retriever
      -> policy_lookup_tool
      -> Customer Agent
```

The policy set covers returns, shipping, inventory, discounts, and
customer service.

## Memory

### Short-term memory

A LangGraph `InMemorySaver` checkpointer stores conversation state by
`thread_id`. The notebook tests two turns in the same thread and a fresh
thread without the previous context.

### Long-term memory

LangGraph `InMemoryStore` stores facts in a user namespace. The notebook
saves a fiscal-year fact and recalls it from a different thread.

`InMemoryStore` is a capstone/demo implementation; a production system
should use persistent storage.

## Human-in-the-loop

Before a reorder/purchase-order action, the workflow calls
`interrupt()`. The same thread is then resumed with
`Command(resume=...)`.

Both approval and rejection paths are demonstrated. The purchase-order
function is only a simulation.

## Error handling

Two strategies are implemented:

1.  `RetryPolicy` for transient task failures.
2.  Validation in `correlation_tool` for invalid campaign IDs, returning
    a structured error instead of silently producing an invalid
    correlation.

The retry demo intentionally fails twice and succeeds on attempt three.

## Observability

LangSmith project:

`ai-data-analyst-agent-capstone`

A real successful trace was captured after tracing was enabled. The
observed project contained successful runs including:

-   `customer_task`
-   `marketing_task`
-   `campaign_performance_tool`
-   `aggregator_task`
-   `ChatGroq`
-   `PydanticToolsParser`
-   `RunnableSequence`

The trace therefore demonstrates internal LLM/tool activity rather than
only a manually created LangSmith test run.

## Important findings from the data

The notebook's ground-truth checks show:

-   Electronics revenue: **\$44,614.97 in February 2026**
-   Electronics revenue: **\$22,850.98 in March 2026**
-   February → March Electronics growth: **-48.78%**
-   **6** products below the reorder threshold
-   **0** products completely out of stock in the inventory snapshot
-   Campaign D: **4.5 conversions per \$1,000 spend**, the weakest
    campaign by that metric
-   Campaign B spend/conversion correlation: **0.987**

The notebook's marketing metric is **conversion efficiency**, not
monetary ROI. Therefore the project does not claim that Campaign B has
the highest true financial ROI without revenue-attribution data.

## Running the project

The notebook is designed for Google Colab.

Required secrets:

``` text
GROQ_API_KEY
LANGSMITH_API_KEY
```

Run the notebook from top to bottom after configuring the secrets.

Do not put API keys directly in the notebook or commit them to Git.

## Repository structure

``` text
.
├── ai_data_analyst_agent_capstone.ipynb
├── README.md
├── CAPSTONE_WRITEUP.md
├── TECHNICAL_DOCUMENTATION.md
├── SUBMISSION_CHECKLIST.md
├── requirements.txt
├── .gitignore
├── data/
└── policies/
```

The `data/` and `policies/` directories are generated by the notebook
and can be excluded from Git if the final repository is intended to
regenerate them.

## Programme information

Training program:[بناء أنظمة وكلاء الذكاء الاصطناعي]

Cohort dates:[August 23–27, 2026]

Author:[Leen Saud Altayyar ]

SDAIA Academy GitHub: https://github.com/SDAIAAcademy

## Limitations

-   The dataset is synthetic.
-   Customer metrics are all-time aggregates and do not provide
    month-level customer behavior.
-   Inventory is a snapshot rather than a historical inventory series.
-   Marketing data supports conversions-per-\$1,000 and correlation, not
    true revenue-based ROI.
-   Long-term memory uses `InMemoryStore`.
-   The final notebook contains an older Groq 120B rate-limit error from
    testing; this is an observed limitation, not evidence that the
    analytical tools themselves failed.
