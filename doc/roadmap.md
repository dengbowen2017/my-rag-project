# Production Agentic RAG Roadmap

This project starts with a single LLM API call and evolves into an Agentic RAG system that keeps an arXiv paper collection up to date and answers questions with citations. The project prioritizes learning system design, architectural evolution, and engineering practices. Each stage defines planned work and acceptance criteria. Record actual results when the stage is complete.

**Start with V0.** Implement V0 through V11 in order and keep each release within its stated scope. Follow the architecture approval process in `AGENTS.md` before introducing new technology or changing the architecture, and record significant decisions in architecture decision records (ADRs). Verify the acceptance criteria before advancing.

Follow the [stage branch and release workflow](../AGENTS.md#stage-branches-and-milestones) for each version. Work on `stage-vNN`, keep the plan and stage-specific CI with that branch, and open a pull request into `main`. After the verified stage lands in `main`, tag the resulting commit `vNN` to create its GitHub Release. For example, V0 uses `stage-v00` and `v00`.

## V0 · Minimal LLM Application

**Goal** — Build a command-line question-answering application with Python and one LLM API. Trace a request from configuration through the model response.

- Set up the CLI entry point to accept a question, read the model configuration and credentials from environment variables, call the model, and print the answer.
- Report missing configuration, authentication failures, and request timeouts with actionable messages.
- **Acceptance** — Verify a successful answer with a live API call and check at least one failure case. Record live API checks and mocked test results separately.

## V1 · Minimal RAG

**Goal** — Answer questions using a small collection of local documents and cite passages the reader can locate.

- Prepare a small set of related Markdown files, plain-text documents, or paper abstracts. Use keyword matching to return the top-k passages with file and paragraph references. Make the search results viewable through the CLI.
- Pass the retrieved passages to the LLM as context and return an answer with citations. State when the available evidence is insufficient.
- Build a fixed evaluation set with questions, expected sources, and questions for which the corpus lacks evidence.
- **Acceptance** — Inspect keyword search results independently. Verify that at least one answer is supported by its cited passage and that a question without supporting evidence receives an appropriate response.

## V2 · Modular RAG

**Goal** — Understand how chunking, embeddings, and vector similarity affect retrieval, and make retrieval and generation independently verifiable.

- Separate document loading, chunking, retrieval, and answer generation into focused modules. Give each chunk a stable source identifier and a location in the original document.
- Embed document chunks and queries. Implement top-k retrieval with in-memory vectors and cosine similarity. Keep keyword search as a baseline.
- **Acceptance** — Compare keyword and vector retrieval using the same corpus and questions, including exact terms and paraphrases. Verify that both modes preserve source references through answer generation.

## V3 · Web Service

**Goal** — Make the existing RAG workflow available through an HTTP API and a browser.

- Use FastAPI to expose health, search, and question-answering endpoints. Define request, response, and error formats.
- Have Gradio call the question-answering API and display the completed answer and its citations. Reuse the existing logic in the CLI and API.
- **Acceptance** — Query the same corpus through the API and Gradio. Verify that empty questions, empty search results, and model failures produce distinguishable feedback.

## V4 · Data Persistence

**Goal** — Preserve document data across restarts and provide a durable source for later ingestion and index rebuilds.

- Store document identifiers, titles, sources, text, and chunk metadata in PostgreSQL. Make database initialization or migrations repeatable.
- Add a manual import command for the local corpus and load the persisted documents into the existing retrieval workflow.
- **Acceptance** — Ask about an imported document after restarting the application. Reimport it and verify that no duplicate records appear and its citations still locate the original text.

## V5 · Containerization and Service Orchestration

**Goal** — Make the verified application environment reproducible with Docker Compose.

- Write a Dockerfile for the application and use Docker Compose to run the web application and PostgreSQL. Configure health checks, environment variables, and persistent volumes.
- Document how to start and stop services, inspect logs, and import sample data.
- **Acceptance** — Start from a clean environment using the documented steps, import the sample data, and answer a question in the browser. Recreate the application container and verify that the stored documents remain queryable.

## V6 · OpenSearch and Hybrid Retrieval

**Goal** — Establish measurable BM25, vector, and hybrid retrieval with OpenSearch.

- Index document chunks in OpenSearch, define the index mapping, and establish a BM25 keyword baseline.
- Generate embeddings with Jina AI. Add vector search and combine keyword and vector rankings with reciprocal rank fusion (RRF). Keep all three modes available for comparison and fall back to BM25 when query embedding fails.
- **Acceptance** — Rebuild the search index from PostgreSQL. Compare all three retrieval modes with the same corpus, chunks, and evaluation questions. Record top-k hit rates and query latency, and verify the BM25 fallback.

## V7 · Automated Paper Ingestion

**Goal** — Turn the manually maintained corpus into an arXiv paper collection that updates on a schedule.

- Build a manual ingestion command that fetches arXiv metadata and abstracts and parses PDFs with Docling. Preserve the paper ID, section information, and source, then write the records to PostgreSQL and update the OpenSearch index.
- After verifying manual ingestion, schedule the same workflow in Airflow for incremental updates. Record task status and support retries.
- **Acceptance** — Follow a small set of public papers from ingestion to searchable citations. Rerun the import and verify that records are updated by paper ID and version. Verify that a failed paper can be identified and retried, and that Airflow can trigger a sync.

## V8 · Local Model and Streaming

**Goal** — Generate answers locally with Ollama and deliver incremental responses over Server-Sent Events (SSE).

- Integrate Ollama into the question-answering workflow while reusing retrieval results, prompt construction, and citations.
- Add a streaming endpoint to FastAPI and display the answer incrementally in Gradio. Handle stream completion, errors, and client disconnections.
- **Acceptance** — Complete one standard request and one streaming request with Ollama. Verify that the client receives partial output before generation completes, that the final response includes citations, and that an interrupted generation produces clear feedback.

## V9 · Performance Optimization

**Goal** — Use Redis to avoid repeating retrieval and generation for identical requests and measure the effect.

- Record latency and model call counts for repeated requests before adding caching. Start with exact-match caching for standard question answering.
- Include inputs that affect the answer, such as the model, retrieval parameters, and index version, in the cache key. Set an expiration period and allow requests to run normally when Redis is unavailable.
- **Acceptance** — Record latency and model call counts for a cache miss and a cache hit under reproducible conditions. Verify that an index update prevents reuse of stale answers and that question answering still works when Redis is unavailable.

## V10 · AI Observability

**Goal** — Trace a question-answering request to see which steps ran, how long they took, and where a failure occurred.

- Deploy Langfuse and the services required by the chosen deployment. Trace requests, cache lookups, embedding, retrieval, and generation.
- Record the model, retrieval mode, returned source identifiers, and available usage data. Control what inputs and outputs are captured and redact sensitive data as needed.
- **Acceptance** — Locate a standard request, a cache hit, and a failed request in Langfuse. Verify that the traces show the steps that ran and identify the failure location.

## V11 · Agentic RAG

**Goal** — Let the system decide when the evidence warrants another retrieval attempt and make the resulting workflow available through Telegram.

- Use LangGraph to orchestrate scope checks, retrieval, document relevance grading, query rewriting, and answer generation. Limit retrieval retries, stop when evidence remains insufficient, and record the outcome of each decision node.
- Verify the agent through the existing API or Gradio before connecting the Telegram bot to the same agent API. Display citations and handle request timeouts in Telegram.
- **Acceptance** — Verify four paths covering direct retrieval success, success after a rewrite, insufficient evidence after retrying, and an out-of-scope question. Compare standard RAG and agentic RAG on source hit rate, citation quality, model call counts, and latency using the fixed evaluation set. Verify that Telegram returns a cited answer through the shared backend.

## End-to-End Acceptance

- Start the services from a clean environment using the documented steps. Complete one cycle of paper ingestion, parsing, persistence, and index updates.
- Ask questions through Gradio and Telegram and use the returned citations to locate the supporting paper passages.
- Inspect request traces, cache activity, and agent decisions in Langfuse. Use the fixed evaluation set to recheck retrieval and answer quality.
