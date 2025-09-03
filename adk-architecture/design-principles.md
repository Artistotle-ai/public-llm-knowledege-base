# Google ADK Multi-Agent Architecture and Design Patterns

Google’s Agent Development Kit (ADK) encourages **modular, multi-agent architectures** over monolithic designs.  
Instead of one giant agent, break a complex task into specialized agents organized hierarchically .  
This improves modularity, clarity, and maintainability: each agent has a narrow focus and you can reuse or replace parts without rewriting everything.  

The **root (or coordinator) agent** acts as an orchestrator, not a worker – it manages the workflow, delegates sub-tasks to specialized agents, and interacts with the user  .  
For example, a “HelpDeskCoordinator” agent might read a user query and, using its instruction or tools, route it to a `BillingAgent` or `SupportAgent` .

Each ADK **BaseAgent** (often an `LlmAgent`) handles a single responsibility.  
For deterministic control flow, ADK provides **Workflow Agents**: `SequentialAgent`, `ParallelAgent`, and `LoopAgent`.  
Use these to enforce order, concurrency, or repetition in the overall process.  

- Wrap a series of reasoning steps in a `SequentialAgent` so each runs in turn, passing results via the shared session state .  
- Use a `ParallelAgent` when multiple independent subtasks can be done simultaneously to save time (a *fan-out/gather* pattern) .  
- Loops (`LoopAgent`) let you repeat subtasks until a condition is met (e.g. refine an answer until it’s good enough) .  

All communication between agents happens via **shared `session.state`**.  
Agents write outputs to state (using `output_key` or structured `output_schema`) and other agents read from it.  
Always define clear state keys so data flows explicitly from one step to the next .  
For example, in a pipeline one agent might write `state["draft"]` and the next reads `{draft}` in its instruction.  
Avoid hiding data in the prompt; use state variables for any intermediate results or parameters.

---

## Core Agent Patterns

Use ADK’s primitives to implement standard collaboration patterns:

### Coordinator/Dispatcher
A central LLM agent routes work to specialist sub-agents .  
You can list child agents in `sub_agents` so that the coordinator can use “agent transfer” (telling the LLM to call a sub-agent by name) or wrap sub-agents as `AgentTool`s for explicit invocation.  
This pattern is ideal when tasks clearly belong to different domains (e.g. billing vs support) .

### Sequential Pipeline
A `SequentialAgent` runs a fixed sequence of sub-agents, passing outputs along via state .  
Use this to model multi-step processes (e.g. validate input → process data → report results).  
This enforces a deterministic flow and is easy to debug, but can be slower if steps could have run in parallel.

### Parallel Fan-Out/Gather
Use a `ParallelAgent` when multiple subtasks are independent and can run concurrently .  
For example, dispatch several information-gathering agents at once (web search, database queries, analysis) and then use a downstream agent to consolidate their results.  
This pattern speeds up independent work but requires careful key management and later aggregation logic.

### Hierarchical Task Decomposition
Build a tree of agents for complex tasks .  
Higher-level agents delegate subtasks to lower-level agents (or even wrap them as tools).  
Decompose the problem recursively: each agent only needs to handle one layer of abstraction.  
This makes very complex goals tractable, though it introduces more layers and potential overhead .

### Generator–Critic (Review/Critique) Pattern
Chain a *generator* agent followed by a *critic* agent in a `SequentialAgent` .  
The critic evaluates or critiques the generator’s output, optionally looping back corrections.  
This improves quality and catches errors, at the cost of extra compute.  

### Iterative Refinement (Loop) Pattern
Use a `LoopAgent` to improve an output over multiple iterations .  
For example, generate an answer, evaluate it, and repeat until it meets success criteria.  
Always set a reasonable `max_iterations` and clear success criteria to avoid infinite loops.

### Human-in-the-Loop
Sometimes you need human oversight or approval.  
Implement checkpoints via tools (e.g. an `approval` `FunctionTool` that pauses until a human responds) .  
This is essential for compliance, high-risk decisions, or uncertain cases.

### Tool Use
Wherever possible, offload tasks to deterministic tools instead of LLMs .  
Define a `FunctionTool` (calculation, database query, code execution, etc.) and include it in your agent’s `tools` list.  
This reduces cost and increases reliability.  
You can also wrap agents as tools (`AgentTool`) to treat entire workflows as callable functions .

---

## Design Principles and Best Practices

- **Single Responsibility:** Each agent has a focused task .  
- **Explicit Interfaces:** Define clear inputs/outputs, use schemas or `output_key`.  
- **Appropriate Agent Types:**  
  - LLM agents for reasoning/understanding.  
  - Workflow agents for orchestration.  
  - Custom agents/tools for deterministic or external logic.  
- **Model Selection and Cost:** Use strong models for hard reasoning, smaller models or tools for simpler tasks  .  
- **State Management:** Use `session.state` consistently; avoid prompt bloat.  
- **Testing & Iteration:** Start simple (sequential), then add parallelism/loops. Log intermediate states.  
- **Monitoring & Fail-safes:** Handle failures gracefully; set retries, timeouts, and safe defaults.  

---

## Trade-offs and Pitfalls

- **Over-Engineering:** Don’t add complexity unless necessary .  
- **Unclear Boundaries:** Overlapping roles cause confusion .  
- **Excessive Delegation:** Too many micro-agents increase latency .  
- **State Confusion:** Inconsistent state usage breaks workflows .  
- **No Error Handling:** Always plan for retries, fallbacks, or escalation .  
- **Cost and Latency:** More agents = more calls = higher cost. Prefer tools/smaller models when possible.  
- **Context Window Limits:** Summarize or use retrieval instead of passing long texts through multiple steps.  

---

## Summary

Leverage ADK’s building blocks to compose a hierarchy of specialized agents.  
Use workflow agents to structure control flow (pipelines, parallel branches, loops)  .  
Always define clear interfaces, use `session.state`, and start with simple flows before adding complexity.  
Monitor costs: prefer tools and lightweight models where possible, and reserve heavyweight LLM calls for complex reasoning  .  

By following these principles and patterns, you can build robust, maintainable multi-agent systems in ADK that tackle complex tasks effectively.

---

**Sources:** ADK official documentation and design guides     , and practical architecture examples in Google’s ADK resources  .
