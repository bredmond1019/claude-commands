# Context Capacity Monitoring

Utilities for tracking and managing context usage to prevent overflow during task execution.

## Overview

Claude has context limits that need to be monitored during long-running tasks. This utility provides functions to track usage, warn about approaching limits, and trigger appropriate actions.

## Core Functions

### estimate_context_usage()

Estimates current context usage based on conversation history and loaded files.

```bash
estimate_context_usage() {
    local conversation_file="${1:-/tmp/claude_conversation.log}"
    local loaded_files="${2:-/tmp/claude_loaded_files.log}"
    
    # Base context (system prompts, etc.)
    local base_context=2000
    
    # Count conversation tokens (rough estimate: 1 token ≈ 4 characters)
    local conversation_chars=0
    if [[ -f "$conversation_file" ]]; then
        conversation_chars=$(wc -c < "$conversation_file")
    fi
    local conversation_tokens=$((conversation_chars / 4))
    
    # Count loaded file tokens
    local file_chars=0
    if [[ -f "$loaded_files" ]]; then
        while read -r file; do
            if [[ -f "$file" ]]; then
                file_chars=$((file_chars + $(wc -c < "$file")))
            fi
        done < "$loaded_files"
    fi
    local file_tokens=$((file_chars / 4))
    
    # Total usage
    local total_tokens=$((base_context + conversation_tokens + file_tokens))
    
    # Export for use in scripts
    export CONTEXT_USED=$total_tokens
    export CONTEXT_CONVERSATION=$conversation_tokens
    export CONTEXT_FILES=$file_tokens
    
    echo "$total_tokens"
}
```

### calculate_context_percentage()

Calculates context usage as a percentage of the limit.

```bash
calculate_context_percentage() {
    local used_tokens="${1:-$(estimate_context_usage)}"
    local max_tokens="${2:-200000}"  # Claude's approximate context limit
    
    local percentage=$(( (used_tokens * 100) / max_tokens ))
    
    # Export for use in scripts
    export CONTEXT_PERCENTAGE=$percentage
    export CONTEXT_REMAINING=$((max_tokens - used_tokens))
    
    echo "$percentage"
}
```

### monitor_context_threshold()

Monitors context usage and triggers actions at specific thresholds.

```bash
monitor_context_threshold() {
    local threshold="${1:-80}"  # Default 80% warning threshold
    local action="${2:-warn}"   # warn, pause, or stop
    
    local current_percentage=$(calculate_context_percentage)
    
    # Define threshold levels
    local warning_threshold=70
    local critical_threshold=85
    local stop_threshold=95
    
    # Check thresholds and take action
    if [[ $current_percentage -ge $stop_threshold ]]; then
        echo "🛑 CRITICAL: Context usage at ${current_percentage}% - Stopping execution"
        if [[ "$action" == "stop" ]]; then
            return 2
        fi
    elif [[ $current_percentage -ge $critical_threshold ]]; then
        echo "⚠️  WARNING: Context usage at ${current_percentage}% - Approaching limit"
        if [[ "$action" == "pause" ]]; then
            echo "Pausing for context management..."
            return 1
        fi
    elif [[ $current_percentage -ge $warning_threshold ]]; then
        echo "ℹ️  INFO: Context usage at ${current_percentage}% - Monitor closely"
    fi
    
    # Check against custom threshold
    if [[ $current_percentage -ge $threshold ]]; then
        echo "📊 Context usage (${current_percentage}%) exceeds threshold (${threshold}%)"
        return 1
    fi
    
    return 0
}
```

### generate_context_report()

Generates a detailed context usage report.

```bash
generate_context_report() {
    local output_file="${1:-/tmp/context_report.md}"
    
    # Get current usage
    estimate_context_usage > /dev/null
    calculate_context_percentage > /dev/null
    
    # Generate report
    cat > "$output_file" << EOF
# Context Usage Report

Generated: $(date)

## Summary
- **Total Usage:** $CONTEXT_USED tokens (${CONTEXT_PERCENTAGE}%)
- **Remaining:** $CONTEXT_REMAINING tokens
- **Status:** $(get_context_status)

## Breakdown
- **Base Context:** 2000 tokens
- **Conversation:** $CONTEXT_CONVERSATION tokens
- **Loaded Files:** $CONTEXT_FILES tokens

## Recommendations
$(generate_context_recommendations)

## Loaded Files
$(if [[ -f "/tmp/claude_loaded_files.log" ]]; then
    echo "Files currently in context:"
    while read -r file; do
        if [[ -f "$file" ]]; then
            size=$(wc -c < "$file")
            echo "- $file ($(numfmt --to=iec-i --suffix=B $size))"
        fi
    done < "/tmp/claude_loaded_files.log"
else
    echo "No file tracking data available"
fi)
EOF
    
    echo "Context report saved to: $output_file"
}
```

### track_file_loading()

Tracks files loaded into context for accurate monitoring.

```bash
track_file_loading() {
    local file="$1"
    local tracking_file="${2:-/tmp/claude_loaded_files.log}"
    
    # Add file to tracking if not already present
    if ! grep -q "^$file$" "$tracking_file" 2>/dev/null; then
        echo "$file" >> "$tracking_file"
    fi
    
    # Update context estimate
    estimate_context_usage > /dev/null
    
    # Check if loading this file exceeded limits
    if ! monitor_context_threshold 90 warn; then
        echo "⚠️  Warning: Loading $file may have exceeded safe context limits"
    fi
}
```

### optimize_context_usage()

Suggests optimizations to reduce context usage.

```bash
optimize_context_usage() {
    local suggestion_count=0
    
    echo "🔍 Analyzing context usage for optimization opportunities..."
    
    # Check for large files
    if [[ -f "/tmp/claude_loaded_files.log" ]]; then
        while read -r file; do
            if [[ -f "$file" ]]; then
                size=$(wc -c < "$file")
                if [[ $size -gt 50000 ]]; then  # Files over 50KB
                    echo "📦 Large file detected: $file ($(numfmt --to=iec-i --suffix=B $size))"
                    echo "   Consider: Loading only relevant sections"
                    suggestion_count=$((suggestion_count + 1))
                fi
            fi
        done < "/tmp/claude_loaded_files.log"
    fi
    
    # Check conversation length
    if [[ $CONTEXT_CONVERSATION -gt 50000 ]]; then
        echo "💬 Long conversation history: $CONTEXT_CONVERSATION tokens"
        echo "   Consider: Starting a new session for unrelated tasks"
        suggestion_count=$((suggestion_count + 1))
    fi
    
    # Check for duplicate file loads
    if [[ -f "/tmp/claude_loaded_files.log" ]]; then
        local duplicates=$(sort "/tmp/claude_loaded_files.log" | uniq -d)
        if [[ -n "$duplicates" ]]; then
            echo "🔄 Duplicate files loaded:"
            echo "$duplicates" | sed 's/^/   - /'
            echo "   Consider: Removing duplicate file references"
            suggestion_count=$((suggestion_count + 1))
        fi
    fi
    
    if [[ $suggestion_count -eq 0 ]]; then
        echo "✅ No immediate optimization opportunities found"
    else
        echo ""
        echo "💡 Found $suggestion_count optimization opportunities"
    fi
}
```

### spawn_context_handler()

Handles context overflow by spawning sub-agents or saving state.

```bash
spawn_context_handler() {
    local context_percentage=$(calculate_context_percentage)
    local task_file="$1"
    local current_task="$2"
    
    if [[ $context_percentage -ge 85 ]]; then
        echo "🚨 Context overflow imminent - Initiating handoff procedure"
        
        # Save current state
        local handoff_dir="tasks/agent-contexts"
        mkdir -p "$handoff_dir"
        
        local timestamp=$(date +%Y%m%d_%H%M%S)
        local handoff_file="$handoff_dir/handoff_${timestamp}.md"
        
        cat > "$handoff_file" << EOF
# Context Handoff Report

## Timestamp: $(date)
## Reason: Context usage at ${context_percentage}%

### Current Task
- File: $task_file
- Task: $current_task
- Status: In Progress

### Completed Tasks
$(grep "\[x\]" "$task_file" | tail -20)

### Remaining Tasks
$(grep "\[ \]" "$task_file" | head -10)

### Context Summary
$(generate_context_report /dev/stdout | grep -A5 "Summary")

### Resumption Instructions
1. Start new Claude instance
2. Load this handoff file
3. Continue from task: $current_task
4. Reference: $task_file

### Modified Files
$(git status --porcelain | grep -E "^[AM]" | cut -c4-)

### Notes
- Context was approaching limit (${context_percentage}%)
- State saved for seamless resumption
- All progress has been committed to git
EOF
        
        echo "✅ Handoff file created: $handoff_file"
        echo ""
        echo "To continue:"
        echo "1. Start new Claude instance"
        echo "2. Run: /user:continue-tasks --handoff $handoff_file"
        
        return 0
    fi
    
    return 1
}
```

## Helper Functions

### get_context_status()

Returns a status indicator based on context usage.

```bash
get_context_status() {
    local percentage=${CONTEXT_PERCENTAGE:-0}
    
    if [[ $percentage -lt 50 ]]; then
        echo "🟢 Healthy"
    elif [[ $percentage -lt 70 ]]; then
        echo "🟡 Moderate"
    elif [[ $percentage -lt 85 ]]; then
        echo "🟠 High"
    else
        echo "🔴 Critical"
    fi
}
```

### generate_context_recommendations()

Generates recommendations based on current usage.

```bash
generate_context_recommendations() {
    local percentage=${CONTEXT_PERCENTAGE:-0}
    
    if [[ $percentage -lt 50 ]]; then
        echo "- Context usage is healthy"
        echo "- Continue normal operations"
    elif [[ $percentage -lt 70 ]]; then
        echo "- Consider completing current task group before loading new files"
        echo "- Avoid loading large documentation files"
    elif [[ $percentage -lt 85 ]]; then
        echo "- Complete current task and prepare for handoff"
        echo "- Avoid loading any new files"
        echo "- Consider spawning sub-agent for remaining tasks"
    else
        echo "- IMMEDIATE ACTION REQUIRED"
        echo "- Save current state and spawn new agent"
        echo "- Do not load any additional files"
    fi
}
```

## Integration Examples

### With Task Execution

```bash
# Monitor context during task execution
execute_task_with_monitoring() {
    local task_file="$1"
    local task_line="$2"
    
    # Check context before starting
    if ! monitor_context_threshold 80 warn; then
        echo "Context usage high - consider handoff after this task"
    fi
    
    # Track any files loaded during task
    track_file_loading "$task_file"
    
    # Execute task
    execute_task "$task_file" "$task_line"
    
    # Check context after completion
    if ! monitor_context_threshold 85 pause; then
        spawn_context_handler "$task_file" "$task_line"
        return 1
    fi
    
    return 0
}
```

### Continuous Monitoring

```bash
# Background context monitor
start_context_monitor() {
    local interval="${1:-60}"  # Check every minute
    local threshold="${2:-80}"
    
    while true; do
        if ! monitor_context_threshold "$threshold" warn; then
            # Send notification or trigger action
            echo "$(date): Context threshold exceeded" >> /tmp/context_alerts.log
        fi
        sleep "$interval"
    done &
    
    echo "Context monitor started (PID: $!)"
}
```

## Configuration

```bash
# Set environment variables for customization
export CONTEXT_MAX_TOKENS=200000
export CONTEXT_WARNING_THRESHOLD=70
export CONTEXT_CRITICAL_THRESHOLD=85
export CONTEXT_TRACKING_ENABLED=true
export CONTEXT_AUTO_OPTIMIZE=true
```

## Best Practices

1. **Regular Monitoring**: Check context usage before and after major operations
2. **Proactive Management**: Plan for handoffs before reaching critical levels
3. **File Hygiene**: Unload files that are no longer needed
4. **State Preservation**: Always save state before context limits are reached
5. **Clear Documentation**: Document handoff points for smooth transitions