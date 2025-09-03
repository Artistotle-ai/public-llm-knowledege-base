# Google ADK Multi-Agent Architecture Guide

## Agent Types
ADK agents are modular **BaseAgent** units. Three core categories:

- **LLM Agents**: Flexible language-driven (`LlmAgent` / `Agent`).
- **Workflow Agents**: Deterministic controllers (`SequentialAgent`, `ParallelAgent`, `LoopAgent`) managing sub-agents’ execution order.
- **Custom Agents**: Specialized `BaseAgent` subclasses for bespoke integrations or logic.

This separation lets you assign single-purpose roles and compose them hierarchically.

---

## Multi-Agent System Design

ADK encourages **composing multiple agents** rather than one monolith.  
A multi-agent system (MAS) is a parent agent with child agents (`sub_agents`), forming a hierarchy.

### Core Patterns
- **Hierarchy**: Create a parent (coordinator) with `sub_agents`. Each child has one parent.
- **Coordinator/Dispatcher**: Convert agents to `AgentTool`s and list them in the coordinator’s `tools`. The coordinator then “calls” sub-agents as tools.
- **Sequential Pipeline**: Use `SequentialAgent`. Each sub-agent runs in order, passing context and writing outputs to `session.state` via `output_key`.
- **Parallel Fan-Out/Gather**: Use `ParallelAgent`. Sub-agents run concurrently, writing to distinct state keys. Often nested in sequential flows.
- **Looping**: Use `LoopAgent` to repeat subtasks until a condition. Terminate via max iterations or escalation.
- **Communication via State**: Pass data through `session.state`. Agents write with `output_key`, later agents read from the same key.
- **Review/Critique Pattern**: Add reviewer agents sequentially to validate or iterate on outputs.

---

## Design Best Practices
- **Single Responsibility**: Each agent has a clear, narrow job.
- **Clear Interfaces**: Define input/output (JSON or schemas). Always use `output_key` for structured outputs.
- **Mind Agent Types**:  
  - Use LLM Agents for reasoning/open tasks.  
  - Use Workflow Agents for control flow/parallelism.  
  - Use Custom Agents for external logic or APIs.
- **State Management**: Use `session.state` to pass results. Avoid large prompt context.
- **Parallelize Independent Work**: Use `ParallelAgent` for independent subtasks.
- **Validation Loops**: Add critic/reviewer agents for quality control, possibly inside loops.
- **Testing & Iteration**: Start simple (sequential), then extend with parallelism/loops. Use logging & memory tools to debug.

---

## Summary
To design robust multi-agent systems with Google ADK:
1. Compose specialized agents hierarchically.  
2. Use workflow agents (`Sequential`, `Parallel`, `Loop`) to structure flows.  
3. Pass data via `session.state`.  
4. Apply patterns: pipelines, fan-out/gather, loops, and reviewers.  
5. Keep agents modular, interfaces clear, and flows testable.  

This provides a flexible foundation for any complex agentic flow.

---

## Sources
- ADK documentation: [Agent types and workflow agents](https://google.github.io/adk-docs/agents/)    
- ADK documentation: [Multi-agent patterns and sub_agents](https://google.github.io/adk-docs/tutorials/multi_agent/)    
- ADK documentation: [ParallelAgent, critique/review loops](https://google.github.io/adk-docs/tutorials/workflow_agents/)    
