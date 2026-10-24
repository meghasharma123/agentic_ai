# Agentic AI

## What is AI agent ?

### LLM
A normal LLM can generate an explanation based on the information in its context.

### AI agent
An agent combines an LLM with tools, control flow, and possibly memory to work toward a goal.

## The AI agent execution loop

1. Understand the request

2. Decide the next step

3. Execute an action

4. Observe the result

5. Finish or repeat

This is often summarized as `reason → act → observe → repeat`. The `ReAct` pattern is a common approach that combines reasoning with actions and observations.

## Tool calling — how an agent interacts with the real world

Tool calling allows an LLM to request that your application execute a specific function.

The model requests the action; the application controls whether and how that action executes. The LLM should not be trusted to authorize itself.

q. Imagine an Intuit agent has a tool named issue_refund(order_id, amount). The LLM selects this tool and requests a refund of ₹10,000, but the user is only authorized to receive ₹2,000.

a. The LLM may propose a refund, but the trusted backend must enforce authorization, refund limits, and business rules before executing it. I would validate the request, reject unauthorized operations, and maintain an audit trail. I would also use idempotency controls to prevent duplicate refunds during retries.

## MCP — Model Context Protocol

MCP provides a standardized protocol for AI applications to connect to external tools and context sources.

- MCP Host: The AI application that manages the interaction with the model and MCP connections.

- MCP Client: A component within the host that communicates with a particular MCP server.

- MCP Server: Exposes tools, resources, and prompts through the MCP protocol.

MCP standardizes how AI applications discover and communicate with external capabilities. If an enterprise exposes order lookup through an MCP server, another compatible AI application can reuse that integration without building a completely custom communication protocol. Authentication, authorization, and business-rule enforcement are still necessary.

### Ordinary tool calling
- A model API supports requesting a function call.
- Your application may define and implement its own tool integration.
- Your application executes the function and returns its result.
- Tool definitions and integrations may be custom to the application.

### MCP
- A protocol standardizes communication with external capability providers.
- An MCP server exposes capabilities through a common interface.
- The MCP client communicates with the server, which handles the requested capability.
- Compatible applications can reuse an MCP integration, subject to configuration and permissions.

**MCP does not automatically guarantee secure access. Authentication, authorization, input validation, and backend business rules still need to be enforced.**

## RAG and embeddings

An `embedding` converts text into a list of numbers called a vector. That vector represents aspects of the text's meaning in a mathematical space. `semantic similarity`

RAG stands for `Retrieval-Augmented Generation`

1. Company documents
Policies, support guides, product documentation

2. Chunk the documents
Split long documents into manageable passages

3. Create embeddings and index them
Store vectors alongside the text and useful metadata


When a user asks a question:
4. Embed the question and retrieve relevant passages

5. Give the question and retrieved passages to the LLM

6. Generate a grounded answer
Ideally, with supporting citations and clear handling of missing information

* The document-processing steps generally happen ahead of the user's question. Retrieval and answer generation happen when the question arrives.

### Concept	Purpose
- Embeddings: Represent text numerically for similarity-based retrieval
- Vector database/index: Stores and searches vectors, often alongside text and metadata
- RAG: Retrieves external information and supplies it to an LLM to generate an answer
- Fine-tuning: Changes model behavior through additional training; it is not the same as retrieving current documents

### Chunking

- Better precision: Retrieve the relevant section instead of the entire document.
- Lower context cost: Send fewer irrelevant tokens to the LLM.
- More useful answers: Keep retrieved content focused on the question.
- Less risk of losing context: Chunk boundaries must be chosen carefully so important rules and exceptions aren't separated.

**There is a trade-off: chunks that are too small may lose context; chunks that are too large may contain too much unrelated information.**

Common strategies include:
- `Fixed-size chunking`: Split by a target token count.
- `Structure-based chunking`: Split by headings, paragraphs or document sections.
- `Overlapping chunks`: Repeat a small amount of text between adjacent chunks to preserve context across boundaries.

* If chunks are too small or split at inappropriate boundaries, the retriever may return a rule without its related exception. The LLM could then generate an incomplete or misleading answer. I would use structure-aware chunking, appropriate chunk sizes, and possibly overlapping chunks or parent-document retrieval to preserve context.

### Vector search vs. hybrid search

- Vector search uses embeddings to retrieve passages based on semantic similarity.
- Keyword search looks for matching words, phrases, or identifiers.
- Hybrid search combines keyword search and vector search

`authorization filtering is separate from relevance ranking`. The system must not return documents the employee isn't allowed to access, even if those documents are highly relevant.

### Reranking

I would add a reranking stage after initial retrieval. The retriever finds candidate passages, and the reranker scores them against the user's query to prioritize the most relevant evidence before passing it to the LLM. This improves the chance that the correct policy appears in the final context.

**Remember: reranking can improve the ordering of retrieved passages, but it cannot recover a relevant passage that the initial retriever never returned.**

### Query rewriting

- A query rewriting component transforms the original question into a clearer search query, using available conversation context where appropriate.
- I would use conversation history and available context to resolve references and rewrite ambiguous queries when appropriate. If essential information remains unavailable, I would ask a clarifying question rather than make unsupported assumptions. This improves retrieval quality and reduces the risk of misleading answers.

### Grounding and citations

- Grounding means the generated answer is supported by the information provided to the model, such as retrieved policy passages.
- Citations help people verify answers, but a citation alone doesn't guarantee correctness. A model could cite a real document that doesn't actually support its claim.

- Citations alone do not guarantee correctness. I would verify that each important claim is supported by the retrieved evidence and check for contradictions. If the evidence is insufficient, the assistant should abstain or retrieve additional information rather than generate an unsupported answer.

### Access control and metadata filtering

- Metadata filtering: Uses attributes such as department, document version, tenant or access classification to narrow retrieval.

Topics remaining to cover:

A. RAG — Retrieval-Augmented Generation
Continue here
- Retrieval evaluation: Document freshness, Recall@K, Precision@K, MRR, nDCG
- Answer quality, grounding, and citation evaluation
- Access control and document freshness
- Hybrid search, reranking, query rewriting
- Chunking, embeddings, vector databases
- Advanced RAG: Graph RAG and agentic RAG

B. Agent orchestration and memory
- ReAct: reason, act, observe, repeat
- Plan-and-execute agents
- Supervisor/worker and multi-agent systems
- Graph-based state machines and reflection
- Short-term state, long-term memory, context management
- Retries, timeouts, fallbacks, idempotency
- Human approval for sensitive actions

C. AWS and Intuit AI platforms
- Amazon Bedrock and foundation models
- Strands Agents SDK
- Amazon Bedrock AgentCore
- Intuit GenOS: GenBuilder/GenStudio, GenRuntime, GenSRF, GenUX
- IAM permissions and least-privilege access
- Model selection, Knowledge Bases, Guardrails

D. LLM engineering
- Prompt design and few-shot examples
- Structured JSON output and schema validation
- Token limits, context windows and token budgets
- Model routing, caching and streaming
- Rate limits and cost/latency optimization

E. Evaluation and AI security
- Golden evaluation datasets
- LLM-as-a-judge and automated evaluation
- Tool-call accuracy, task success and groundedness
- Prompt injection, jailbreaks and data leakage
- PII protection, audit logs and excessive agency
- Human oversight and safe failure handling

F. Production engineering
- Python async/await, typing, Pydantic and pytest
- FastAPI and REST APIs
- OAuth2/JWT and authentication propagation
- Docker, containers and Kubernetes fundamentals
- CI/CD, rollout and rollback
- OpenTelemetry traces, metrics and logs
- Observability, token cost, latency and reliability

G. Senior-level interview practice
- Agent architecture design
- RAG system design
- Tool security and authorization
- Failure scenarios and trade-offs
- Production incident debugging
- System design using AWS and agent frameworks