# MCP

YT LINK: https://www.youtube.com/watch?v=RhTiAOGwbYE

[https://www.youtube.com/watch?v=RhTiAOGwbYE&t=525s](https://www.youtube.com/watch?v=RhTiAOGwbYE&t=525s)

## MCP Tutorial (KodeKloud, Jul 2025)

## Summary & Key Concepts

**Purpose:**

- A practical, beginner-friendly walkthrough for understanding and building an MCP server & client, including hands-on labs.
- Covers background, architecture, components (resources, tools, prompts), wiring with AI agents (LLMs), and real-world applications.
- This summary also includes insights from our conversation to clarify MCP workflows, LLM roles, and practical examples.

---

## 1. **Why MCP?**

- **Problem:** LLMs (like GPT) can generate text but cannot “take action” (e.g., book flights) on their own.
- **Solution:** AI agents + Model Context Protocol (MCP) bridge LLMs with real-world APIs/services.
- **AI Agents:** Like old automation/orchestrator scripts **but smarter**; they use LLMs for flexible reasoning (e.g., what steps to run, preferences, looping logic).

---

## 2. **MCP Architecture & Workflow**

- **MCP = A standard for linking AI agents with 3rd-party tools and data.**
- **Client-Server Design:**
    - **MCP Server:** Hosts resources, tools, prompts; runs API calls on request.
    - **MCP Client:** Usually runs in your IDE (VS Code/Cursor), connects to the server via Standard IO or HTTP.
    - **LLMs live in the client**, not the server (except for strictly defined cases).
- **Communication:** Uses JSON-RPC 2.0 (structured remote procedure calls), can run locally or remotely.

---

## 3. **Main MCP Components**

- **Resources:** Reference/static data (e.g., airport lists, refund policies, guides).
- **Tools:** Discrete actions/APIs (e.g., `search_flights`, `book_flight`) defined with input/output schemas.
- **Prompts:** Templates/instructions for the LLM or agent guiding workflow (e.g., how and when to use tools/resources, formatting rules).

**Table:**

| Component | Function | Example in Flight Finder |
| --- | --- | --- |
| Resource | Static/reference info | Airport codes, refund policy |
| Tool | Action/API end-point | `search_flights`, `create_booking` |
| Prompt | AI instructions/workflow guidance | "Call search_flights with ..." |

---

## 4. **End-to-End Flight Search Example**

1. **User prompt:** “Find me flights from BLR to DEL”
2. **Agent (LLM) parses request**, consults MCP server for...:
    - Tools (e.g., `search_flights`)
    - Resources (e.g., airport codes)
    - Prompts (pre-defined instruction set)
3. **LLM uses prompt guidance** to fill out tool schema ("origin": BLR, "destination": DEL), sends request to MCP server.
4. **MCP Server executes tool**, returns results (flight list).
5. **Agent formats reply** (may enrich with resources), delivers answer.

---

## 5. **Building an MCP Server & Client**

- **MCP Servers:**
    - Can be built by anyone, as long as protocol specs are followed (listed officially/in communities).
    - Use SDKs to define components (resources, tools, prompts) as Python functions (decorators like `@mcp.resource`, `@mcp.tool`).
    - Support stateful/stateless operation; use Standard IO or HTTP transports.
- **Example:** Define resources (airport info), tools (search, booking), prompts (structured workflow hints).
- **Lab:** Follow along to configure the environment (API keys, server startup, MCP.json config, etc.)

---

## 6. **Client Features: Roots, Sampling, Elicitation**

- **Roots:** Securely expose project folders to the server as needed for certain tools.
- **Sampling:** When a server wants a specific LLM operation (like summarizing), client manages that process.
- **Elicitation:** Server can request extra info from user via client, useful for confirmation or extra details in a workflow.

---

## 7. **Important Conversation Clarifications**

- **MCP Server doesn’t run LLMs:** It is just a backend executor and registry for tools, resources, prompts.
- **Client (with LLM) does the thinking:** All reasoning/understanding comes from the client agent’s LLM, which decides which MCP tool or resource to use based on user input and server metadata.
- **Server is dumb, Client is smart:** Server only executes; the client (which runs an LLM) interprets, chooses, and makes structured calls to the server.

**Analogy:**

- MCP server is a vending machine full of labeled buttons (APIs/tools); client with LLM is the person who knows which button to press based on your spoken request.

---

## 8. **Hands-on and Labs Coverage**

- Lab setup using VS Code server and API config.
- Testing the server, running commands, validating server status (green state = ready).
- Exploring server.py (sample code on tools/resources/prompts).
- Connecting MCP server and client (MCP.json files, transport configs).
- Try/test prompts, approve tool usage, and get sample results (flight search).

---

## 9. **Best Practices, Specs, and Security**

- Always follow standard input/output schemas and resource formatting.
- Mind authentication/security for remote servers.
- Community-built servers should be used with caution (may lack QA).

---

## 10. **Useful References & Next Steps**

- MCP SDKs at [https://modelcontextprotocol.io](https://modelcontextprotocol.io/)
- Hands-on labs: [https://kode.wiki/4lFwf5p](https://kode.wiki/4lFwf5p)
- Next video topic: Building a custom AI agent using LangGraph
- Recap concepts here if needing a quick memory refresh!

---

## **Memory Jog:**

If you forget:

- MCP connects AI agents (with LLMs) to third-party APIs
- Agents use server-hosted "tools" and "resources" following prompt guidelines
- All reasoning and choices come from the client (LLM); server is only for executing defined capabilities
- Review table above for component relationships and workflow

Let me know if you need visual diagrams, a cheat sheet for server/client config, or a concept quiz to reinforce these concepts!
