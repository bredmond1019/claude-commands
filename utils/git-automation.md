# Git Automation Utilities

Utilities for automated git commits after task completion with consistent formatting.

## Core Functions

### commit_task_completion()

Commits changes after completing a task with a standardized message format.

```bash
commit_task_completion() {
    local task_id="$1"
    local task_description="$2"
    local files_modified="$3"  # Space-separated list of files
    local agent_name="${4:-Agent}"  # Optional agent identifier
    
    # Stage the specified files
    if [[ -n "$files_modified" ]]; then
        git add $files_modified
    else
        echo "Warning: No files specified for commit"
        return 1
    fi
    
    # Check if there are changes to commit
    if ! git diff --cached --quiet; then
        # Generate commit message
        local commit_message="Complete task $task_id: $task_description

- Marked task $task_id as complete in task list
- Modified files: $(echo $files_modified | tr ' ' ', ')
- Agent: $agent_name

🤖 Generated with Claude Code (https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
        
        # Commit changes
        git commit -m "$commit_message"
        
        echo "✅ Committed task completion: $task_id"
        return 0
    else
        echo "No changes to commit for task $task_id"
        return 1
    fi
}
```

### commit_subtask_batch()

Commits multiple subtask completions in a single commit.

```bash
commit_subtask_batch() {
    local parent_task="$1"
    local subtasks_completed="$2"  # Comma-separated list
    local files_modified="$3"
    
    # Stage files
    git add $files_modified
    
    if ! git diff --cached --quiet; then
        # Parse subtask list
        local subtask_count=$(echo "$subtasks_completed" | tr ',' '\n' | wc -l)
        
        # Generate commit message
        local commit_message="Complete $subtask_count subtasks under $parent_task

Completed subtasks:
$(echo "$subtasks_completed" | tr ',' '\n' | sed 's/^/- /')

Modified files: $(echo $files_modified | tr ' ' ', ')

🤖 Generated with Claude Code (https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
        
        git commit -m "$commit_message"
        
        echo "✅ Committed $subtask_count subtask completions"
        return 0
    else
        echo "No changes to commit"
        return 1
    fi
}
```

### auto_commit_on_save()

Automatically commits when specific files are saved (useful for continuous progress tracking).

```bash
auto_commit_on_save() {
    local watch_file="$1"
    local task_context="$2"
    
    # Calculate file hash
    local current_hash=$(md5sum "$watch_file" | cut -d' ' -f1)
    local last_hash_file="/tmp/.git_auto_${watch_file//\//_}.hash"
    
    # Check if file changed
    if [[ -f "$last_hash_file" ]]; then
        local last_hash=$(cat "$last_hash_file")
        if [[ "$current_hash" == "$last_hash" ]]; then
            return 0  # No changes
        fi
    fi
    
    # File changed, check for task completions
    local new_completions=$(grep -n "\[x\]" "$watch_file" | 
                           grep -v -f <(git show HEAD:"$watch_file" 2>/dev/null | grep -n "\[x\]" || echo "") |
                           cut -d: -f1)
    
    if [[ -n "$new_completions" ]]; then
        # Extract task info for each new completion
        local task_info=""
        while read -r line_num; do
            local task_line=$(sed -n "${line_num}p" "$watch_file")
            task_info="${task_info}\n- Line $line_num: $task_line"
        done <<< "$new_completions"
        
        # Commit the changes
        git add "$watch_file"
        git commit -m "Progress update: $task_context

New completions:$task_info

🤖 Generated with Claude Code (https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
        
        echo "✅ Auto-committed progress update"
    fi
    
    # Update hash
    echo "$current_hash" > "$last_hash_file"
}
```

### create_atomic_commit()

Creates atomic commits with rollback capability.

```bash
create_atomic_commit() {
    local operation="$1"
    local files="$2"
    shift 2
    local command="$@"
    
    # Create a backup point
    local backup_branch="backup-$(date +%s)"
    git branch "$backup_branch" 2>/dev/null
    
    # Stage files
    git add $files
    
    # Execute the operation
    if eval "$command"; then
        # Operation succeeded, commit
        git commit -m "Atomic operation: $operation

Files modified: $(echo $files | tr ' ' ', ')
Operation completed successfully

🤖 Generated with Claude Code (https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
        
        # Clean up backup branch
        git branch -D "$backup_branch" 2>/dev/null
        
        echo "✅ Atomic commit successful"
        return 0
    else
        # Operation failed, rollback
        git reset --hard HEAD
        echo "❌ Operation failed, changes rolled back"
        
        # Keep backup branch for investigation
        echo "Backup branch kept at: $backup_branch"
        return 1
    fi
}
```

### generate_commit_message()

Generates standardized commit messages based on context.

```bash
generate_commit_message() {
    local commit_type="$1"  # feature, fix, docs, refactor, test, chore
    local scope="$2"        # Component or area affected
    local description="$3"
    local body="${4:-}"     # Optional detailed description
    local breaking="${5:-}" # Optional breaking change note
    
    # Generate conventional commit message
    local message="${commit_type}"
    
    if [[ -n "$scope" ]]; then
        message="${message}(${scope})"
    fi
    
    message="${message}: ${description}"
    
    if [[ -n "$body" ]]; then
        message="${message}\n\n${body}"
    fi
    
    if [[ -n "$breaking" ]]; then
        message="${message}\n\nBREAKING CHANGE: ${breaking}"
    fi
    
    # Add automation footer
    message="${message}\n\n🤖 Generated with Claude Code (https://claude.ai/code)\n\nCo-Authored-By: Claude <noreply@anthropic.com>"
    
    echo -e "$message"
}
```

### commit_with_validation()

Commits changes with pre-commit validation.

```bash
commit_with_validation() {
    local files="$1"
    local message="$2"
    
    # Stage files
    git add $files
    
    # Run validation checks
    local validation_passed=true
    
    # Check for merge conflicts
    if grep -r "<<<<<<< HEAD" $files 2>/dev/null; then
        echo "❌ Error: Merge conflicts detected"
        validation_passed=false
    fi
    
    # Check for debug statements
    if grep -r "console\.log\|debugger\|TODO" $files 2>/dev/null | grep -v "// *TODO"; then
        echo "⚠️  Warning: Debug statements or TODOs found"
        # This is a warning, not a failure
    fi
    
    # Check file sizes
    while read -r file; do
        if [[ -f "$file" ]]; then
            local size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
            if [[ $size -gt 1048576 ]]; then  # 1MB
                echo "❌ Error: File $file is too large (>1MB)"
                validation_passed=false
            fi
        fi
    done <<< "$(echo $files | tr ' ' '\n')"
    
    if [[ "$validation_passed" == "true" ]]; then
        git commit -m "$message"
        echo "✅ Commit successful with validation"
        return 0
    else
        git reset HEAD $files
        echo "❌ Commit aborted due to validation errors"
        return 1
    fi
}
```

## Integration Examples

### With Task Execution

```bash
# Source utilities
source utils/git-automation.md
source utils/task-parser.md

# Execute task and commit
execute_and_commit_task() {
    local task_file="$1"
    local task_line="$2"
    
    # Get task details
    local task_info=$(sed -n "${task_line}p" "$task_file")
    local task_id=$(echo "$task_info" | grep -o "[0-9]\+\.[0-9]\+")
    local task_desc=$(echo "$task_info" | sed 's/.*\] //')
    
    # Mark task complete
    mark_task_complete "$task_file" "$task_line"
    
    # Commit the change
    commit_task_completion "$task_id" "$task_desc" "$task_file"
}
```

### With Multi-Agent Coordination

```bash
# Commit with agent context
commit_agent_progress() {
    local agent_id="$1"
    local task_file="tasks/agent-${agent_id}-tasks.md"
    
    # Find completed tasks since last commit
    local last_commit_tasks=$(git show HEAD:"$task_file" 2>/dev/null | grep -c "\[x\]" || echo 0)
    local current_tasks=$(grep -c "\[x\]" "$task_file")
    local new_completions=$((current_tasks - last_commit_tasks))
    
    if [[ $new_completions -gt 0 ]]; then
        commit_with_validation "$task_file" "$(generate_commit_message \
            "feat" \
            "agent-$agent_id" \
            "Complete $new_completions tasks" \
            "Agent $agent_id has completed $new_completions additional tasks.\nTotal progress: $current_tasks tasks completed.")"
    fi
}
```

### Continuous Integration

```bash
# Set up file watcher for auto-commits
watch_task_progress() {
    local task_file="$1"
    local interval="${2:-5}"  # Check every 5 seconds by default
    
    echo "Watching $task_file for changes..."
    
    while true; do
        auto_commit_on_save "$task_file" "Task progress"
        sleep "$interval"
    done
}
```

## Best Practices

1. **Atomic Commits**: Each commit should represent one logical change
2. **Meaningful Messages**: Use descriptive commit messages that explain why, not what
3. **Regular Commits**: Commit after each subtask to maintain granular history
4. **Validation**: Run checks before committing to maintain code quality
5. **Agent Context**: Include agent information in multi-agent scenarios

## Configuration

Set these environment variables to customize behavior:

```bash
export GIT_AUTO_COMMIT_ENABLED=true
export GIT_AUTO_COMMIT_INTERVAL=300  # 5 minutes
export GIT_AUTO_COMMIT_BRANCH=main
export GIT_AUTO_COMMIT_PUSH=false    # Don't auto-push
```