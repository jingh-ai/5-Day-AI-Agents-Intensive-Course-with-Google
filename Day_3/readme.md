# Day 3 (Context Engineering: Sessions and Memory)

## Course Materials
+ [Context Engineering Sessions and Memory](./docs/Context%20Engineering%20Sessions%20and%20Memory.pdf)
+ [summary podcast episode](https://www.youtube.com/watch?v=FMcExVE15a4)
+ [DAY 3 Livestream](https://www.youtube.com/watch?v=8o-GXj8A3nE)

## Code
+ [Agent Sessions](./code/Day%203a%20Agent%20Sessions.ipynb)
+ [Agent Memory](./code/Day%203b%20Agent%20Memory.ipynb)

## Summary

### I. Context Engineering Overview
Context Engineering is the process of dynamically assembling and managing information within an LLM's context window to enable stateful, intelligent agents. LLMs are inherently stateless, meaning their reasoning is confined to the context window of a single API call. Context Engineering constructs a state-aware prompt by strategically selecting and injecting different types of information.

The complex payload assembled for the LLM includes:
+ Context to guide reasoning: System Instructions, Tool Definitions, and Few-Shot Examples.
+ Evidential & Factual Data: Long-Term Memory, External Knowledge (RAG), Tool Outputs, Sub-Agent Outputs, and Artifacts.
+ Immediate conversational information: Conversation History, State/Scratchpad, and the User’s Prompt.

The process manifests as a continuous cycle for each conversation turn:
1. Fetch Context: Retrieve necessary information (memories, RAG documents).
2. Prepare Context: Dynamically construct the full prompt (a blocking process).
3. Invoke LLM and Tools: Iteratively call the LLM and tools; output is appended to the context.
4. Upload Context: New information is uploaded to persistent storage, often asynchronously, allowing for background memory consolidation.

### II. Sessions
A session is a foundational element of Context Engineering and serves as the container for a single, continuous conversation. It allows the agent to maintain context and provide coherent responses within the bounds of that conversation.
Components of a Session:
+ Chronological History (Events): The turn-by-turn record, including user input, agent responses, tool calls, and tool outputs.
+ Agent’s Working Memory (State/Scratchpad): Temporary, structured data relevant to the current conversation (e.g., items in a shopping cart).

Managing Long Context and Compaction: As conversation history grows, it increases API costs, latency, and can lead to "context rot". Compaction strategies intelligently trim the history to preserve vital information while reducing the token count. Strategies include:
+ Keep the last N turns: Keeps only the most recent N turns (a "sliding window").
+ Token-Based Truncation: Includes as many messages as possible without exceeding a predefined token limit.
+ Recursive Summarization: Older conversation parts are replaced by an AI-generated summary, often run asynchronously in the background.

Production Considerations for Sessions:
+ Security and Privacy: Requires strict isolation (via ACLs) to ensure one user cannot access another user's session data. Personally Identifiable Information (PII) should be redacted before storage.
+ Data Integrity: Operations must be appended in a deterministic chronological order.
+ Performance: Session data is on the "hot path," requiring reading and writing to be extremely fast to ensure a responsive user experience.

### III. Memory
Memory is the mechanism for long-term persistence, capturing and consolidating key information across multiple sessions to provide a continuous and personalized experience. Memory is a snapshot of extracted, meaningful information, persisted across sessions.
Capabilities of Memory:
+ Personalization: Tailoring future responses based on user preferences and past interactions.
+ Context Window Management: Compacting long conversations by creating summaries and extracting key facts.
+ Agent Self-Improvement and Adaptation: Creating procedural memories about successful strategies and tool use.

Memory vs. RAG (Retrieval-Augmented Generation): 

| Feature | RAG Engines | Memory Managers | 
| :--- | :--- | :--- | 
| Primary Goal | Inject external, factual knowledge. | Create a personalized and stateful experience. | 
| Data Source | Static, pre-indexed external knowledge base (e.g., PDFs). | Dialogue between the user and agent. | 
| Isolation Level | Generally Shared. | Highly Isolated (typically per-user). | 
| Role | Expert on the world’s facts (Research Librarian). | Expert on the user (Personal Assistant). |

Types and Organization of Memory: 

| Classification | Description |
| :--- | :--- |
| Types of Information | Declarative memories ("knowing what"—facts, figures, events) and Procedural memories ("knowing how"—skills and workflows). |
| Organization Patterns | Collections (self-contained, natural language memories), Structured User Profile (core facts for quick lookups), and Rolling Summary (a single, continuously updated master document). |
| Storage Architecture | Vector databases (for semantic search of unstructured content) and Knowledge graphs (for structured, relational queries). |
| Creation Mechanisms | Explicit (user gives direct command) or Implicit (agent infers from conversation). Logic can be Internal (built into framework) or External (specialized service). |
| Memory Scope | User-Level (persists across sessions for personalization), Session-Level (for compaction of a single conversation), or Application-level (global context, often for procedural memories). |
| Multimodal | Memory is usually derived from a multimodal source (input is image/audio, memory content is text) rather than containing multimodal content itself. |

### IV. Memory Generation and Retrieval
Memory generation is an LLM-driven ETL pipeline (Extract, Transform, Load) designed to extract and condense memories.
Memory Generation Pipeline:
1. Ingestion: Client provides raw data (e.g., conversation history).
2. Extraction & Filtering: An LLM extracts meaningful content that fits predefined topic definitions, filtering signal from noise. This can be guided by JSON schemas or few-shot examples.
3. Consolidation: The most sophisticated stage; an LLM compares new information with existing memories to handle conflict resolution, deduplication, and information evolution. It may result in an UPDATE, CREATE, or DELETE/INVALIDATE operation. Consolidation also involves forgetting (proactively pruning old, stale, or low-confidence memories).
4. Storage: The new or updated memory is persisted to durable storage.
Memory Provenance and Trustworthiness: Provenance tracks a memory's origin and history to allow the agent to critically evaluate its quality. Provenance helps during consolidation for conflict resolution (prioritizing the most trusted source or most recent information). A memory’s confidence should evolve, increasing through corroboration and decreasing (decaying) over time.
Triggering Generation and Production:
+ Generation can be triggered by Session Completion, Turn Cadence, Real-Time (every turn), or Explicit Command.
+ Memory-as-a-Tool: A sophisticated approach where the agent is given a tool to autonomously decide when to generate a memory.
+ Production Requirement: Memory generation is computationally expensive and must run asynchronously as a background process to avoid blocking the user experience.
Memory Retrieval and Inference: Effective retrieval considers Relevance (semantic similarity), Recency (time-based), and Importance (significance).
+ Retrieval Timing:
    + Proactive Retrieval: Memories are automatically loaded at the start of every turn.
    + Reactive Retrieval (Memory-as-a-Tool): The agent queries its memory only when it decides context is necessary, increasing efficiency but incurring latency when used.
+ Placement in Context Window:
    + System Instructions: Memories are appended to the system prompt, giving them high authority, ideal for stable "global" information like a user profile.
    + Conversation History: Memories are injected directly into the turn-by-turn dialogue, often as tool output, but risk being mistaken for actual dialogue.

### V. Security and Evaluation
Privacy and Security Risks: Memory requires stringent security controls due to user data. Key measures include:
+ Data Isolation: Strict isolation at the user level, enforced via Access Control Lists (ACLs).
+ Sanitization: The system must validate and sanitize information (e.g., using Model Armor) before committing it to long-term memory to prevent memory poisoning (corruption by malicious user input).
+ PII Redaction: Sensitive information must be redacted before persistence.
Testing and Evaluation: Evaluation ensures the agent remembers the right things and uses them successfully. Metrics include:
+ Generation Quality: Measuring Precision (percentage of accurate/relevant generated memories) and Recall (percentage of critical facts captured).
+ Retrieval Performance: Measuring Recall@K (whether the correct memory is found in the top K results) and Latency.
+ End-to-End Task Success: Evaluating the agent’s final output using memory against a golden answer.

--------------------------------------------------------------------------------
Analogy: Context Engineering is like a concert conductor (the agent framework) preparing the music (the LLM prompt). The Session is the conductor’s immediate sheet music—the temporary, turn-by-turn script that must be read quickly and efficiently. Memory is the conductor's archived library of knowledge and experience—it contains long-term, curated records about past performances (declarative memory) and effective techniques (procedural memory), ensuring every new performance is personalized, informed, and accurate.