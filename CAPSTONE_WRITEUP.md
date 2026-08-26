# Capstone Write-Up

## 1. Agent fundamentals --- 15 points

The project uses real Pandas-backed analytical functions and exposes
them as LangChain tools rather than hardcoded answers. The tools
calculate revenue, sales growth, product performance, category/region
sales, customer metrics, inventory risks, campaign performance, and
campaign correlation. Results are represented with Pydantic models
before being exposed to the agents. The notebook's ground-truth checks
passed and verified a February-to-March Electronics revenue decline of
48.78%, six low-stock products, Campaign D as the weakest campaign by
conversions per \$1,000 spend, and a 0.987 Campaign B spend/conversion
correlation.

## 2. Multi-agent / routing architecture --- 15 points

The project uses an LLM-based manager/router with a Pydantic
`RoutingDecision` schema. The router decides whether Sales, Customer,
Inventory, and/or Marketing specialists are needed, and it can select
multiple specialists for a broad question. This pattern was chosen so
routing is semantic and model-driven rather than implemented as keyword
matching.

## 3. RAG pipeline --- 15 points

The Customer Agent uses a tool-based policy RAG pipeline. Five policy
documents are created, loaded with `DirectoryLoader`, split with
`RecursiveCharacterTextSplitter`, embedded with
`sentence-transformers/all-MiniLM-L6-v2`, stored in Chroma, and queried
through a retriever. The retriever is exposed as `policy_lookup_tool`
and added to the Customer Agent's real tool set, so retrieval is part of
the agent workflow rather than a disconnected demonstration.
### RAG Pattern Choice

This project uses a 2-Step RAG pattern because the policy lookup task is a relatively simple factual retrieval problem. The workflow retrieves relevant policy information first and then uses the retrieved context to answer the question. A multi-step Agentic RAG approach was not necessary because the task does not require iterative retrieval, complex reasoning across multiple sources, or dynamic tool selection. A Hybrid RAG approach was also unnecessary because the available use case can be handled effectively with a straightforward retrieve-then-use flow.

## 4. Context & state management --- 15 points

Short-term context uses the LangGraph checkpointer with a `thread_id`;
the notebook defines two turns on one thread and a separate fresh thread
to demonstrate the boundary. Long-term memory uses LangGraph
`InMemoryStore` in a user namespace. A fiscal-year fact is saved and
then recalled from a different thread. The implementation therefore
separates conversation state from cross-thread memory, while the README
notes that the in-memory store is appropriate for the capstone
demonstration rather than production persistence.

## 5. Human-in-the-loop --- 10 points

The reorder workflow uses LangGraph `interrupt()` before the simulated
purchase-order action. The paused workflow is resumed on the same thread
with `Command(resume=...)`. The notebook includes both an approval path,
where the reorder action is executed, and a rejection path, where the
reorder is skipped. This pattern was chosen because submitting a
purchase order is an external business action that should require
explicit human approval.

## 6. LangGraph Functional API & error handling --- 15 points

The workflow uses LangGraph's Functional API through `@task` and
`@entrypoint`. For error handling, specialist and aggregator tasks are
assigned a `RetryPolicy`, while `correlation_tool` validates campaign
IDs and returns a structured error for invalid input. The notebook also
contains a deterministic retry demonstration in which a task fails twice
and succeeds on the third attempt. These two strategies cover transient
failures and bad input separately.

## 7. Workflow pattern --- 10 points

The selected workflow pattern is **Parallelization**. The manager must
first finish routing because the system needs to know which specialists
are relevant. After routing, selected specialist tasks are launched
through `@task` without immediately waiting for each result. The
workflow then collects the futures and sends the combined specialist
outputs to the aggregator. Parallelization was selected because the
specialist analyses are independent once routing is known.

## 8. LangSmith observability --- 5 points

LangSmith tracing was enabled for the `ai-data-analyst-agent-capstone`
project and a real workflow was executed after tracing was configured.
The resulting project showed successful `customer_task`,
`marketing_task`, `campaign_performance_tool`, `aggregator_task`,
`ChatGroq`, `PydanticToolsParser`, and `RunnableSequence` runs. The
useful observation was not merely that tracing existed: the trace
exposed the internal specialist, tool, LLM, parser, and aggregation
stages. The trace also supports the limitation that the marketing answer
is based on conversion efficiency rather than verified monetary ROI,
while customer-level category behavior was not available.

## Honest limitations

The notebook also contains a recorded Groq 120B `429 RateLimitError`
from an earlier specialist-agent test. That failure is retained as an
observed testing limitation rather than being described as a successful
run. The final LangSmith trace was subsequently captured successfully
using the configured project, and the notebook's deterministic Pandas
tool sanity checks passed independently of the LLM rate-limit incident.
