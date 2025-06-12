# Reorganize Tasks Command

Reorganize and rebalance tasks between agents or within a single task list.

## Usage

```
/user:tasks:reorganize <source-list> [target-list] [options]
```

## Arguments

- `source-list`: Path to the source task list file (required)
- `target-list`: Path to the target task list file (optional, for inter-agent moves)
- `options`: Additional reorganization options (merge, split, rebalance)

Examples:

- `/user:reorganize-tasks tasks/tasks-list.md` (reorganize single list)
- `/user:reorganize-tasks tasks/agent-1-list.md tasks/agent-2-list.md` (move tasks between agents)
- `/user:reorganize-tasks tasks/agent-*.md --rebalance` (rebalance all agent lists)

## Process Overview

I will:

1. Analyze current task distribution and completion status
2. Identify reorganization opportunities
3. Present reorganization plan for approval
4. Execute the reorganization
5. Update coordination files if multi-agent

## Implementation

### Step 1: Parse Arguments and Load Task Lists

Let me parse the provided arguments: $ARGUMENTS

I'll determine:

- Source task list(s) to reorganize
- Target destination (if specified)
- Reorganization mode (single list, inter-agent, or multi-agent rebalance)

### Step 2: Analyze Current Task State

I'll analyze the task list(s) to understand:

**Task Status Analysis:**

- Completed tasks (marked with [x])
- In-progress tasks
- Pending tasks
- Task dependencies and relationships

**Workload Analysis:**

- Task count per agent (if multi-agent)
- Estimated complexity per task group
- Time estimates based on subtask count
- Current progress percentage

**Reorganization Opportunities:**

1. **Dependency Optimization**: Reorder tasks based on dependencies
2. **Workload Balancing**: Even distribution across agents
3. **Skill Matching**: Group related tasks together
4. **Progress Optimization**: Move blocked tasks to available agents
5. **Completion Grouping**: Consolidate nearly-complete sections

### Step 3: Generate Reorganization Plan

Based on the analysis, I'll create a reorganization plan:

**Single List Reorganization:**

```markdown
## Reorganization Plan for: [filename]

### Current Structure:

- Total tasks: X (Y completed, Z pending)
- Current order: [brief description]

### Proposed Changes:

1. Move task 3.0 before 2.0 (dependency optimization)
2. Group all UI tasks together (tasks 2.3, 4.1, 5.2)
3. Move completed tasks to bottom

### Benefits:

- Clearer implementation path
- Better dependency flow
- Improved task grouping
```

**Inter-Agent Reorganization:**

```markdown
## Inter-Agent Task Movement Plan

### From: agent-1-list.md

Moving tasks:

- [ ] 3.4 Task description (overloaded agent)
- [ ] 3.5 Related task

### To: agent-2-list.md

Reason: Agent 2 has completed 80% of tasks

### Coordination Updates:

- Update multi-agent-coordination.md
- Adjust agent prompts
```

**Multi-Agent Rebalancing:**

```markdown
## Multi-Agent Rebalancing Plan

### Current Distribution:

- Agent 1: 15 tasks (3 complete, 12 pending)
- Agent 2: 8 tasks (7 complete, 1 pending)
- Agent 3: 14 tasks (5 complete, 9 pending)

### Proposed Rebalance:

- Agent 1: 12 tasks (-3)
- Agent 2: 11 tasks (+3)
- Agent 3: 14 tasks (no change)

### Task Movements:

1. Agent 1 → Agent 2: Tasks 4.5-4.7
2. Dependencies preserved
3. Related tasks kept together
```

### Step 4: Confirm Reorganization

**Would you like to proceed with this reorganization plan? (yes/no)**

If yes, I'll:

1. Execute the proposed changes
2. Update all affected task files
3. Maintain task formatting and numbering
4. Update coordination files

If no, you can:

1. Provide specific adjustments
2. Cancel the reorganization
3. Choose a different reorganization strategy

### Step 5: Execute Reorganization

Once confirmed, I'll:

**File Updates:**

1. **Backup current state** (in memory)
2. **Rewrite task lists** with new organization
3. **Preserve task state** (completed/pending status)
4. **Update task numbers** if needed
5. **Maintain subtask relationships**

**For Multi-Agent:**

1. Update `tasks/multi-agent-coordination.md`
2. Adjust workload distribution section
3. Update dependencies if affected
4. Note reorganization in history

### Step 6: Verify and Report

After reorganization:

**Verification Steps:**

1. Confirm all tasks accounted for
2. Verify no tasks lost or duplicated
3. Check dependency integrity
4. Validate file formats

**Final Report:**

```markdown
## Reorganization Complete

### Summary:

- Tasks reorganized: X
- Files updated: Y
- New distribution: [summary]

### Next Steps:

1. Review updated task lists
2. Resume work with new organization
3. Use /user:continue-tasks for agents to continue
```

## Special Features

### Handling Completed Agents

If an agent has completed all tasks:

```
/user:reorganize-tasks tasks/agent-1-list.md --reassign-completed
```

This will:

1. Identify the completed agent
2. Find agents with remaining work
3. Suggest optimal task redistribution
4. Reassign the completed agent to help

### Emergency Rebalancing

For significant imbalances:

```
/user:reorganize-tasks tasks/agent-*.md --emergency-rebalance
```

This will:

1. Completely redistribute all pending tasks
2. Optimize for fastest completion
3. Ignore original groupings if needed
4. Focus on parallel execution

## Important Notes

- Always preserves task completion status
- Maintains parent-subtask relationships
- Updates coordination files automatically
- Creates reorganization history for tracking
- Handles dependencies intelligently
- Supports both minor adjustments and major restructuring
