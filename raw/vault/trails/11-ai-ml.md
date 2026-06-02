# Trail: AI & Modern Stack

**Objective:** Understand how LLMs work, how to make them domain-specific, and how to build production agent systems — from evaluation metrics to the architecture of this very second brain.
**Prerequisites:** Trail 04 — Backend Patterns (for API/integration context)
**Review time:** ~30 min
**Nodes:** 6

---

1. [[Confusion Matrix, L1, L2]]
   > Before deploying any model, you need to measure it. This note covers the confusion matrix (TP/FP/TN/FN → precision/recall/F1), and L1/L2 regularization (penalizing model complexity to prevent overfitting). These are the vocabulary of model evaluation — you need them to have any informed conversation about whether an AI system is "working."

2. [[LLMs- Fine Tuning, RAG]]
   > Two strategies for making a general LLM useful in your specific domain. Fine-tuning bakes domain knowledge into weights (expensive, fast inference). RAG retrieves relevant documents at query time and augments the prompt (flexible, current data). CAG preloads a full knowledge base into context. This second brain uses a variant of RAG/CAG — you're reading about the pattern it's built on.

3. [[AI Agents]]
   > LLMs as controllers: reasoning (planning), acting (tool use), and memory. This note covers the React (Reasoning + Acting) pattern, how agents break down complex problems into sub-tasks, and where agent architectures work better than RAG alone. Agents are the architecture; LangGraph (next note) is the framework.

4. [[MCP]]
   > Model Context Protocol: the standard for how LLMs interface with external tools and data sources. This second brain is exposed as an MCP server so your LLM can read your vault and tailor explanations to what you already know. After this note, the phrase "expose your vault as MCP" is a concrete technical act, not an abstraction.

5. [[Langgraph Flow]]
   > LangGraph replaces imperative while-loops with a declarative graph: nodes are async functions, edges define control flow, shared state flows through the graph. This note covers the full API (StateGraph, ToolNode, conditional edges, checkpointers) and the Cerebrum implementation from Ripples. Level 1–6 roadmap included.

6. [[Software Is Changing (Again) by Andrej Karpathy]]
   > Zoom out: Karpathy's essay on the shift from traditional software to LLM-native software. After five notes on AI mechanics, this note asks the bigger question: what does this mean for how software is built, and for your role as an engineer? Read this last — it reframes everything in the trail as part of a larger shift.
