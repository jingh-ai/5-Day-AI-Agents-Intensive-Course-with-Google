# Day 2 (Agent Tools & Interoperability with Model Context Protocol (MCP))

## Course Materials
+ [Agent Tools MCP](./docs/Agent%20Tools%20MCP.pdf)
+ [summary podcast episode](https://www.youtube.com/watch?v=Cr4NA6rxHAM)
+ [DAY 2 Livestream](https://www.youtube.com/live/8Gk1BE3uYek)

## Code
+ [Agent Tools](./code/Day%202A%20Agent%20Tools.ipynb)
+ [Agent Tools Best Practices](./code/Day%202b%20Agent%20Tools%20Best%20Practices.ipynb)

## Summary

### I. Models, Tools, and Agents
**Role of Tools:**

+ Foundation models, on their own, are limited to generating content based on training data; they cannot access new data, interact with external systems, or influence their environment.
+ Tools act as the agent's "eyes" and "hands," enabling the AI system to perceive and act on the external world.
+ Tools generally allow a model to know something (retrieve data from structured or unstructured sources) or do something (perform an action, often by calling an external API).
+ AI Agents use a foundation model's reasoning capability to achieve goals, making external tools essential for their capacity to take action.

**Types of Tools:**

+ A tool is defined like a function in a non-AI program, declaring a contract that includes a clear name, parameters, and a natural language description.
+ Function Tools are external functions developers define that the model can call; the definition is provided to the model in the request context.
+ Built-in Tools have their definitions implicitly given to the model behind the scenes of the model service (e.g., Google's Gemini API features like Grounding with Google Search).
+ Agent Tools allow one agent to be invoked as a tool by a primary agent, preventing a full conversation handoff and maintaining control.
+ Tools are commonly categorized by function: Information Retrieval, Action/Execution, System/API Integration, and Human-in-the-Loop.

### II. Best Practices for Tool Design
Effective tool design is crucial for reliable agentic systems.

| Design Area    | Best Practice | Rationale / Detail |
| :--- | :---- | :---- |
| Documentation  | Use clear, descriptive names. Describe all input and output parameters clearly. Clarify descriptions using simple terminology and avoid jargon. | Documentation (name, description, attributes) is passed to the model to help it use the tool correctly. |
| Instruction  | Describe actions, not implementations. Say "create a bug to describe the issue" instead of "use the create_bug tool". | This avoids confusing the LLM and prevents dependencies between system instructions and tool implementation. |
| Functionality  | Publish tasks, not API calls. Tools should encapsulate a specific task the agent performs, rather than being thin wrappers over complex Enterprise APIs. | Tools should be as granular as possible and limited to a single function with a clear purpose. |
| Output  | Design for concise output. Avoid returning large data tables, downloaded files, or generated images, as these consume the LLM's context window and affect performance/cost. | Use external systems (like temporary databases or artifact services) for large data storage, returning a reference instead of the data itself. |
| Validation  | Use optional schema validation for tool inputs and outputs. This serves as both documentation for the LLM and a run-time check on tool operation. |  |
| Error Handling  | Provide descriptive error messages. Error responses should instruct the calling LLM on what to do to address the specific error, rather than just returning an error code. |  |

### III. Understanding the Model Context Protocol (MCP)
**Standardization and Architecture:**

+ MCP, introduced by Anthropic in 2024, is an open standard designed to address the "N x M" integration problem, where custom connections grow exponentially with each new model (N) or tool (M).
+ It aims to create a unified, plug-and-play protocol that decouples the AI agent from the specific implementation details of the tools.
+ MCP uses a client-server model inspired by the Language Server Protocol (LSP).

**Core Architectural Components:**
+ MCP Host: Manages individual MCP clients, orchestrates tool use, manages the user experience, and enforces security policies.
+ MCP Client: A software component within the Host that maintains the connection, issues commands, and manages the communication session with the Server.
+ MCP Server: A program providing capabilities (tools/data) to AI applications, often acting as an adapter for an external system. Responsibilities include tool discovery, executing commands, and returning results.

**Communication Layer:**
+ The base protocol uses JSON-RPC 2.0 for a lightweight, text-based, and language-agnostic structure.
+ Communication flows through four fundamental message types: Requests, Results, Errors, and Notifications.
+ MCP supports two transport protocols: stdio (for local environments where the server runs as a subprocess) and Streamable HTTP (the recommended remote protocol).

**Key Primitives (Capabilities):**
+ Tools are the core driver of MCP value and have the broadest adoption. The Tool entity is a standardized way for a server to describe an available function.
    + Tool definitions must conform to a JSON schema including name, description, and inputSchema. outputSchema is critical for client validation and LLM instruction.
+ Other Server-Side Capabilities: Resources (contextual data like log files or database schemas) and Prompts (reusable prompt examples/templates).
+ Client-Side Capabilities: Sampling (server requests an LLM completion from the client), Elicitation (server requests additional user information from the client's UI), and Roots (defines filesystem boundaries where servers can operate, currently file: URIs). Adoption for client-side capabilities is significantly lower than for Tools.

### IV. Critical Risks and Security Challenges
While MCP accelerates development, its decentralized origins create enterprise readiness gaps, particularly in security, identity management, and observability. Securing MCP requires addressing both traditional API vulnerabilities and new threat vectors unique to agent protocols.
Top Security Risks and Mitigations:

| Risk    | Description | Mitigations |
| :--- | :---- | :---- |
| Dynamic Capability Injection | MCP servers can dynamically change the list of tools offered without client notification, potentially introducing unapproved or dangerous capabilities. | Implement client-side explicit allowlists of permitted tools. Require mandatory change notification from servers. Use Secure API/Agent Gateways (like Apigee) to inspect and filter the list of tools returned to the client. |
| Tool Shadowing | Malicious tools can use overly broad descriptions to overshadow legitimate, trusted tools, tricking the agent into choosing the rogue tool to intercept or exfiltrate data. | Prevent naming collisions (potentially using LLM-based semantic checks). Require Human-in-the-Loop (HIL) for all high-risk operations (e.g., file deletion, modification of production data). Restrict access to unauthorized MCP servers. |
| Malicious Tool Definitions / Consumed Contents | Tool descriptors or external content ingested by tools (Resources) can contain injectable prompts, leading to agent manipulation, or return values may leak sensitive data. | Input Validation (sanitizing user inputs) and Output Sanitization (filtering sensitive data like API tokens or credit card numbers, and active content like HTML) before returning results to the LLM. Strict allowlist validation of MCP resources. |
| Sensitive Information Leaks | Sensitive information stored in the conversation context can be unintentionally transmitted to agent tools that are unauthorized to access it. | Use custom annotations on tool inputs and outputs to identify, track, and control the flow of sensitive data. Implement Taint Sources/Sinks to tag data that originates from untrusted sources (tainted). |
| Lack of Scope Limiting | The MCP protocol only supports coarse-grained client-server authorization, lacking native support for per-tool or per-resource authorization. | Use scoped credentials with short expiration periods. Adhere to the principle of least privilege. Transmit secrets and credentials through a secure side channel, keeping them out of the agent context. |
| Confused Deputy Problem | A classic security vulnerability where the privileged MCP server (the "deputy") is tricked by an attacker's prompt injection on the AI model into misusing its authority and performing an unauthorized action (e.g., exfiltrating sensitive code). |   |

Conclusion: MCP establishes a critical standard for tool interoperability, but enterprise adoption requires organizations to implement external layers of centralized governance, control, and multi-layered defenses to manage the security risks inherent in its decentralized design.

--------------------------------------------------------------------------------
Analogy: Think of the Model Context Protocol (MCP) as the standardized electrical outlet (the standard interface) that allows any specialized appliance (the tool) to plug into the main power source (the foundation model/agent). However, because MCP was designed for flexibility (allowing any new appliance), the enterprise must install a custom, heavily guarded circuit breaker panel (centralized governance and API gateways) to ensure only approved, safe appliances are connected and that they can't short-circuit the system or steal power (data exfiltration or unauthorized actions).