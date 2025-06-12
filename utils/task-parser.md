# Task Parser Utilities

Utilities for parsing task lists, detecting completion status, and analyzing task progress.

## Purpose

Provides consistent task parsing and analysis functionality across all workflow commands, enabling accurate progress tracking and task completion detection.

## Core Functions

### parse_task_list()

Parses a task file and extracts task information with completion status.

```bash
parse_task_list() {
    local task_file="$1"
    local indent_level="${2:-all}"  # Optional: filter by indent level
    
    if [[ ! -f "$task_file" ]]; then
        echo "Error: Task file not found: $task_file"
        return 1
    fi
    
    # Arrays to store task data
    declare -a tasks=()
    declare -a statuses=()
    declare -a levels=()
    declare -a line_numbers=()
    
    local line_num=0
    while IFS= read -r line; do
        line_num=$((line_num + 1))
        
        # Match task lines with various indent levels
        if [[ "$line" =~ ^([[:space:]]*)-[[:space:]]\[(.)\][[:space:]](.+) ]]; then
            local indent="${BASH_REMATCH[1]}"
            local status="${BASH_REMATCH[2]}"
            local task_text="${BASH_REMATCH[3]}"
            local level=$(( ${#indent} / 2 ))  # Assuming 2 spaces per level
            
            # Apply indent level filter if specified
            if [[ "$indent_level" == "all" ]] || [[ "$level" -eq "$indent_level" ]]; then
                tasks+=("$task_text")
                statuses+=("$status")
                levels+=("$level")
                line_numbers+=("$line_num")
            fi
        fi
    done < "$task_file"
    
    # Export arrays for use in calling script
    export TASK_LIST=("${tasks[@]}")
    export TASK_STATUSES=("${statuses[@]}")
    export TASK_LEVELS=("${levels[@]}")
    export TASK_LINE_NUMBERS=("${line_numbers[@]}")
    
    # Return task count
    echo "${#tasks[@]}"
}
```

### get_task_stats()

Calculates statistics about task completion.

```bash
get_task_stats() {
    local task_file="$1"
    
    # Count different task states
    local total_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[.\]" "$task_file" 2>/dev/null || echo 0)
    local completed_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[x\]" "$task_file" 2>/dev/null || echo 0)
    local pending_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[\]" "$task_file" 2>/dev/null || echo 0)
    local cancelled_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[~\]" "$task_file" 2>/dev/null || echo 0)
    
    # Calculate percentages
    local completion_percent=0
    if [[ $total_tasks -gt 0 ]]; then
        completion_percent=$(( (completed_tasks * 100) / total_tasks ))
    fi
    
    # Output stats
    echo "Total Tasks: $total_tasks"
    echo "Completed: $completed_tasks"
    echo "Pending: $pending_tasks"
    echo "Cancelled: $cancelled_tasks"
    echo "Completion: ${completion_percent}%"
    
    # Export for use in scripts
    export TASK_STATS_TOTAL=$total_tasks
    export TASK_STATS_COMPLETED=$completed_tasks
    export TASK_STATS_PENDING=$pending_tasks
    export TASK_STATS_CANCELLED=$cancelled_tasks
    export TASK_STATS_PERCENT=$completion_percent
}
```

### find_next_task()

Finds the next uncompleted task in the list.

```bash
find_next_task() {
    local task_file="$1"
    local current_line="${2:-0}"  # Optional: start searching from specific line
    
    local line_num=0
    local found_next=false
    
    while IFS= read -r line; do
        line_num=$((line_num + 1))
        
        # Skip lines before current position
        if [[ $line_num -le $current_line ]]; then
            continue
        fi
        
        # Find first uncompleted task
        if [[ "$line" =~ ^[[:space:]]*-[[:space:]]\[\][[:space:]](.+) ]]; then
            local task_text="${BASH_REMATCH[1]}"
            echo "Line: $line_num"
            echo "Task: $task_text"
            echo "Full: $line"
            return 0
        fi
    done < "$task_file"
    
    # No uncompleted tasks found
    echo "No pending tasks found"
    return 1
}
```

### find_last_completed()

Finds the most recently completed task.

```bash
find_last_completed() {
    local task_file="$1"
    
    local last_completed_line=0
    local last_completed_task=""
    local last_completed_full=""
    local line_num=0
    
    while IFS= read -r line; do
        line_num=$((line_num + 1))
        
        # Track completed tasks
        if [[ "$line" =~ ^[[:space:]]*-[[:space:]]\[x\][[:space:]](.+) ]]; then
            last_completed_line=$line_num
            last_completed_task="${BASH_REMATCH[1]}"
            last_completed_full="$line"
        fi
    done < "$task_file"
    
    if [[ $last_completed_line -gt 0 ]]; then
        echo "Line: $last_completed_line"
        echo "Task: $last_completed_task"
        echo "Full: $last_completed_full"
        return 0
    else
        echo "No completed tasks found"
        return 1
    fi
}
```

### mark_task_complete()

Marks a specific task as complete in the file.

```bash
mark_task_complete() {
    local task_file="$1"
    local line_number="$2"
    
    if [[ ! -f "$task_file" ]]; then
        echo "Error: Task file not found: $task_file"
        return 1
    fi
    
    # Create a temporary file
    local temp_file="${task_file}.tmp"
    
    # Process the file line by line
    local current_line=0
    while IFS= read -r line; do
        current_line=$((current_line + 1))
        
        if [[ $current_line -eq $line_number ]]; then
            # Replace [ ] with [x] on this line
            if [[ "$line" =~ ^([[:space:]]*-[[:space:]])\[\]([[:space:]].+) ]]; then
                echo "${BASH_REMATCH[1]}[x]${BASH_REMATCH[2]}" >> "$temp_file"
            else
                echo "$line" >> "$temp_file"
            fi
        else
            echo "$line" >> "$temp_file"
        fi
    done < "$task_file"
    
    # Replace original file
    mv "$temp_file" "$task_file"
    echo "Marked task complete at line $line_number"
}
```

### get_task_hierarchy()

Analyzes parent-child relationships between tasks.

```bash
get_task_hierarchy() {
    local task_file="$1"
    local target_line="${2:-0}"  # Optional: get hierarchy for specific task
    
    # Parse all tasks with their levels
    parse_task_list "$task_file"
    
    if [[ $target_line -gt 0 ]]; then
        # Find parent of specific task
        local target_level=-1
        local target_index=-1
        
        # Find the target task
        for i in "${!TASK_LINE_NUMBERS[@]}"; do
            if [[ "${TASK_LINE_NUMBERS[$i]}" -eq "$target_line" ]]; then
                target_level="${TASK_LEVELS[$i]}"
                target_index=$i
                break
            fi
        done
        
        if [[ $target_level -ge 0 ]]; then
            # Find parent (previous task with lower level)
            for ((i=$target_index-1; i>=0; i--)); do
                if [[ "${TASK_LEVELS[$i]}" -lt "$target_level" ]]; then
                    echo "Parent Line: ${TASK_LINE_NUMBERS[$i]}"
                    echo "Parent Task: ${TASK_LIST[$i]}"
                    echo "Parent Status: ${TASK_STATUSES[$i]}"
                    break
                fi
            done
            
            # Find children (following tasks with higher level)
            echo "Children:"
            for ((i=$target_index+1; i<${#TASK_LINE_NUMBERS[@]}; i++)); do
                if [[ "${TASK_LEVELS[$i]}" -le "$target_level" ]]; then
                    break  # Reached end of children
                elif [[ "${TASK_LEVELS[$i]}" -eq $((target_level + 1)) ]]; then
                    echo "  - Line ${TASK_LINE_NUMBERS[$i]}: ${TASK_LIST[$i]} [${TASK_STATUSES[$i]}]"
                fi
            done
        fi
    else
        # Display full hierarchy
        echo "Task Hierarchy:"
        for i in "${!TASK_LIST[@]}"; do
            local indent=""
            for ((j=0; j<${TASK_LEVELS[$i]}; j++)); do
                indent="  $indent"
            done
            echo "${indent}- [${TASK_STATUSES[$i]}] ${TASK_LIST[$i]} (line ${TASK_LINE_NUMBERS[$i]})"
        done
    fi
}
```

### validate_task_dependencies()

Checks if task dependencies are satisfied.

```bash
validate_task_dependencies() {
    local task_file="$1"
    local coordination_file="${2:-tasks/multi-agent-coordination.md}"
    
    # Parse task list
    parse_task_list "$task_file"
    
    local has_issues=false
    
    # Check each incomplete task
    for i in "${!TASK_STATUSES[@]}"; do
        if [[ "${TASK_STATUSES[$i]}" == " " ]]; then
            local task="${TASK_LIST[$i]}"
            
            # Check if task mentions waiting on something
            if [[ "$task" =~ waiting[[:space:]]on|depends[[:space:]]on|after|requires ]]; then
                echo "Warning: Task at line ${TASK_LINE_NUMBERS[$i]} has potential dependency:"
                echo "  $task"
                has_issues=true
            fi
        fi
    done
    
    # Check coordination file for dependencies
    if [[ -f "$coordination_file" ]]; then
        # Extract agent dependencies
        local waiting_on=$(grep -A2 "Waiting On:" "$coordination_file" | tail -n +2 | head -n 1)
        if [[ -n "$waiting_on" ]] && [[ "$waiting_on" != "none" ]]; then
            echo ""
            echo "Coordination dependencies found:"
            echo "  $waiting_on"
            has_issues=true
        fi
    fi
    
    if [[ "$has_issues" == "false" ]]; then
        echo "No dependency issues found"
    fi
    
    return 0
}
```

## Usage Examples

### Basic Task Parsing

```bash
# Source the utilities
source utils/task-parser.md

# Parse task list
task_count=$(parse_task_list "tasks/agent-1-tasks.md")
echo "Found $task_count tasks"

# Access parsed data
for i in "${!TASK_LIST[@]}"; do
    echo "Task: ${TASK_LIST[$i]}"
    echo "Status: ${TASK_STATUSES[$i]}"
    echo "Level: ${TASK_LEVELS[$i]}"
    echo "Line: ${TASK_LINE_NUMBERS[$i]}"
    echo ""
done
```

### Progress Tracking

```bash
# Get task statistics
get_task_stats "tasks/agent-2-tasks.md"

# Use exported stats
if [[ $TASK_STATS_PERCENT -ge 100 ]]; then
    echo "All tasks completed!"
elif [[ $TASK_STATS_PERCENT -ge 80 ]]; then
    echo "Almost done - $TASK_STATS_PENDING tasks remaining"
fi
```

### Task Navigation

```bash
# Find next task to work on
if find_next_task "tasks/agent-3-tasks.md"; then
    # Task info is printed by the function
    # Can also parse the output
fi

# Find last completed task
if find_last_completed "tasks/agent-3-tasks.md"; then
    echo "Resuming after last completed task"
fi
```

### Task Completion

```bash
# Mark a specific task as complete
mark_task_complete "tasks/agent-1-tasks.md" 25

# Mark current task complete after finding it
next_task_output=$(find_next_task "tasks/agent-1-tasks.md")
if [[ $? -eq 0 ]]; then
    line_num=$(echo "$next_task_output" | grep "Line:" | cut -d' ' -f2)
    mark_task_complete "tasks/agent-1-tasks.md" "$line_num"
fi
```

## Integration Notes

- All task files should follow the standard markdown checklist format
- Task completion is tracked with [x], pending with [ ], cancelled with [~]
- Indent levels determine task hierarchy (2 spaces per level)
- Functions export variables for use in calling scripts
- Error handling included for missing files and invalid formats