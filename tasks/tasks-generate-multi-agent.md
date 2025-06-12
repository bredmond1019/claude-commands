# /user:generate-multi-agent-tasks

# Generating Multi-Agent Task Distribution from a Completed Task List

## Goal

To guide an AI assistant in taking an already completed task list (generated from @ai-dev-tasks/generate-tasks.mdc) and distributing it across 3-5 specialized agents for parallel execution. The task distribution should optimize for concurrent work while maintaining logical dependencies.

## Usage

```
/user:tasks:generate-multi-agent <task-list-file> [--agents <number>] [--output-dir <path>]
```

## Arguments

- `<task-list-file>`: Path to the completed task list file (typically `tasks/tasks-list.md`)
- `--agents <number>`: Optional. Number of agents to distribute tasks across (3-5, default: auto-detect)
- `--output-dir <path>`: Optional. Directory for output files (default: `tasks/`)

## Input Requirements

- **Required:** A completed task list file (tasks-list.md) that was generated using the generate-tasks.mdc process
- **Expected Format:** The task list should contain parent tasks, sub-tasks, and relevant files as specified in @ai-dev-tasks/generate-tasks.mdc

## Output

- **Format:** Multiple Markdown (`.md`) files
- **Location:** `/tasks/`
- **Filenames:**
  - `agent-<X: int>-tasks.md` (one file per agent)
  - `multi-agent-coordination.md` (coordination overview)

## Process

1. **Receive Task List Reference:** The user points the AI to a completed task list file (tasks-list.md)
2. **Analyze Task List:** The AI reads and analyzes the existing parent tasks, sub-tasks, dependencies, and relevant files.
3. **Assess Parallelization Opportunities:** Determine the optimal number of agents (3-5) based on:
   - Task complexity and interdependencies
   - Number of distinct components/modules
   - Opportunities for parallel development
   - File ownership and integration points
4. **Phase 1: Generate Agent Roles & Task Distribution:** Create specialized agent roles and distribute existing tasks among them. Present the distribution to the user without creating files yet. Inform the user: "I have distributed the tasks across [X] agents. Ready to generate individual agent task files? Respond with 'Go' to proceed."
5. **Wait for Confirmation:** Pause and wait for the user to respond with "Go".
6. **Phase 2: Generate Individual Agent Files:** Once confirmed, create separate task files for each agent with their specific responsibilities, dependencies, and handoff requirements.
7. **Create Coordination File:** Generate a coordination overview file that shows dependencies, integration points, and timeline.
8. **Save All Files:** Save the individual agent files and coordination file in the `/tasks/` directory.

## Output Files

- `tasks/agent-1-tasks.md` through `tasks/agent-N-tasks.md`: Individual agent task lists
- `tasks/multi-agent-coordination.md`: Coordination overview with dependencies and milestones

## Agent Distribution Strategy

### 3-Agent Setup (Simple Features)

- **Frontend Agent:** UI components, user interactions, styling
- **Backend Agent:** API endpoints, business logic, data persistence
- **Integration Agent:** Testing, deployment, cross-system integration

### 4-Agent Setup (Medium Features)

- **UI/UX Agent:** Frontend components, user experience, styling
- **API Agent:** Backend endpoints, data validation, business logic
- **Data Agent:** Database schema, data models, migrations
- **QA Agent:** Testing, integration, deployment, documentation

### 5-Agent Setup (Complex Features)

- **Frontend Agent:** User interface components and interactions
- **Backend Agent:** Core business logic and API endpoints
- **Data Agent:** Database design, models, and data layer
- **Integration Agent:** Third-party services, external APIs
- **QA Agent:** Testing, quality assurance, deployment

## Task Distribution Algorithm

```python
def distribute_tasks(tasks, agent_count):
    # 1. Group tasks by functional area
    task_groups = group_by_area(tasks)

    # 2. Calculate task complexity scores
    complexity_scores = calculate_complexity(tasks)

    # 3. Balance workload across agents
    agent_assignments = balance_workload(task_groups, complexity_scores, agent_count)

    # 4. Identify dependencies between agents
    dependencies = identify_dependencies(agent_assignments)

    # 5. Create execution phases for parallel work
    execution_phases = create_phases(agent_assignments, dependencies)

    return agent_assignments, dependencies, execution_phases
```

## Example

```bash
# Basic usage
/user:generate-multi-agent-tasks tasks/tasks-list.md

# Specify 4 agents
/user:generate-multi-agent-tasks tasks/tasks-list.md --agents 4

# Custom output directory
/user:generate-multi-agent-tasks tasks/tasks-list.md --output-dir tasks/sprint-1/
```

## Output Formats

### Individual Agent Task Files (agent-[int:]-tasks.md)

Each agent gets their own task file following this structure:

```markdown
# Agent Tasks: [Role Name]

## Agent Role

**Primary Focus:** [Brief description of agent's main responsibility]

## Key Responsibilities

- [Responsibility 1]
- [Responsibility 2]
- [Responsibility 3]

## Assigned Tasks

### From Original Task List

- [ ] [Parent Task X] - [Originally task 1.0 from main list]
  - [ ] [Sub-task X.1] - [Originally task 1.1 from main list]
  - [ ] [Sub-task X.2] - [Originally task 1.2 from main list]
- [ ] [Parent Task Y] - [Originally task 3.0 from main list]
  - [ ] [Sub-task Y.1] - [Originally task 3.1 from main list]

## Relevant Files

- `path/to/file1.ts` - [Description and purpose for this agent]
- `path/to/file1.test.ts` - Unit tests for file1.ts
- `path/to/file2.tsx` - [Description and purpose for this agent]

## Dependencies

### Prerequisites (What this agent needs before starting)

- **From [Other Agent Role]:** [Specific deliverable needed]
- **From [Another Agent Role]:** [Specific deliverable needed]

### Provides to Others (What this agent delivers)

- **To [Other Agent Role]:** [Specific deliverable provided]
- **To [Another Agent Role]:** [Specific deliverable provided]

## Handoff Points

- **After Task [X]:** Notify [Other Agent Role] that [specific component/API/file] is ready
- **Before Task [Y]:** Wait for confirmation from [Other Agent Role] that [dependency] is complete

## Testing Responsibilities

- Unit tests for all files owned by this agent
- Integration testing coordination with [specific other agents]
- [Any specific testing requirements]

## Notes

- Follow existing code conventions in the codebase
- Coordinate with other agents at specified handoff points
```

### Coordination Overview File (multi-agent-coordination.md)

````markdown
# Multi-Agent Coordination: [Feature Name]

## Agent Overview

### Agent Count: [3-5]

**Rationale:** [Brief explanation of why this number of agents was chosen]

### Agent Roles

1. **[Agent Role 1]:** [Brief description]
2. **[Agent Role 2]:** [Brief description]
3. **[Agent Role 3]:** [Brief description]
   [Continue for all agents...]

## Task Distribution Summary

### Original Task List Breakdown

- **[Agent Role 1]:** Tasks [list of task numbers from original]
- **[Agent Role 2]:** Tasks [list of task numbers from original]
- **[Agent Role 3]:** Tasks [list of task numbers from original]

## Critical Dependencies

### Sequential Dependencies (must happen in order)

1. **[Agent X] → [Agent Y]:** [Task/deliverable] must complete before [Task/deliverable] can start
2. **[Agent Y] → [Agent Z]:** [Task/deliverable] must complete before [Task/deliverable] can start

### Parallel Opportunities

- **Phase 1:** [Agent X] and [Agent Y] can work simultaneously on [tasks]
- **Phase 2:** [Agent Y] and [Agent Z] can work simultaneously on [tasks]

## Integration Milestones

1. **[Milestone Name]:** [Agents involved] - [Description and success criteria]
2. **[Milestone Name]:** [Agents involved] - [Description and success criteria]

## Communication Protocol

- **Daily Check-ins:** All agents report progress on dependencies
- **Handoff Notifications:** Agents must confirm completion of deliverables
- **Issue Escalation:** [Process for resolving blocking issues]

## Shared Resources

- **[File/Component Name]:** Agents [X, Y] - [Coordination requirements]
- **[Another Resource]:** Agents [Y, Z] - [Coordination requirements]

```



## Interaction Model

The process explicitly requires a pause after generating agent roles and high-level distribution to get user confirmation ("Go") before proceeding to generate detailed tasks. This ensures the multi-agent strategy aligns with user expectations before diving into specifics.

## Coordination File Contents

The `multi-agent-coordination.md` includes:
- Agent overview and rationale
- Task distribution summary
- Critical dependencies (sequential and parallel)
- Integration milestones
- Communication protocol
- Shared resources

## Notes

- Task distribution optimizes for parallel execution while respecting dependencies
- Each agent should have roughly balanced workload
- Integration points are clearly marked for smooth handoffs
- All task references maintain traceability to original task list

## Target Audience

Assume the primary readers are **multiple junior developers** working as a coordinated team, each taking on one agent role. Tasks should be clear enough for independent execution while maintaining coordination points.
```
````
