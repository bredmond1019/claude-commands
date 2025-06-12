# Context Manager Utilities

Reusable functions for saving and loading agent context across workflow sessions.

## Purpose

Provides standardized context management for multi-agent workflows, ensuring consistent state preservation and restoration across all agents.

## Context Structure

```bash
# Standard context file structure
CONTEXT_TEMPLATE='# Agent %s Context

## Current State
- Agent ID: %s
- Task File: %s
- Last Updated: %s
- Context Usage: %s%%

## Environment
- Working Directory: %s
- Git Branch: %s
- Modified Files:
%s

## Progress Summary
- Completed Tasks: %d
- In Progress: %s
- Pending Tasks: %d

## Notes
%s

## Dependencies
- Waiting On: %s
- Provides To: %s

## Session History
%s
'
```

## Core Functions

### save_agent_context()

Saves the current agent state to a context file.

```bash
save_agent_context() {
    local agent_id="$1"
    local task_file="$2"
    local context_file="${3:-tasks/agent-contexts/agent-${agent_id}-context.md}"
    local context_usage="${4:-0}"
    local notes="${5:-}"
    
    # Ensure context directory exists
    mkdir -p "$(dirname "$context_file")"
    
    # Get environment info
    local working_dir=$(pwd)
    local git_branch=$(git branch --show-current 2>/dev/null || echo "not in git repo")
    local modified_files=$(git status --porcelain 2>/dev/null | awk '{print "  - " $2}' || echo "  - none")
    
    # Count tasks
    local completed_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[x\]" "$task_file" 2>/dev/null || echo 0)
    local total_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[.\]" "$task_file" 2>/dev/null || echo 0)
    local pending_tasks=$((total_tasks - completed_tasks))
    
    # Find current task
    local current_task=$(grep -m1 "^[[:space:]]*-[[:space:]]\[\][[:space:]]" "$task_file" 2>/dev/null | sed 's/.*\[\][[:space:]]//' || echo "none")
    
    # Get dependencies from coordination file
    local waiting_on=""
    local provides_to=""
    if [[ -f "tasks/multi-agent-coordination.md" ]]; then
        # Extract dependencies for this agent
        waiting_on=$(grep -A5 "Agent $agent_id" tasks/multi-agent-coordination.md | grep "Waiting On:" | sed 's/.*Waiting On: //' || echo "none")
        provides_to=$(grep -A5 "Agent $agent_id" tasks/multi-agent-coordination.md | grep "Provides To:" | sed 's/.*Provides To: //' || echo "none")
    fi
    
    # Get session history if context already exists
    local session_history=""
    if [[ -f "$context_file" ]]; then
        session_history=$(grep -A100 "## Session History" "$context_file" 2>/dev/null | tail -n +2 || echo "")
    fi
    
    # Append new session entry
    local new_session="
### Session $(date +%Y%m%d_%H%M%S)
- Started: $(date)
- Tasks Completed This Session: $completed_tasks
- Context Usage: ${context_usage}%
- Notes: ${notes:-none}"
    
    session_history="${session_history}${new_session}"
    
    # Write context file
    printf "$CONTEXT_TEMPLATE" \
        "$agent_id" \
        "$agent_id" \
        "$task_file" \
        "$(date)" \
        "$context_usage" \
        "$working_dir" \
        "$git_branch" \
        "$modified_files" \
        "$completed_tasks" \
        "$current_task" \
        "$pending_tasks" \
        "${notes:-none}" \
        "${waiting_on:-none}" \
        "${provides_to:-none}" \
        "$session_history" \
        > "$context_file"
    
    echo "Context saved to: $context_file"
}
```

### load_agent_context()

Loads and displays agent context from a saved file.

```bash
load_agent_context() {
    local context_file="$1"
    
    if [[ ! -f "$context_file" ]]; then
        echo "Error: Context file not found: $context_file"
        return 1
    fi
    
    # Extract key information
    local agent_id=$(grep "Agent ID:" "$context_file" | head -1 | sed 's/.*Agent ID: //')
    local task_file=$(grep "Task File:" "$context_file" | head -1 | sed 's/.*Task File: //')
    local last_updated=$(grep "Last Updated:" "$context_file" | head -1 | sed 's/.*Last Updated: //')
    local context_usage=$(grep "Context Usage:" "$context_file" | head -1 | sed 's/.*Context Usage: //')
    local current_task=$(grep "In Progress:" "$context_file" | head -1 | sed 's/.*In Progress: //')
    
    echo "=== Agent Context Loaded ==="
    echo "Agent ID: $agent_id"
    echo "Task File: $task_file"
    echo "Last Updated: $last_updated"
    echo "Context Usage: $context_usage"
    echo "Current Task: $current_task"
    echo ""
    
    # Export variables for use in calling script
    export LOADED_AGENT_ID="$agent_id"
    export LOADED_TASK_FILE="$task_file"
    export LOADED_CURRENT_TASK="$current_task"
}
```

### detect_context_usage()

Estimates current context usage based on conversation length.

```bash
detect_context_usage() {
    # This is a placeholder - actual implementation would need Claude API integration
    # For now, we'll use a simple heuristic based on time or task count
    local start_time="${1:-0}"
    local completed_tasks="${2:-0}"
    
    # Rough estimate: assume 10% context per hour or 5% per completed task
    local time_based=0
    if [[ -n "$start_time" ]] && [[ "$start_time" != "0" ]]; then
        local elapsed=$(( $(date +%s) - start_time ))
        local hours=$(( elapsed / 3600 ))
        time_based=$(( hours * 10 ))
    fi
    
    local task_based=$(( completed_tasks * 5 ))
    
    # Use the higher of the two estimates
    local usage=$(( time_based > task_based ? time_based : task_based ))
    
    # Cap at 95% to leave room for final operations
    if [[ $usage -gt 95 ]]; then
        usage=95
    fi
    
    echo "$usage"
}
```

### should_spawn_subagent()

Determines if a sub-agent should be spawned based on context usage.

```bash
should_spawn_subagent() {
    local context_usage="$1"
    local remaining_tasks="$2"
    local threshold="${3:-80}"  # Default 80% threshold
    
    if [[ $context_usage -ge $threshold ]] && [[ $remaining_tasks -gt 0 ]]; then
        return 0  # Yes, spawn sub-agent
    else
        return 1  # No, continue in current context
    fi
}
```

### create_subagent_handoff()

Creates a handoff document for spawning a sub-agent.

```bash
create_subagent_handoff() {
    local agent_id="$1"
    local task_file="$2"
    local context_file="$3"
    local handoff_file="tasks/agent-contexts/agent-${agent_id}-handoff-$(date +%Y%m%d_%H%M%S).md"
    
    cat > "$handoff_file" << EOF
# Sub-Agent Handoff for Agent $agent_id

## Parent Context
- Original Agent: $agent_id
- Context File: $context_file
- Task File: $task_file
- Handoff Time: $(date)

## Continuation Instructions

Continue working on the tasks in $task_file starting from where the parent agent left off.

### To resume:
\`\`\`bash
/user:continue-tasks agent-id=$agent_id task-file=$task_file context-file=$context_file
\`\`\`

## Remaining Tasks
$(grep "^[[:space:]]*-[[:space:]]\[\]" "$task_file")

## Important Notes from Parent
$(grep -A10 "## Notes" "$context_file" | tail -n +2 | head -n 10)

## Dependencies Status
$(grep -A5 "## Dependencies" "$context_file" | tail -n +2)
EOF
    
    echo "$handoff_file"
}
```

## Usage Examples

### Basic Context Save

```bash
# Source the utilities
source utils/context-manager.md

# Save context for agent 1
save_agent_context "1" "tasks/agent-1-tasks.md" "" "25" "Completed infrastructure setup"
```

### Context Load and Resume

```bash
# Load context
load_agent_context "tasks/agent-contexts/agent-2-context.md"

# Use loaded variables
echo "Resuming work on: $LOADED_CURRENT_TASK"
```

### Sub-Agent Spawning

```bash
# Check if sub-agent needed
context_usage=$(detect_context_usage "$START_TIME" "$COMPLETED_TASKS")
remaining=$(grep -c "^[[:space:]]*-[[:space:]]\[\]" "$TASK_FILE")

if should_spawn_subagent "$context_usage" "$remaining"; then
    handoff=$(create_subagent_handoff "$AGENT_ID" "$TASK_FILE" "$CONTEXT_FILE")
    echo "Context limit approaching. Created handoff: $handoff"
    echo "Spawn a new Claude instance and run:"
    echo "  /user:continue-tasks agent-id=$AGENT_ID"
fi
```

## Integration Notes

- All commands should source this file for consistent context management
- Context files are stored in `tasks/agent-contexts/` by default
- Context is automatically saved after each subtask completion
- Sub-agent handoffs maintain continuity across context boundaries