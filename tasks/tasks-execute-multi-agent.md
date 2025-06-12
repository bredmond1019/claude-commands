# /user:execute-multi-agent-tasks

Execute tasks in parallel across multiple Claude agents based on task distribution.

## Usage

```
/user:tasks:execute-multi-agent [--coordination-file <path>] [--agents <list>] [--mode <mode>]
```

## Arguments

- `--coordination-file <path>`: Path to coordination file (default: `tasks/multi-agent-coordination.md`)
- `--agents <list>`: Comma-separated list of agent IDs to execute (default: all)
- `--mode <mode>`: Execution mode - `parallel` or `sequential` (default: parallel)

## Process

1. **Load Coordination File**: Parse multi-agent coordination to understand task distribution
2. **Generate Agent Prompts**: Create specific prompts for each agent based on their tasks
3. **Spawn Claude Instances**: Launch new Claude instances for parallel execution
4. **Monitor Progress**: Track agent progress through task files and coordination updates
5. **Coordinate Handoffs**: Manage dependencies and handoff points between agents
6. **Aggregate Results**: Collect and summarize completion status across all agents

## Execution Flow

```python
def execute_multi_agent_tasks(coordination_file, agent_list=None, mode='parallel'):
    # 1. Parse coordination file
    coordination = parse_coordination_file(coordination_file)
    agents = agent_list or coordination['agents'].keys()

    # 2. Check dependencies
    ready_agents = find_ready_agents(coordination, agents)

    # 3. Generate prompts for each agent
    agent_prompts = {}
    for agent_id in ready_agents:
        agent_prompts[agent_id] = generate_agent_prompt(
            agent_id,
            coordination['agents'][agent_id],
            coordination['dependencies']
        )

    # 4. Execute based on mode
    if mode == 'parallel':
        execute_parallel(agent_prompts)
    else:
        execute_sequential(agent_prompts)

    # 5. Monitor and coordinate
    monitor_multi_agent_progress(agents, coordination_file)
```

## Agent Prompt Generation

Each agent receives a customized prompt containing:

```markdown
You are Agent [ID]: [Role Name] responsible for [primary focus].

## Your Tasks

[List of assigned tasks from agent-specific task file]

## Dependencies

- Waiting on: [Other agents and specific deliverables]
- Others waiting on you: [Agents depending on your deliverables]

## Key Context

- Project: [Project name and description]
- Your scope: [Specific areas of responsibility]
- Coordination file: [Path to multi-agent-coordination.md]

## Instructions

1. Work through your assigned tasks in order
2. Update task completion status in your task file
3. Commit changes after each subtask
4. Check coordination file for dependency updates
5. Mark handoff points when reached

For each task:

- Mark complete with [x] when finished
- Commit with descriptive message
- Note any blockers in tasks/blockers.md
```

## Claude Instance Spawning

```bash
spawn_claude_agent() {
    local agent_id="$1"
    local prompt_file="$2"
    local working_dir="$3"

    # Generate launch command
    local launch_cmd="claude"

    # Add project-specific flags
    launch_cmd="$launch_cmd --project \"$working_dir\""
    launch_cmd="$launch_cmd --name \"Agent-$agent_id\""

    # Set context from prompt file
    launch_cmd="$launch_cmd --prompt \"@$prompt_file\""

    # Enable automation features
    launch_cmd="$launch_cmd --auto-commit"
    launch_cmd="$launch_cmd --watch-files \"tasks/agent-${agent_id}-tasks.md\""

    # Launch in background
    echo "Launching Agent $agent_id..."
    eval "$launch_cmd" &

    # Store PID for monitoring
    echo "$!" > "/tmp/claude_agent_${agent_id}.pid"
}
```

## Progress Monitoring

```python
def monitor_multi_agent_progress(agents, coordination_file):
    monitoring = True
    last_status = {}

    while monitoring:
        current_status = {}
        all_complete = True

        for agent_id in agents:
            # Check task file progress
            task_file = f"tasks/agent-{agent_id}-tasks.md"
            stats = get_task_stats(task_file)
            current_status[agent_id] = stats

            if stats['completion'] < 100:
                all_complete = False

            # Check for changes
            if agent_id in last_status:
                if stats != last_status[agent_id]:
                    print(f"Agent {agent_id}: {stats['completed']}/{stats['total']} tasks complete")

        # Update coordination status
        update_coordination_status(coordination_file, current_status)

        # Check for completion
        if all_complete:
            print("All agents have completed their tasks!")
            monitoring = False

        last_status = current_status
        time.sleep(30)  # Check every 30 seconds
```

## Dependency Management

```bash
check_agent_dependencies() {
    local agent_id="$1"
    local coordination_file="$2"

    # Extract dependencies for this agent
    local deps=$(grep -A5 "Agent $agent_id.*Prerequisites" "$coordination_file" |
                 grep "From.*:" |
                 sed 's/.*From \(.*\):.*/\1/')

    local all_ready=true

    for dep in $deps; do
        # Check if dependency is marked complete
        if ! grep -q "$dep.*✅" "$coordination_file"; then
            echo "Agent $agent_id waiting on: $dep"
            all_ready=false
        fi
    done

    if [[ "$all_ready" == "true" ]]; then
        echo "Agent $agent_id dependencies satisfied"
        return 0
    else
        return 1
    fi
}
```

## Coordination File Updates

The command automatically updates the coordination file with:

- Agent progress percentages
- Completed milestones
- Active blockers
- Handoff completions

Example update:

```markdown
## Agent Status (Last Updated: 2024-01-15 10:30:00)

### Progress

- **Agent 1**: 8/10 tasks (80%) ✅ Milestone: Foundation Complete
- **Agent 2**: 5/12 tasks (42%) 🚧 Active
- **Agent 3**: 0/8 tasks (0%) ⏸️ Waiting on Agent 1
- **Agent 4**: 2/6 tasks (33%) 🚧 Active

### Recent Handoffs

- ✅ Agent 1 → Agent 2: API templates ready (10:15)
- ✅ Agent 1 → Agent 3: Coordination spec complete (10:20)
```

## Error Handling

- **Agent Failure**: Detect crashed agents and offer restart options
- **Dependency Deadlock**: Identify circular dependencies and alert user
- **File Conflicts**: Detect and resolve git conflicts between agents
- **Context Overflow**: Monitor agent context usage and trigger handoffs

## Example Usage

```bash
# Execute all agents in parallel (default)
/user:execute-multi-agent-tasks

# Execute specific agents only
/user:execute-multi-agent-tasks --agents 1,3

# Sequential execution for testing
/user:execute-multi-agent-tasks --mode sequential

# Custom coordination file
/user:execute-multi-agent-tasks --coordination-file tasks/sprint-1/coordination.md
```

## Integration Features

### Real-time Dashboard

```
┌─────────────────────────────────────┐
│ Multi-Agent Execution Status        │
├─────────────────────────────────────┤
│ Agent 1: ████████░░ 80% (8/10)     │
│ Agent 2: ████░░░░░░ 42% (5/12)     │
│ Agent 3: ░░░░░░░░░░ 0%  (waiting)  │
│ Agent 4: ███░░░░░░░ 33% (2/6)      │
├─────────────────────────────────────┤
│ Overall: ███████░░░ 40% (15/36)    │
│ Time Elapsed: 00:45:32              │
└─────────────────────────────────────┘
```

### Slack/Discord Integration

- Send progress updates to team channels
- Alert on milestone completions
- Notify on blocker issues

### Auto-scaling

- Spawn additional agents for lagging tasks
- Redistribute work from completed agents
- Balance workload dynamically

## Notes

- Requires Claude CLI with multi-instance support
- Agents operate in isolated contexts
- Coordination through file system ensures consistency
- Git serves as the source of truth for progress
- Monitor system resources when running many agents
