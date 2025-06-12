##########################
####### MULTI AGENTS #####
##########################

This workflow demonstrates how to use the multi-agent commands for parallel task execution.

## Overview

The multi-agent workflow allows you to distribute tasks across 3-5 specialized agents that work in parallel. This significantly speeds up development while maintaining code quality and avoiding conflicts.

## Step-by-Step Workflow

### Step 1: Generate PRD (if needed)

If you don't have a PRD yet, generate one from your design document:

```bash
/user:prd:generate design-doc.md
```

### Step 2: Generate Initial Task List

Generate a comprehensive task list from your PRD:

```bash
/user:tasks:generate tasks/project-prd.md
```

### Step 3: Distribute Tasks to Multiple Agents

Use the multi-agent task generator to automatically distribute tasks:

```bash
/user:tasks:generate-multi-agent tasks/tasks-list.md --agents 4
```

This command will:
- Analyze the task list and identify optimal groupings
- Create separate task lists for each agent (e.g., `tasks/agent-1-tasks.md`, `tasks/agent-2-tasks.md`)
- Generate a coordination file at `tasks/multi-agent-coordination.md`
- Create agent-specific prompts with clear responsibilities

### Step 4: Execute Tasks in Parallel

Option A - Deploy all agents at once:
```bash
/user:tasks:execute-multi-agent
```

Option B - Deploy specific agents:
```bash
/user:tasks:execute-multi-agent --agents 1,3 --mode parallel
```

Option C - Use the deployment command for custom scenarios:
```bash
/user:deploy:multiple-agents "Complete the backend API tasks" --from-file design-doc.md
```

### Step 5: Monitor Progress

The coordination file (`tasks/multi-agent-coordination.md`) tracks:
- Agent status and progress
- Task dependencies and handoffs
- Completion percentages
- Context usage

### Step 6: Continue Interrupted Work

If an agent stops due to context limits:

```bash
# Continue a specific agent
/user:tasks:continue 2

# Continue all agents in parallel
/user:tasks:continue-parallel
```

### Step 7: Handle Task Reorganization

If tasks need to be rebalanced between agents:

```bash
/user:tasks:reorganize tasks/agent-1-tasks.md tasks/agent-2-tasks.md
```

### Step 8: Mark Tasks Complete

When all agents finish their work:

```bash
/user:tasks-mark-complete
```

### Step 9: Commit All Changes

```bash
/user:tasks-commit
```

### Step 10: Review and Iterate

After completion, review the work and generate next iteration:

```bash
/user:prd:iterate-project --mode improve
```

## Agent Prompt Structure

When agents are created, they receive structured prompts like:

```markdown
You are Agent {N} responsible for {focus-area}. Complete these {X} tasks from @tasks/agent-{N}-tasks.md.

**Coordination File**: @tasks/multi-agent-coordination.md

**Docs to Reference:**
- Project Design Doc: @{design-doc-path}
- Original PRD: @{prd-path}
- Your Task List: @tasks/agent-{N}-tasks.md

**Tasks:**
{task-list}

**Your focus areas:**
{focus-areas}

**Key requirements:**
{requirements}

**Dependencies:**
{dependencies}

**Success criteria:**
{success-criteria}

For each task:
- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Update coordination file when starting/completing tasks
- Commit after each completed subtask
- Mark tasks complete with [x] in your task list
- Check coordination file for handoffs from other agents
```

## Coordination File Format

The multi-agent coordination file tracks:

```markdown
# Multi-Agent Task Coordination

## Agent Status
- Agent 1: Active (65% complete, 45% context used)
- Agent 2: Waiting (80% complete, 78% context used)
- Agent 3: Active (50% complete, 30% context used)

## Task Dependencies
- Agent 2 waiting for Agent 1 to complete API endpoints
- Agent 3 can proceed independently

## Handoffs
- [ ] Agent 1 → Agent 2: API endpoint definitions
- [x] Agent 1 → Agent 3: Database models completed

## Progress Summary
Total Tasks: 45
Completed: 28 (62%)
In Progress: 7
Blocked: 2
Remaining: 8
```

## Best Practices

1. **Optimal Agent Count**: Use 3-5 agents for best results
2. **Task Distribution**: Let the system analyze and distribute tasks automatically
3. **Dependency Management**: The coordination file prevents conflicts
4. **Context Management**: Agents automatically pause at 80% context usage
5. **Atomic Commits**: Each subtask completion triggers a git commit
6. **Progress Tracking**: All task lists are updated in real-time

## Troubleshooting

### If only one agent was running:
```bash
/user:parallel-persist
```

### If tasks need redistribution:
```bash
/user:tasks:reorganize tasks/agent-1-tasks.md tasks/agent-2-tasks.md
```

### To review all agent progress:
```bash
# Check coordination file
cat tasks/multi-agent-coordination.md

# Check individual agent task lists
ls tasks/agent-*-tasks.md
```

## Advanced Usage

### Custom Agent Deployment

For specialized scenarios:

```bash
/user:deploy:multiple-agents "Create 3 agents to review and optimize each microservice" \
  --from-file architecture.md \
  --coordination-file tasks/optimization-coordination.md
```

### Project DevOps Review

Deploy agents to review different aspects of the project:

```bash
/user:project-devops
```

This creates agents to:
- Review and update README
- Analyze codebase structure
- Review and improve tests
- Generate development environment setup

## Example Complete Workflow

```bash
# 1. Generate PRD from design
/user:prd:generate design.md references/api-spec.md

# 2. Generate comprehensive task list
/user:tasks:generate

# 3. Distribute to 4 agents
/user:tasks:generate-multi-agent tasks/tasks-list.md --agents 4

# 4. Execute all agents in parallel
/user:tasks:execute-multi-agent

# 5. Monitor progress (in another terminal)
watch cat tasks/multi-agent-coordination.md

# 6. Continue any stopped agents
/user:tasks:continue-parallel

# 7. Mark all complete
/user:tasks-mark-complete

# 8. Commit everything
/user:tasks-commit

# 9. Generate next iteration
/user:prd:iterate-project --mode next-phase
```

## Key Differences from Manual Approach

1. **Automated Distribution**: No manual task splitting required
2. **Smart Coordination**: Built-in dependency tracking
3. **Parallel Execution**: Single command deploys all agents
4. **Progress Tracking**: Centralized coordination file
5. **Resume Capability**: Easy continuation of interrupted work
6. **Conflict Prevention**: Agents avoid working on same files