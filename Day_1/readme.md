# Day 1 (Introduction to Agents)

## Course Materials
+ [Introduction to Agents](./resources/Introduction%20to%20Agents.pdf)
+ [summary podcast episode](https://www.youtube.com/watch?v=zTxvGzpfF-g)
+ [DAY 1 Livestream](https://www.youtube.com/live/ZaUcqznlhv8?si=_miqMKJtzUbK3o4l)

## Code
+ [From Prompt to Action](./code/Day%201A%20From%20Prompt%20to%20Action.ipynb)
+ [Agent Architectures](./code/Day%201b%20Agent%20Architectures.ipynb)

## Summary

The field of artificial intelligence is experiencing a paradigm shift from models focused on passive, discrete tasks to a new class of software capable of autonomous problem-solving and task execution. Agents are the natural evolution of Language Models (LMs), made useful in software.
An AI Agent is defined as the combination of models, tools, an orchestration layer, and runtime services which uses the LM in a loop to accomplish a goal.

### I. Core Agent Architecture
An agent is deconstructed into three essential components, plus a deployment layer:
1. Model (The "Brain"): The core Language Model (LM) or foundation model that serves as the agent's central reasoning engine. Its role is to process information, evaluate options, and make decisions. Architectural choices include using a "team of specialists," where a powerful frontier model handles initial planning, while faster, more cost-effective models manage simpler, high-volume tasks (Model Routing).
2. Tools (The "Hands"): Mechanisms that connect the agent's reasoning to the outside world, enabling actions beyond text generation.
    + Retrieving Information: Tools like Retrieval-Augmented Generation (RAG) and Natural Language to SQL (NL2SQL) allow the agent to access up-to-date, factual information, grounding it in reality and reducing hallucinations.
    + Executing Actions: Wrapping existing APIs or code functions allows the agent to actively change the world, such as scheduling a meeting or updating a customer record. Tools may also include a Human in the Loop (HITL) function to request confirmation or input from a user.
    + Function Calling: Standards like OpenAPI specification or the Model Context Protocol (MCP) provide the structured contract needed for the model to reliably invoke and interpret tool responses.
3. Orchestration Layer (The "Nervous System"): The governing process that manages the agent's operational loop. It is the conductor that decides when the model should reason, which tool should act, and how the results should inform the next step. This layer implements planning, memory (state), and reasoning strategy execution using techniques like Chain-of-Thought.
4. Deployment (The "Body and Legs"): Involves hosting the agent on a secure, scalable server and integrating it with production services for monitoring, logging, and management, often using specialized platforms like Vertex AI Agent Engine.

### II. The Agentic Problem-Solving Process
An agent operates on a continuous, cyclical process to achieve objectives, often described as a "Think, Act, Observe" loop. This process involves five fundamental steps:
1. Get the Mission: The process starts with a high-level goal, provided by a user or an automated trigger.
2. Scan the Scene: The orchestration layer perceives the environment to gather context, accessing short-term memory, long-term memory (via tools), and available resources.
3. Think It Through: Driven by the reasoning model, the agent analyzes the Mission and the Scene to devise a plan, often requiring a chain of reasoning.
4. Take Action: The orchestration layer executes the first concrete step of the plan by invoking the appropriate tool (calling an API, running a script).
5. Observe and Iterate: The agent observes the outcome of the action, adds the new information to its context/memory, and the loop repeats, returning to Step 3 until the initial Mission is achieved.

### III. Taxonomy of Agentic Systems (Levels of Capability)
Agentic systems are classified into levels of increasing complexity:
| Level    | Name | Core Capability |
| :--- | :---- | :---- |
| Level 0  | Core Reasoning System | Language Model operating in isolation, relying only on pre-trained knowledge; functionally "blind" to real-time events. |
| Level 1  | Connected Problem-Solver | Reasoning engine connected to external tools, allowing interaction with the world (e.g., using a search API for real-time data). |
| Level 2  | Strategic Problem-Solver | Ability to strategically plan complex, multi-part goals, leveraging context engineering to curate relevant information for each step. |
| Level 3  | Collaborative Multi-Agent System | Shifts to a "team of specialists" approach where agents delegate missions to other specialized agents, treating them as tools. |
| Level 4  | Self-Evolving System | Ability to identify gaps in its own capabilities and dynamically create new tools or new agents to fill them. |

### IV. Agent Ops and Production Management
Agent Ops is the structured operational discipline tailored for managing stochastic (probabilistic) agentic systems, serving as an evolution of DevOps and MLOps.
+ Quality Evaluation: Since traditional pass/fail tests are insufficient, quality is assessed using an "LM as Judge" which evaluates the agent’s output against a predefined rubric and golden datasets.
+ Debugging: When issues arise, OpenTelemetry traces provide a high-fidelity, step-by-step recording of the agent’s execution path (trajectory), exposing the model’s internal reasoning, tool selection, and parameters used.
+ Feedback Loop: Human feedback (e.g., bug reports) is cherished as a valuable resource; effective Agent Ops capture this data, replicate the issue, and convert the scenario into a new, permanent test case for evaluation.

### V. Security, Governance, and Interoperability
Scaling agents requires robust frameworks for security and governance.
+ Security: Requires a defense-in-depth approach, combining deterministic guardrails (hardcoded rules) with reasoning-based defenses (AI guard models) to protect against threats like prompt injection.
+ Agent Identity: Agents are a new class of principal requiring their own secure, verifiable "digital passport" (e.g., using SPIFFE), separate from the user or developer, allowing for least-privilege permissions and granular access control.
+ Governance: To manage "agent sprawl" in an enterprise fleet, a centralized control plane (gateway) is needed. This gateway enforces runtime policies (authentication and authorization) and connects to a central registry (an enterprise app store) for governance and inventory management.
+ Agent Interoperability (A2A): The Agent2Agent (A2A) protocol is the open standard enabling agents to connect and collaborate. Agents publish an Agent Card (a JSON file) to advertise their capabilities and network endpoint, enabling standardized discovery and task-oriented communication.

### VI. Agent Evolution and Learning
Agents must adapt to dynamic environments to prevent performance degradation ("aging"). Learning is fueled by:
+ Runtime Experience: Including session logs, traces, and critical Human-in-the-Loop (HITL) authoritative corrections.
+ External Signals: Such as updated enterprise policies or critiques from other agents.
Adaptation methods include continuous refinement of prompts and context engineering, and tool optimization or autonomous tool creation. For advanced optimization, an off-production platform called Agent Gym is used for simulation, trial-and-error, and dynamic evaluation.

### VII. Examples of Advanced Agents
+ Google Co-Scientist: Functions as a virtual research collaborator, designed to accelerate scientific discovery. It operates by having a Supervisor agent delegate tasks and resources to an ecosystem of specialized agents, generating and evaluating novel hypotheses.
+ AlphaEvolve Agent: Discovers and optimizes algorithms for complex problems. It uses an evolutionary process by combining code generation from Language Models with automated evaluation, where the most promising solutions inspire the next generation of code.
