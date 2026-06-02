# Langgraph Flow

Here's a comprehensive breakdown of LangGraph, grounded in cerebrum's actual implementation, then expanded to industry-relevant features.

---

## 1. The Big Picture: What Problem Does LangGraph Solve?

You already know the **legacy approach** in cerebrum — a manual `while` loop that calls Gemini, checks for tool calls, executes them, and loops back:

```
    while round_count < max_tool_rounds:
        content, tool_calls, model_parts = await llm_client.generate_chat_completion_single_turn(
            session_id, llm_config, system_instructions, message_context,
            function_registry=function_registry, tool_definitions=None,
        )
        # ... append assistant message ...
        if not tool_calls:
            break
        # ... manually iterate tool_calls, parse args, call functions, handle errors ...
        round_count += 1
```

**The problem:** This imperative loop works, but as flows get complex (branching logic, parallel tool calls, human-in-the-loop, retries, multi-agent routing), the `while` loop becomes spaghetti. LangGraph replaces this with a **declarative graph** where flow control is visible as structure.

Think of it like this:

```
LEGACY (imperative)              LANGGRAPH (declarative)
┌──────────────────┐            ┌──────────────────────────────┐
│  while True:     │            │                              │
│    call LLM      │            │  init_context ──► add_message│
│    if tools:     │            │       │                      │
│      run tools   │            │  evaluate_templates ──► llm  │
│    else:         │            │       │            ▼         │
│      break       │            │       │     ┌─► tools ──┐   │
│                  │            │       │     │            │   │
│  (all flow is    │            │       ▼     └──── llm ◄─┘   │
│   hidden in code)│            │      END                     │
└──────────────────┘            │  (flow IS the structure)     │
                                └──────────────────────────────┘
```

---

## 2. Core LangGraph Concepts (via Cerebrum's Code)

### Concept A: StateGraph — The Graph Container

Everything starts with a `StateGraph` parameterized by a state schema:

```
    workflow = StateGraph(ChatState)
```

A `StateGraph` is a **directed graph** where:

- **Nodes** = functions that read state and return partial updates
- **Edges** = connections defining execution order
- **State** = a shared TypedDict that flows through every node

### Concept B: State — The Shared Memory

The state is the single source of truth flowing through the graph. In cerebrum:

```
class ChatState(TypedDict, total=False):
    """State for the chat agent graph. All keys optional for partial updates."""
    message_context: List[Dict[str, Any]]
    conversation_history: List[Dict[str, Any]]
    user_exists: bool
    provider_client: Any
    session_id: str
    project_id: str
    project_settings: Dict[str, Any]
    llm_config: Dict[str, Any]
    system_instructions: str
    llm_client: Any
    langgraph_tools: Any
    tool_definitions: Any
    # ... more fields ...
    last_llm_content: Optional[str]
    last_tool_calls: Optional[List[Dict[str, Any]]]
    tool_calls_executed: int
```

**Key mental model:** Each node receives the *full* state and returns a *partial* dict. LangGraph merges the partial update into the existing state automatically.

```
Node receives:  { session_id: "abc", message_context: [...], llm_client: <obj>, ... }
Node returns:   { message_context: [... updated ...], user_exists: True }
                  ↑ only the changed keys
LangGraph:      merges returned keys into existing state → next node gets updated state
```

For example, `init_context_node` returns only the fields it changes:

```
    return {
        "message_context": message_context,
        "conversation_history": list(conversation_history),
        "user_exists": user_exists,
        "provider_client": provider_client,
    }
```

### Concept C: Nodes — The Processing Steps

Nodes are just async functions `(state) -> partial_state_update`. Cerebrum defines 5:

```
┌─────────────────┐    ┌─────────────────┐    ┌──────────────────────┐
│  init_context    │───►│  add_message     │───►│  evaluate_templates  │
│                  │    │                  │    │                      │
│ Load history     │    │ Add RAG results  │    │ Eval Jinja templates │
│ Init provider    │    │ Append user msg  │    │ Build tools          │
│ Build context    │    │                  │    │ Set system prompt     │
└─────────────────┘    └─────────────────┘    └──────────┬───────────┘
                                                         │
                                                         ▼
                            ┌──────────────────────────────┐
                            │          llm                  │
                            │                               │
                            │ Call Gemini with tools         │
                            │ Store content + tool_calls    │
                            └──────────┬───────────────────┘
                                       │
                              ┌────────┴────────┐
                              │  conditional     │
                              │  _route_after_llm│
                              ├────────┬────────┤
                         tool_calls?   │   no tool_calls?
                              │        │        │
                              ▼        │        ▼
                         ┌────────┐    │    ┌──────┐
                         │ tools  │    │    │ END  │
                         │        │────┘    └──────┘
                         │ Execute│
                         │ via    │
                         │ToolNode│──── loops back to llm
                         └────────┘
```

### Concept D: Edges — Wiring the Flow

Three types of edges in LangGraph:

| Edge Type | Cerebrum Usage | What It Does |
| --- | --- | --- |
| **Normal edge** | `add_edge("init_context", "add_message")` | Always go from A to B |
| **Conditional edge** | `add_conditional_edges("llm", _route_after_llm, {...})` | Call a function to decide where to go |
| **Entry point** | `set_entry_point("init_context")` | Where the graph starts |

The conditional edge is the most powerful — it's what creates the **agent loop**:

```
def _route_after_llm(state: ChatState) -> str:
    """If LLM returned tool_calls, go to tools; else END."""
    last_tool_calls = state.get("last_tool_calls") or []
    return "tools" if last_tool_calls else "__end__"
```

```
    workflow.add_conditional_edges("llm", _route_after_llm, {"tools": "tools", "__end__": END})
    workflow.add_edge("tools", "llm")
```

This creates a cycle: `llm → tools → llm → tools → ... → END`. The graph keeps looping until the LLM stops requesting tools.

### Concept E: Compile — Locking the Graph

```
    return workflow.compile()
```

`compile()` validates the graph, checks for unreachable nodes, and returns a `CompiledGraph` that's a LangChain `Runnable` — meaning it supports `.invoke()`, `.ainvoke()`, `.stream()`, `.astream()`, etc.

### Concept F: Invoking the Graph

```
    graph = create_chat_graph()
    config = {"recursion_limit": 50}
    final_state = await graph.ainvoke(initial_state, config=config)
```

- `ainvoke` = run the entire graph async, return the final state
- `recursion_limit: 50` = safety valve against infinite tool loops (50 node transitions max)

### Concept G: ToolNode — Prebuilt Tool Execution

Instead of manually parsing tool calls and invoking functions (like the legacy `while` loop), LangGraph provides `ToolNode`:

```
    tool_node = ToolNode(langgraph_tools)
    tool_node_state = {"messages": lc_messages}
    result = await tool_node.ainvoke(tool_node_state)
```

`ToolNode` automatically:

- Reads the last `AIMessage` to find tool calls
- Dispatches each call to the matching `StructuredTool`
- Handles errors gracefully
- Returns `ToolMessage` results

The tools themselves are built from the provider mapping JSON as `StructuredTool`:

```
        tool = StructuredTool.from_function(
            coroutine=resolved_function,
            name=function_name,
            description=description,
            args_schema=args_schema,
            infer_schema=False,
        )
```

---

## 3. Legacy vs LangGraph — Side-by-Side Comparison

| Aspect | Legacy (cerebrum) | LangGraph (cerebrum) |
| --- | --- | --- |
| **Flow control** | `while round_count < 10` loop | Graph edges + conditional routing |
| **Tool execution** | Manual: `await callable_fn(**args)` | `ToolNode` handles dispatch |
| **Tool definition** | `FunctionRegistry` class | `StructuredTool` + OpenAI-format defs |
| **Error handling** | Manual try/except per tool | ToolNode catches + returns error messages |
| **State** | Mutable lists passed around | Immutable `TypedDict`, nodes return partial updates |
| **Context building** | `MessageContextManager` class | Separate nodes: `init_context` → `add_message` → `evaluate_templates` |
| **Visibility** | Flow hidden in imperative code | Flow **is** the graph structure — can be visualized |
| **Feature flag** | Default path | `project_settings["use_langgraph"] == True` |

The routing decision happens here:

```
    _ulg = project_settings.get("use_langgraph")
    use_langgraph = _ulg is True or (
        isinstance(_ulg, str) and _ulg.strip().lower() == "true"
    )

    if use_langgraph:
        async for chunk in get_response_via_langgraph(
            messages, project_id, session_id,
            # ... params ...
        ):
            yield chunk
        return
```

---

## 4. Industry-Relevant LangGraph Knowledge You Need

Here's a structured roadmap from foundational to advanced, marking what cerebrum uses today vs what's available:

### Level 1: Foundations (cerebrum uses these)

| Concept | Status in Cerebrum | What to Know |
| --- | --- | --- |
| `StateGraph` | Used | The main graph builder class |
| `TypedDict` state | Used | Defines the shape of data flowing through |
| Nodes as functions | Used | `async def node(state) -> dict` |
| Normal + conditional edges | Used | `add_edge`, `add_conditional_edges` |
| `ToolNode` (prebuilt) | Used | Auto-dispatches tool calls |
| `StructuredTool` | Used | Wraps Python functions as LangChain tools |
| `ainvoke` | Used | Async graph execution |
| `recursion_limit` | Used | Safety cap on node transitions |

### Level 2: State Management (cerebrum should adopt)

### **Annotated Reducers**

Currently, cerebrum's state uses plain `TypedDict` fields and each node creates a **full copy** of lists:

```python
message_context = list(state["message_context"])  # copy entire list
# ... append ...
return {"message_context": message_context}  # return full list
```

In production LangGraph, you'd use **reducers** to define how updates merge:

```python
from typing import Annotated
from langgraph.graph import add_messages

class ChatState(TypedDict):
    messages: Annotated[list, add_messages]  # auto-appends, deduplicates by ID
    session_id: str
```

With `add_messages`, a node only needs to return the **new** messages — LangGraph handles appending and deduplication. This is the industry standard approach.

### **Checkpointers (Persistence + Time Travel)**

Cerebrum currently has **no checkpointer** — conversation history is managed via Redis in `session_helper.py`. LangGraph offers built-in persistence:

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

# In-memory (dev)
checkpointer = MemorySaver()

# PostgreSQL (production)
checkpointer = AsyncPostgresSaver.from_conn_string(POSTGRES_URL)

graph = workflow.compile(checkpointer=checkpointer)
```

With a checkpointer:

- Every node execution creates a **checkpoint** (snapshot of state)
- You can **resume** from any checkpoint (time travel)
- **Thread-based conversations** work out of the box via `config={"configurable": {"thread_id": session_id}}`
- You get **automatic conversation continuity** without manual Redis get/set

```
Without checkpointer (cerebrum today):
  Request → Load from Redis → Run graph → Save to Redis

With checkpointer:
  Request → graph.ainvoke(state, config={"configurable": {"thread_id": "abc"}})
  LangGraph automatically loads previous state, resumes, and saves
```

### Level 3: Streaming (cerebrum should adopt)

Cerebrum currently uses `ainvoke` which waits for the entire graph to finish. LangGraph offers several streaming modes:

```python
# Stream node-by-node outputs
async for event in graph.astream(state, config):
    print(event)  # {"llm": {"last_llm_content": "Hello..."}}

# Stream individual tokens from LLM
async for event in graph.astream_events(state, config, version="v2"):
    if event["event"] == "on_chat_model_stream":
        print(event["data"]["chunk"].content, end="")

# Stream specific node updates
async for mode, chunk in graph.astream(state, config, stream_mode=["updates", "messages"]):
    ...
```

| Stream Mode | What You Get | Use Case |
| --- | --- | --- |
| `"values"` | Full state after each node | Debugging, logging |
| `"updates"` | Only the partial update from each node | Efficient monitoring |
| `"messages"` | LLM token chunks as they generate | Real-time chat UI |
| `astream_events` | Fine-grained events (tool start/end, LLM tokens) | Complex UIs, progress bars |

### Level 4: Human-in-the-Loop

LangGraph has first-class support for pausing execution and waiting for human input:

```python
from langgraph.graph import StateGraph, END

workflow = StateGraph(State)
workflow.add_node("agent", agent_node)
workflow.add_node("tools", tool_node)

# Add an interrupt BEFORE tools execute
graph = workflow.compile(
    checkpointer=checkpointer,
    interrupt_before=["tools"]  # pause here, ask human to approve
)
```

```
Flow with human-in-the-loop:

User: "Book me an appointment for tomorrow 3pm"
  │
  ▼
[llm] → tool_call: book_appointment(date="tomorrow", time="3pm")
  │
  ▼
[INTERRUPT] ← Graph pauses here
  │
  ▼
System: "I'd like to book tomorrow at 3pm. Shall I proceed?" → User
  │
User: "Yes"
  │
  ▼
graph.ainvoke(None, config)  ← Resume from checkpoint
  │
  ▼
[tools] → actually executes book_appointment
```

This is very relevant for cerebrum's booking use case — confirming appointments before executing them.

### Level 5: Subgraphs and Multi-Agent

```
┌─────────────────────────────────────────┐
│              Supervisor Agent            │
│                                         │
│  "Route to the right specialist"        │
│                                         │
│   ┌──────────┐  ┌──────────┐  ┌──────┐│
│   │ Booking  │  │ FAQ      │  │ Triage││
│   │ Agent    │  │ Agent    │  │ Agent ││
│   │(subgraph)│  │(subgraph)│  │      ││
│   └──────────┘  └──────────┘  └──────┘│
└─────────────────────────────────────────┘
```

LangGraph supports **subgraphs** — a graph within a graph:

```python
booking_graph = create_booking_subgraph().compile()
faq_graph = create_faq_subgraph().compile()

main = StateGraph(MainState)
main.add_node("router", router_node)
main.add_node("booking", booking_graph)  # subgraph as node
main.add_node("faq", faq_graph)
main.add_conditional_edges("router", route_fn, {"booking": "booking", "faq": "faq"})
```

### Level 6: Advanced Industry Patterns

### **Map-Reduce (Parallel Execution)**

```python
# Fan-out: run multiple nodes in parallel, collect results
from langgraph.constants import Send

def route_to_parallel(state):
    return [Send("process_item", {"item": item}) for item in state["items"]]

workflow.add_conditional_edges("start", route_to_parallel, ["process_item"])
```

### **Dynamic Breakpoints**

```python
from langgraph.errors import NodeInterrupt

async def tool_node(state):
    if state["amount"] > 1000:
        raise NodeInterrupt("High-value transaction requires approval")
    # proceed normally
```

### **Cross-Thread Memory (Shared Stores)**

```python
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()
graph = workflow.compile(checkpointer=checkpointer, store=store)

# In a node:
async def node(state, config, *, store):
    user_prefs = store.search(("user_preferences", user_id))
    store.put(("user_preferences", user_id), "theme", {"value": "dark"})
```

This enables long-term memory across different conversation threads — something beyond what cerebrum's per-session Redis history provides.

### **LangGraph Platform / LangGraph Cloud**

- Managed deployment of graphs as APIs
- Built-in persistence, streaming, cron jobs
- LangSmith integration for tracing/debugging
- This is the production deployment story LangChain is pushing

---

## 5. Knowledge Roadmap — What to Learn in Order

```
YOU ARE HERE
    │
    ▼
┌──────────────────────────────────────────────────────────────┐
│ LEVEL 1: Foundations ✅ (cerebrum already uses these)         │
│  StateGraph, nodes, edges, conditional edges, ToolNode       │
│  ainvoke, recursion_limit, TypedDict state                   │
└────────────────────────────┬─────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────┐
│ LEVEL 2: State & Persistence                                 │
│  Annotated reducers (add_messages), Checkpointers            │
│  Thread-based conversations, Time travel / replay            │
│  → Would replace cerebrum's manual Redis session mgmt        │
└────────────────────────────┬─────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────┐
│ LEVEL 3: Streaming                                           │
│  astream (values/updates/messages), astream_events           │
│  Token-level streaming for real-time chat UI                 │
│  → Would enable cerebrum LangGraph path to stream like legacy│
└────────────────────────────┬─────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────┐
│ LEVEL 4: Human-in-the-Loop                                   │
│  interrupt_before/after, dynamic breakpoints                 │
│  NodeInterrupt, resume from checkpoint                       │
│  → Critical for booking confirmations in cerebrum            │
└────────────────────────────┬─────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────┐
│ LEVEL 5: Multi-Agent & Subgraphs                             │
│  Graphs as nodes, supervisor patterns, handoffs              │
│  Map-reduce for parallel processing                          │
│  → Future: route to booking agent vs FAQ agent vs triage     │
└────────────────────────────┬─────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────┐
│ LEVEL 6: Production / Platform                               │
│  LangGraph Cloud, LangSmith tracing, Shared stores           │
│  Cross-thread memory, Cron jobs, Deployment                  │
│  → Long-term: managed infrastructure for all concerto AI     │
└──────────────────────────────────────────────────────────────┘
```

---

## 6. Quick Reference — LangGraph API Cheat Sheet

| API | Purpose | Example |
| --- | --- | --- |
| `StateGraph(Schema)` | Create graph | `workflow = StateGraph(ChatState)` |
| `.add_node(name, fn)` | Register a node | `workflow.add_node("llm", llm_node)` |
| `.add_edge(a, b)` | Fixed transition | `workflow.add_edge("tools", "llm")` |
| `.add_conditional_edges(node, fn, map)` | Dynamic routing | `workflow.add_conditional_edges("llm", route_fn, {...})` |
| `.set_entry_point(name)` | Start node | `workflow.set_entry_point("init_context")` |
| `.compile(checkpointer=)` | Lock + optimize | `graph = workflow.compile()` |
| `graph.ainvoke(state, config)` | Run full graph | Returns final state |
| `graph.astream(state, config)` | Stream node outputs | Yields per-node updates |
| `graph.astream_events(state)` | Stream everything | Token-level events |
| `ToolNode(tools)` | Prebuilt tool executor | Auto-dispatches tool calls |
| `MemorySaver()` | In-memory checkpoints | Dev/testing |
| `AsyncPostgresSaver` | Postgres checkpoints | Production persistence |
| `interrupt_before=["node"]` | Pause before node | Human-in-the-loop |
| `Send(node, state)` | Fan-out | Parallel node execution |

---

## Summary

Cerebrum's LangGraph implementation is a clean Level 1 usage — it replaced the manual `while` loop with a declarative `init_context → add_message → evaluate_templates → llm ⟷ tools → END` graph. The biggest industry-relevant features it hasn't adopted yet are **checkpointing** (would eliminate the manual Redis history management), **streaming** (would match the legacy path's token streaming), and **human-in-the-loop** (natural fit for booking confirmations). The multi-agent patterns at Level 5 are where the industry is heading — routing conversations to specialized subgraphs based on intent.
