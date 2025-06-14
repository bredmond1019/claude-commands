# /user:deploy:multiple-agents --prompt <prompt> [--from-file <path>] [--coordination-file <path>]

Deploy multiple agents to work in parallel to generate a detailed task list.

## Usage

```
/user:deploy:multiple-agents --prompt <prompt> [--from-file <path>]
```

## Arguments

- `--prompt <prompt>`: Prompt describing the task for the agents that will be generated (required)
- `--from-file <path>`: Path to the main design document (optional)
- `--coordination-file <path>`: Path to the coordination file (optional)

Example: `/user:deploy:multiple-agents "Generate 3 Agents to review each microservice and create documentation for each" --from-file project-design.md --coordination-file coordination.md`

Review the following files if they exist:

User Prompt: {<prompt>}
Design Document/Additional Context: {<from-file>}
Agent Coordination File: {<coordination-file>}

## Process

1. **Load Coordination File**: Parse multi-agent coordination to understand task distribution
2. **Generate Agent Prompts**: Create specific prompts for each agent based on their tasks
3. **Spawn Claude Instances**: Launch new Claude instances for parallel execution with the agent prompts
4. **Monitor Progress**: Track agent progress through task files and coordination updates
5. **Coordinate Handoffs**: Manage dependencies and handoff points between agents
6. **Aggregate Results**: Collect and summarize completion status across all agents

**Parallel Execution Management:**
Deploy multiple Sub Agents to generate agents in parallel for maximum efficiency and creative diversity:

- Launch 3-5 Sub Agents simultaneously using Task tool
- Monitor agent progress and completion
- Collect and validate all completed agents

## Agent Task File Format

Each agent receives a task file with:

- Agent role and primary focus
- Assigned tasks from the original list
- Relevant files for their scope
- Dependencies on other agents
- Handoff points for coordination
- Testing responsibilities

```markdown
You are Agent <X> responsible for <your-focus-area>. Complete these <Y> tasks from <Z>. Use @tasks/ai-dev-tasks/process-task-list.mdc

**Tasks:**

**Your focus areas:**

**Key requirements:**

**Dependencies:**

**Success criteria:**

**Optional Files to Reference:**

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them.
```
