# Error Handling Utilities

This file contains standardized error handling patterns and utilities for Claude Code commands.

## Core Error Handler

```bash
# Central error handling function
handle_error() {
    local error_code="$1"
    local error_message="$2"
    local recovery_suggestion="${3:-}"
    local command_name="${4:-command}"
    
    echo "==============================================="
    echo "ERROR in $command_name (Code: $error_code)"
    echo "==============================================="
    echo ""
    echo "Error: $error_message"
    
    if [[ -n "$recovery_suggestion" ]]; then
        echo ""
        echo "Suggestion: $recovery_suggestion"
    fi
    
    echo ""
    echo "For help, run: /user:$command_name --help"
    echo "==============================================="
    
    # Log error for debugging
    log_error "$error_code" "$error_message" "$command_name"
    
    exit "$error_code"
}
```

## Error Code Standards

```bash
# Standardized error codes across all commands
declare -A ERROR_CODES=(
    ["SUCCESS"]=0
    ["MISSING_ARGS"]=1
    ["FILE_NOT_FOUND"]=2
    ["PERMISSION_DENIED"]=3
    ["INVALID_FORMAT"]=4
    ["TASK_FAILED"]=5
    ["CONTEXT_OVERFLOW"]=6
    ["GIT_ERROR"]=7
    ["NETWORK_ERROR"]=8
    ["DEPENDENCY_ERROR"]=9
    ["UNKNOWN_ERROR"]=99
)

# Get error code by name
get_error_code() {
    local error_name="$1"
    echo "${ERROR_CODES[$error_name]:-99}"
}
```

## Specific Error Handlers

### Argument Errors
```bash
# Handle missing or invalid arguments
handle_argument_error() {
    local missing_arg="$1"
    local usage_example="$2"
    
    local message="Required argument '$missing_arg' not provided"
    local suggestion="Usage: $usage_example"
    
    handle_error "$(get_error_code MISSING_ARGS)" "$message" "$suggestion"
}

# Handle invalid argument values
handle_invalid_argument() {
    local arg_name="$1"
    local arg_value="$2"
    local expected_format="$3"
    
    local message="Invalid value for '$arg_name': '$arg_value'"
    local suggestion="Expected format: $expected_format"
    
    handle_error "$(get_error_code INVALID_FORMAT)" "$message" "$suggestion"
}
```

### File Operation Errors
```bash
# Handle file not found errors
handle_file_not_found() {
    local file_path="$1"
    local file_type="${2:-file}"
    
    local message="$file_type not found: $file_path"
    local suggestion="Check the file path and ensure the $file_type exists"
    
    # Try to find similar files
    local dir=$(dirname "$file_path")
    local filename=$(basename "$file_path")
    local similar_files=$(find "$dir" -maxdepth 1 -name "*${filename%.*}*" 2>/dev/null | head -3)
    
    if [[ -n "$similar_files" ]]; then
        suggestion="$suggestion. Similar files found:\n$similar_files"
    fi
    
    handle_error "$(get_error_code FILE_NOT_FOUND)" "$message" "$suggestion"
}

# Handle permission errors
handle_permission_error() {
    local file_path="$1"
    local operation="$2"  # read, write, execute
    
    local message="Permission denied: Cannot $operation $file_path"
    local suggestion="Check file permissions with: ls -l $file_path"
    
    handle_error "$(get_error_code PERMISSION_DENIED)" "$message" "$suggestion"
}
```

### Task Execution Errors
```bash
# Handle task execution failures
handle_task_error() {
    local task_id="$1"
    local task_description="$2"
    local error_details="$3"
    
    local message="Task $task_id failed: $task_description"
    local suggestion="Error details: $error_details\nTry running with --debug for more information"
    
    # Save task state for recovery
    save_task_state "$task_id" "failed"
    
    handle_error "$(get_error_code TASK_FAILED)" "$message" "$suggestion"
}

# Handle context overflow
handle_context_overflow() {
    local current_task="$1"
    local context_file="$2"
    
    local message="Context capacity exceeded while processing: $current_task"
    local suggestion="State saved to: $context_file\nContinue with: /user:continue-tasks context_file=\"$context_file\""
    
    handle_error "$(get_error_code CONTEXT_OVERFLOW)" "$message" "$suggestion"
}
```

### Git Errors
```bash
# Handle git operation errors
handle_git_error() {
    local operation="$1"
    local error_output="$2"
    
    local message="Git $operation failed"
    local suggestion="Git error: $error_output\nCheck git status and resolve any conflicts"
    
    handle_error "$(get_error_code GIT_ERROR)" "$message" "$suggestion"
}
```

## Error Logging

```bash
# Log errors for debugging and analysis
log_error() {
    local error_code="$1"
    local error_message="$2"
    local command_name="$3"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    local log_dir="$HOME/.claude/logs"
    mkdir -p "$log_dir"
    
    local log_file="$log_dir/errors.log"
    
    echo "[$timestamp] $command_name (Code: $error_code): $error_message" >> "$log_file"
}

# Read recent errors
show_recent_errors() {
    local count="${1:-10}"
    local log_file="$HOME/.claude/logs/errors.log"
    
    if [[ -f "$log_file" ]]; then
        echo "Recent errors (last $count):"
        tail -n "$count" "$log_file"
    else
        echo "No error log found"
    fi
}
```

## Recovery Mechanisms

### State Preservation
```bash
# Save state before critical operations
save_state() {
    local state_name="$1"
    local state_data="$2"
    
    local state_dir="$HOME/.claude/state"
    mkdir -p "$state_dir"
    
    local state_file="$state_dir/${state_name}_$(date +%s).json"
    echo "$state_data" > "$state_file"
    
    echo "State saved: $state_file"
}

# Restore from saved state
restore_state() {
    local state_name="$1"
    local state_dir="$HOME/.claude/state"
    
    # Find most recent state file
    local latest_state=$(ls -t "$state_dir/${state_name}_"*.json 2>/dev/null | head -1)
    
    if [[ -f "$latest_state" ]]; then
        cat "$latest_state"
        return 0
    else
        echo "No saved state found for: $state_name"
        return 1
    fi
}
```

### Rollback Support
```bash
# Create rollback point
create_rollback_point() {
    local rollback_name="$1"
    local files_to_backup="$2"  # Space-separated list
    
    local rollback_dir="$HOME/.claude/rollback/$rollback_name"
    mkdir -p "$rollback_dir"
    
    # Backup specified files
    for file in $files_to_backup; do
        if [[ -f "$file" ]]; then
            cp "$file" "$rollback_dir/"
            echo "Backed up: $file"
        fi
    done
    
    # Save rollback metadata
    echo "Created: $(date)" > "$rollback_dir/metadata.txt"
    echo "Files: $files_to_backup" >> "$rollback_dir/metadata.txt"
}

# Perform rollback
rollback() {
    local rollback_name="$1"
    local rollback_dir="$HOME/.claude/rollback/$rollback_name"
    
    if [[ ! -d "$rollback_dir" ]]; then
        echo "ERROR: No rollback point found: $rollback_name"
        return 1
    fi
    
    # Restore files
    for file in "$rollback_dir"/*; do
        if [[ -f "$file" ]] && [[ "$(basename "$file")" != "metadata.txt" ]]; then
            local original_name=$(basename "$file")
            cp "$file" "./$original_name"
            echo "Restored: $original_name"
        fi
    done
    
    echo "Rollback completed from: $rollback_name"
}
```

## Graceful Degradation

```bash
# Try operation with fallback options
try_with_fallback() {
    local primary_command="$1"
    local fallback_command="$2"
    local error_message="$3"
    
    # Try primary command
    if eval "$primary_command" 2>/dev/null; then
        return 0
    fi
    
    # Primary failed, try fallback
    echo "Primary method failed, trying fallback..."
    if eval "$fallback_command" 2>/dev/null; then
        echo "Fallback succeeded"
        return 0
    fi
    
    # Both failed
    handle_error "$(get_error_code UNKNOWN_ERROR)" "$error_message" "Both primary and fallback methods failed"
}
```

## Error Prevention

### Validation Wrapper
```bash
# Wrap operations with validation
safe_operation() {
    local operation_name="$1"
    local validation_func="$2"
    local operation_func="$3"
    shift 3
    local args=("$@")
    
    echo "Validating $operation_name..."
    if ! $validation_func "${args[@]}"; then
        handle_error "$(get_error_code INVALID_FORMAT)" "Validation failed for $operation_name"
    fi
    
    echo "Executing $operation_name..."
    if ! $operation_func "${args[@]}"; then
        handle_error "$(get_error_code TASK_FAILED)" "$operation_name failed"
    fi
    
    echo "$operation_name completed successfully"
}
```

### Timeout Protection
```bash
# Execute with timeout to prevent hanging
execute_with_timeout() {
    local timeout_seconds="$1"
    local command="$2"
    local timeout_message="${3:-Operation timed out}"
    
    # Execute command with timeout
    timeout "$timeout_seconds" bash -c "$command"
    local exit_code=$?
    
    if [[ $exit_code -eq 124 ]]; then
        handle_error "$(get_error_code TASK_FAILED)" "$timeout_message" "Operation exceeded ${timeout_seconds}s timeout"
    elif [[ $exit_code -ne 0 ]]; then
        handle_error "$exit_code" "Command failed: $command"
    fi
}
```

## Usage in Commands

```markdown
**ERROR HANDLING:**
Implement comprehensive error handling:

1. **Argument Validation**
   - Use handle_argument_error() for missing arguments
   - Use handle_invalid_argument() for incorrect values

2. **File Operations**
   - Use handle_file_not_found() with helpful suggestions
   - Use handle_permission_error() for access issues

3. **Task Execution**
   - Create rollback points before critical operations
   - Save state regularly for recovery
   - Use handle_context_overflow() when approaching limits

4. **Recovery**
   - Provide clear recovery instructions in error messages
   - Save enough state to resume operations
   - Implement rollback for reversible operations

Example:
if ! validate_file_exists "$input_file"; then
    handle_file_not_found "$input_file" "input specification"
fi
```