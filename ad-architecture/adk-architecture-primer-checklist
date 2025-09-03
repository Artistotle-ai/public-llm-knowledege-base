# ADK Multi-Agent Architecture Checklist

## Agent Types
- [ ] **LLM Agents**: Use for reasoning, planning, open-ended tasks.  
- [ ] **Workflow Agents**:  
  - `SequentialAgent` → step-by-step pipeline  
  - `ParallelAgent` → run subtasks concurrently  
  - `LoopAgent` → repeat until condition  
- [ ] **Custom Agents**: For APIs or non-LLM logic.

---

## Architecture Patterns
- [ ] **Hierarchy**: Define parent agent → add `sub_agents`.  
- [ ] **Coordinator/Dispatcher**: Convert child agents into `AgentTool`s, call via parent.  
- [ ] **Sequential Pipeline**: Ordered execution, pass data via `output_key` → `session.state`.  
- [ ] **Parallel Fan-Out/Gather**: Independent subtasks run in parallel, each writes to unique state key.  
- [ ] **Looping**: Repeat agents until stop condition (max iterations / escalate).  
- [ ] **Review/Critique**: Add reviewer agents to validate or trigger rework.

---

## Best Practices
- [ ] **Single Responsibility**: One clear role per agent.  
- [ ] **Clear Interfaces**: Define input/output (schemas, JSON).  
- [ ] **State Management**: Use `session.state` instead of overloading prompts.  
- [ ] **Parallelize**: Put independent work into `ParallelAgent`.  
- [ ] **Validation**: Add critic/reviewer for quality control.  
- [ ] **Test & Iterate**: Start simple (sequential), expand with parallel + loops.  
- [ ] **Logging/Debugging**: Use ADK memory/log tools during build.

---

## Quick Flow Design
1. Break problem → specialized agents.  
2. Decide control flow: Sequential / Parallel / Loop.  
3. Define inputs/outputs + `output_key`s.  
4. Connect agents via `sub_agents` or tools.  
5. Add validation/review if needed.  
6. Test incrementally, expand complexity gradually.  

---

✅ Use this checklist to design robust multi-agent flows in ADK.  
