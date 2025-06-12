# continue-tasks

Resume an interrupted workflow from a saved agent context, continuing from the last completed task.

## Arguments

- `agent-id` (optional): The agent identifier to resume (e.g., "1", "2", "3", "4"). If not provided, attempts to detect from current context.
- `task-file` (optional): Path to the specific task file to resume. Defaults to `tasks/agent-{id}-tasks.md`.
- `context-file` (optional): Path to the agent context file. Defaults to `tasks/agent-contexts/agent-{id}-context.md`.

## Usage

```bash
/user:tasks:continue 2
/user:tasks:continue agent-id=3 task-file=tasks/custom-tasks.md
/user:tasks:continue context-file=tasks/agent-contexts/saved-context.md
```

## Workflow

1. **Load Context**: Read the saved agent context from the specified or default context file
2. **Parse Task Status**: Analyze the task list to identify completed [x] and incomplete [ ] tasks
3. **Find Resume Point**: Locate the last completed subtask and the next uncompleted task
4. **Restore Environment**: Set up the working environment based on saved context
5. **Continue Execution**: Resume task execution from the identified point
6. **Update Progress**: Mark tasks as complete and commit changes as work progresses
7. **Save Context**: Periodically save updated context for future resumption

## Implementation

```bash
#!/bin/bash

# Parse arguments
AGENT_ID=""
TASK_FILE=""
CONTEXT_FILE=""

# Parse named arguments
IFS=' ' read -ra ARGS <<< "$ARGUMENTS"
for arg in "${ARGS[@]}"; do
    if [[ "$arg" =~ ^[0-9]+$ ]]; then
        AGENT_ID="$arg"
    elif [[ "$arg" =~ ^agent-id=(.+)$ ]]; then
        AGENT_ID="${BASH_REMATCH[1]}"
    elif [[ "$arg" =~ ^task-file=(.+)$ ]]; then
        TASK_FILE="${BASH_REMATCH[1]}"
    elif [[ "$arg" =~ ^context-file=(.+)$ ]]; then
        CONTEXT_FILE="${BASH_REMATCH[1]}"
    fi
done

# Set defaults if not provided
if [[ -z "$AGENT_ID" && -z "$CONTEXT_FILE" ]]; then
    # Try to detect agent ID from current context
    if [[ -f "tasks/current-agent.txt" ]]; then
        AGENT_ID=$(cat tasks/current-agent.txt)
    else
        echo "Error: No agent ID provided and could not detect from context"
        echo "Usage: /user:continue-tasks [agent-id] [task-file=path] [context-file=path]"
        exit 1
    fi
fi

if [[ -z "$TASK_FILE" && -n "$AGENT_ID" ]]; then
    TASK_FILE="tasks/agent-${AGENT_ID}-tasks.md"
fi

if [[ -z "$CONTEXT_FILE" && -n "$AGENT_ID" ]]; then
    CONTEXT_FILE="tasks/agent-contexts/agent-${AGENT_ID}-context.md"
fi

# Validate files exist
if [[ ! -f "$TASK_FILE" ]]; then
    echo "Error: Task file not found: $TASK_FILE"
    exit 1
fi

if [[ ! -f "$CONTEXT_FILE" ]]; then
    echo "Error: Context file not found: $CONTEXT_FILE"
    echo "Creating new context file..."
    mkdir -p "$(dirname "$CONTEXT_FILE")"
    cat > "$CONTEXT_FILE" << EOF
# Agent $AGENT_ID Context

## Current State
- Agent ID: $AGENT_ID
- Task File: $TASK_FILE
- Last Updated: $(date)

## Environment
- Working Directory: $(pwd)
- Git Branch: $(git branch --show-current 2>/dev/null || echo "not in git repo")

## Progress Summary
Starting fresh context for Agent $AGENT_ID
EOF
fi

# Read context
echo "Loading context from: $CONTEXT_FILE"
echo "Reading task list from: $TASK_FILE"
echo ""

# Find the last completed task and next uncompleted task
LAST_COMPLETED=""
NEXT_TASK=""
IN_PROGRESS=""

while IFS= read -r line; do
    if [[ "$line" =~ ^[[:space:]]*-[[:space:]]\[x\][[:space:]](.+) ]]; then
        LAST_COMPLETED="${BASH_REMATCH[1]}"
    elif [[ "$line" =~ ^[[:space:]]*-[[:space:]]\[\][[:space:]](.+) ]] && [[ -z "$NEXT_TASK" ]]; then
        NEXT_TASK="${BASH_REMATCH[1]}"
        # Check if this task was marked as in progress in context
        if grep -q "In Progress: $NEXT_TASK" "$CONTEXT_FILE" 2>/dev/null; then
            IN_PROGRESS="$NEXT_TASK"
        fi
    fi
done < "$TASK_FILE"

# Display status
echo "=== Workflow Resume Status ==="
echo "Agent: $AGENT_ID"
echo "Last Completed: ${LAST_COMPLETED:-None}"
echo "In Progress: ${IN_PROGRESS:-None}"
echo "Next Task: ${NEXT_TASK:-All tasks completed!}"
echo ""

# Update context with resume information
cat >> "$CONTEXT_FILE" << EOF

## Resume Point - $(date)
- Last Completed: ${LAST_COMPLETED:-None}
- Resuming Task: ${NEXT_TASK:-None}
- Status: Workflow resumed
EOF

# Save current agent ID for future detection
echo "$AGENT_ID" > tasks/current-agent.txt

# Generate resume prompt
if [[ -n "$NEXT_TASK" ]]; then
    echo "=== Resume Instructions ==="
    echo "Continue working on the following task:"
    echo "- $NEXT_TASK"
    echo ""
    echo "Context has been restored. You should:"
    echo "1. Review the task requirements"
    echo "2. Check any partially completed work"
    echo "3. Continue implementation from where you left off"
    echo "4. Mark the task complete with [x] when done"
    echo "5. Commit your changes after each subtask"
    echo "6. Monitor context usage and save state if needed"

    # Read any additional context notes
    if grep -q "## Notes" "$CONTEXT_FILE"; then
        echo ""
        echo "=== Previous Session Notes ==="
        sed -n '/## Notes/,/^##/p' "$CONTEXT_FILE" | grep -v "^##" | head -n -1
    fi
else
    echo "All tasks have been completed for Agent $AGENT_ID!"
    echo "Consider running /user:iterate-project to identify next steps."
fi

# Display task summary
echo ""
echo "=== Task Summary ==="
TOTAL_TASKS=$(grep -c "^[[:space:]]*-[[:space:]]\[.\]" "$TASK_FILE")
COMPLETED_TASKS=$(grep -c "^[[:space:]]*-[[:space:]]\[x\]" "$TASK_FILE")
echo "Progress: $COMPLETED_TASKS/$TOTAL_TASKS tasks completed"
echo "Completion: $(( COMPLETED_TASKS * 100 / TOTAL_TASKS ))%"
```

## Context File Format

The context file (`tasks/agent-contexts/agent-{id}-context.md`) should contain:

```markdown
# Agent {id} Context

## Current State

- Agent ID: {id}
- Task File: {path}
- Last Updated: {timestamp}
- Context Usage: {percentage}

## Environment

- Working Directory: {path}
- Git Branch: {branch}
- Modified Files: {list}

## Progress Summary

- Completed Tasks: {list}
- In Progress: {current task}
- Pending Tasks: {count}

## Notes

{Any important observations or blockers}

## Dependencies

- Waiting On: {list of dependencies}
- Provides To: {list of deliverables}
```

## Error Handling

- If no agent ID is provided and cannot be detected, displays usage instructions
- If task file doesn't exist, shows error message
- If context file doesn't exist, creates a new one with default structure
- Validates git repository status before attempting git operations
- Handles cases where all tasks are already completed

## Integration Points

- Works with task files generated by `/user:generate-multi-agent-tasks`
- Context files are updated by `/user:execute-multi-agent-tasks`
- Can be followed by `/user:iterate-project` when all tasks are complete
- Coordinates with other agents through `tasks/multi-agent-coordination.md`
