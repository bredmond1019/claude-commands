# Claude Instance Spawning Utilities

Utilities for spawning and managing multiple Claude instances for parallel task execution.

## Core Functions

### spawn_claude_instance()

Spawns a new Claude instance with specific configuration.

```bash
spawn_claude_instance() {
    local agent_id="$1"
    local prompt_file="$2"
    local working_dir="${3:-$(pwd)}"
    local options="${4:-}"
    
    # Validate inputs
    if [[ -z "$agent_id" ]] || [[ -z "$prompt_file" ]]; then
        echo "Error: agent_id and prompt_file are required"
        return 1
    fi
    
    if [[ ! -f "$prompt_file" ]]; then
        echo "Error: Prompt file not found: $prompt_file"
        return 1
    fi
    
    # Build Claude command
    local claude_cmd="claude"
    
    # Add instance identification
    claude_cmd="$claude_cmd --instance-name \"Agent-${agent_id}\""
    
    # Set working directory
    claude_cmd="$claude_cmd --project-dir \"$working_dir\""
    
    # Load prompt
    claude_cmd="$claude_cmd --initial-prompt \"@$prompt_file\""
    
    # Add monitoring and automation flags
    claude_cmd="$claude_cmd --auto-save"
    claude_cmd="$claude_cmd --auto-commit"
    claude_cmd="$claude_cmd --context-monitor"
    
    # Add custom options
    if [[ -n "$options" ]]; then
        claude_cmd="$claude_cmd $options"
    fi
    
    # Launch instance
    echo "🚀 Launching Claude Agent $agent_id..."
    echo "Command: $claude_cmd"
    
    # Execute in background and capture PID
    eval "$claude_cmd > \"/tmp/claude_agent_${agent_id}.log\" 2>&1 &"
    local pid=$!
    
    # Save instance info
    save_instance_info "$agent_id" "$pid" "$working_dir"
    
    # Wait for instance to initialize
    sleep 3
    
    # Verify instance started
    if kill -0 "$pid" 2>/dev/null; then
        echo "✅ Agent $agent_id started successfully (PID: $pid)"
        return 0
    else
        echo "❌ Failed to start Agent $agent_id"
        return 1
    fi
}
```

### save_instance_info()

Saves instance information for management and monitoring.

```bash
save_instance_info() {
    local agent_id="$1"
    local pid="$2"
    local working_dir="$3"
    
    local instance_dir="/tmp/claude_instances"
    mkdir -p "$instance_dir"
    
    local instance_file="$instance_dir/agent_${agent_id}.info"
    
    cat > "$instance_file" << EOF
AGENT_ID=$agent_id
PID=$pid
WORKING_DIR=$working_dir
START_TIME=$(date +%s)
STATUS=running
LOG_FILE=/tmp/claude_agent_${agent_id}.log
TASK_FILE=tasks/agent-${agent_id}-tasks.md
EOF
    
    # Also save to a central registry
    echo "$agent_id:$pid:$(date +%s)" >> "$instance_dir/registry.txt"
}
```

### spawn_multiple_agents()

Spawns multiple agents based on coordination file.

```bash
spawn_multiple_agents() {
    local coordination_file="${1:-tasks/multi-agent-coordination.md}"
    local agent_list="${2:-all}"
    
    # Parse coordination file to get agent count
    local total_agents=$(grep -c "^[0-9]\+\. \*\*.*Agent:" "$coordination_file" || echo 0)
    
    if [[ "$agent_list" == "all" ]]; then
        agent_list=$(seq 1 "$total_agents" | tr '\n' ' ')
    fi
    
    echo "🚀 Spawning multiple Claude agents..."
    echo "Agents to spawn: $agent_list"
    
    local spawned=0
    local failed=0
    
    for agent_id in $agent_list; do
        # Generate prompt for this agent
        local prompt_file="/tmp/agent_${agent_id}_prompt.md"
        generate_agent_prompt "$agent_id" "$coordination_file" > "$prompt_file"
        
        # Spawn the agent
        if spawn_claude_instance "$agent_id" "$prompt_file" "$(pwd)"; then
            spawned=$((spawned + 1))
        else
            failed=$((failed + 1))
        fi
        
        # Small delay between spawns
        sleep 2
    done
    
    echo ""
    echo "📊 Spawn Summary:"
    echo "   Successfully spawned: $spawned"
    echo "   Failed to spawn: $failed"
    
    if [[ $failed -eq 0 ]]; then
        return 0
    else
        return 1
    fi
}
```

### monitor_instances()

Monitors running Claude instances.

```bash
monitor_instances() {
    local instance_dir="/tmp/claude_instances"
    local update_interval="${1:-10}"
    
    echo "📊 Monitoring Claude instances (updating every ${update_interval}s)..."
    echo "Press Ctrl+C to stop monitoring"
    
    while true; do
        clear
        echo "=== Claude Instance Monitor ==="
        echo "Time: $(date)"
        echo ""
        
        # Check each registered instance
        if [[ -f "$instance_dir/registry.txt" ]]; then
            echo "Agent ID | PID   | Status    | CPU  | Memory | Tasks Complete"
            echo "---------|-------|-----------|------|--------|---------------"
            
            while IFS=: read -r agent_id pid start_time; do
                if [[ -f "$instance_dir/agent_${agent_id}.info" ]]; then
                    source "$instance_dir/agent_${agent_id}.info"
                    
                    # Check if process is running
                    local status="stopped"
                    local cpu="N/A"
                    local mem="N/A"
                    
                    if kill -0 "$pid" 2>/dev/null; then
                        status="running"
                        # Get resource usage (macOS compatible)
                        cpu=$(ps -p "$pid" -o %cpu | tail -1 | tr -d ' ')
                        mem=$(ps -p "$pid" -o rss | tail -1 | awk '{print int($1/1024)"M"}')
                    fi
                    
                    # Get task completion
                    local tasks_complete="N/A"
                    if [[ -f "$TASK_FILE" ]]; then
                        local completed=$(grep -c "\[x\]" "$TASK_FILE" || echo 0)
                        local total=$(grep -c "\[.\]" "$TASK_FILE" || echo 0)
                        tasks_complete="$completed/$total"
                    fi
                    
                    printf "%-8s | %-5s | %-9s | %-4s | %-6s | %s\n" \
                           "$agent_id" "$pid" "$status" "$cpu%" "$mem" "$tasks_complete"
                fi
            done < "$instance_dir/registry.txt"
        else
            echo "No instances registered"
        fi
        
        sleep "$update_interval"
    done
}
```

### stop_instance()

Stops a specific Claude instance gracefully.

```bash
stop_instance() {
    local agent_id="$1"
    local force="${2:-false}"
    
    local instance_file="/tmp/claude_instances/agent_${agent_id}.info"
    
    if [[ ! -f "$instance_file" ]]; then
        echo "Error: No instance found for Agent $agent_id"
        return 1
    fi
    
    source "$instance_file"
    
    if kill -0 "$PID" 2>/dev/null; then
        echo "Stopping Agent $agent_id (PID: $PID)..."
        
        if [[ "$force" == "true" ]]; then
            kill -9 "$PID"
            echo "Force stopped"
        else
            # Try graceful shutdown first
            kill -TERM "$PID"
            
            # Wait up to 10 seconds for graceful shutdown
            local count=0
            while kill -0 "$PID" 2>/dev/null && [[ $count -lt 10 ]]; do
                sleep 1
                count=$((count + 1))
            done
            
            if kill -0 "$PID" 2>/dev/null; then
                echo "Graceful shutdown failed, force stopping..."
                kill -9 "$PID"
            fi
        fi
        
        # Update status
        sed -i '' "s/STATUS=running/STATUS=stopped/" "$instance_file" 2>/dev/null || \
        sed -i "s/STATUS=running/STATUS=stopped/" "$instance_file"
        
        echo "✅ Agent $agent_id stopped"
    else
        echo "Agent $agent_id is not running"
    fi
}
```

### stop_all_instances()

Stops all running Claude instances.

```bash
stop_all_instances() {
    local force="${1:-false}"
    local instance_dir="/tmp/claude_instances"
    
    if [[ ! -f "$instance_dir/registry.txt" ]]; then
        echo "No instances to stop"
        return 0
    fi
    
    echo "Stopping all Claude instances..."
    
    while IFS=: read -r agent_id pid start_time; do
        stop_instance "$agent_id" "$force"
    done < "$instance_dir/registry.txt"
    
    echo "✅ All instances stopped"
}
```

### restart_instance()

Restarts a Claude instance with the same configuration.

```bash
restart_instance() {
    local agent_id="$1"
    local instance_file="/tmp/claude_instances/agent_${agent_id}.info"
    
    if [[ ! -f "$instance_file" ]]; then
        echo "Error: No instance configuration found for Agent $agent_id"
        return 1
    fi
    
    source "$instance_file"
    
    # Stop if running
    stop_instance "$agent_id"
    
    # Wait a moment
    sleep 2
    
    # Respawn with same configuration
    local prompt_file="/tmp/agent_${agent_id}_prompt.md"
    
    if [[ -f "$prompt_file" ]]; then
        spawn_claude_instance "$agent_id" "$prompt_file" "$WORKING_DIR"
    else
        echo "Error: Original prompt file not found"
        return 1
    fi
}
```

### get_instance_logs()

Retrieves logs from a Claude instance.

```bash
get_instance_logs() {
    local agent_id="$1"
    local lines="${2:-50}"
    local follow="${3:-false}"
    
    local log_file="/tmp/claude_agent_${agent_id}.log"
    
    if [[ ! -f "$log_file" ]]; then
        echo "No log file found for Agent $agent_id"
        return 1
    fi
    
    if [[ "$follow" == "true" ]]; then
        echo "Following logs for Agent $agent_id (Ctrl+C to stop)..."
        tail -f "$log_file"
    else
        echo "=== Last $lines lines from Agent $agent_id logs ==="
        tail -n "$lines" "$log_file"
    fi
}
```

### cleanup_instances()

Cleans up instance files and stops orphaned processes.

```bash
cleanup_instances() {
    local instance_dir="/tmp/claude_instances"
    
    echo "🧹 Cleaning up Claude instances..."
    
    # Stop any running instances
    stop_all_instances
    
    # Clean up files
    rm -rf "$instance_dir"
    rm -f /tmp/claude_agent_*.log
    rm -f /tmp/agent_*_prompt.md
    
    # Look for orphaned Claude processes
    local orphans=$(pgrep -f "claude.*Agent-" | tr '\n' ' ')
    
    if [[ -n "$orphans" ]]; then
        echo "Found orphaned processes: $orphans"
        echo "Terminating..."
        kill $orphans 2>/dev/null
    fi
    
    echo "✅ Cleanup complete"
}
```

## Helper Functions

### generate_spawn_config()

Generates spawn configuration from task requirements.

```bash
generate_spawn_config() {
    local agent_id="$1"
    local task_count="$2"
    local complexity="${3:-medium}"
    
    local config=""
    
    # Adjust resources based on task load
    case "$complexity" in
        low)
            config="--memory-limit 2G --cpu-limit 50"
            ;;
        medium)
            config="--memory-limit 4G --cpu-limit 75"
            ;;
        high)
            config="--memory-limit 8G --cpu-limit 100"
            ;;
    esac
    
    # Add task-specific options
    if [[ $task_count -gt 20 ]]; then
        config="$config --extended-context"
    fi
    
    echo "$config"
}
```

### verify_spawn_requirements()

Checks system requirements before spawning.

```bash
verify_spawn_requirements() {
    local agent_count="$1"
    
    # Check Claude CLI availability
    if ! command -v claude &> /dev/null; then
        echo "❌ Error: Claude CLI not found"
        return 1
    fi
    
    # Check system resources
    local available_memory=$(free -g 2>/dev/null | awk '/^Mem:/ {print $7}' || \
                           sysctl hw.memsize 2>/dev/null | awk '{print int($2/1073741824)}')
    
    local required_memory=$((agent_count * 2))  # 2GB per agent minimum
    
    if [[ $available_memory -lt $required_memory ]]; then
        echo "⚠️  Warning: Low memory. Available: ${available_memory}GB, Required: ${required_memory}GB"
    fi
    
    # Check for existing instances
    local existing=$(pgrep -c -f "claude.*Agent-" || echo 0)
    
    if [[ $existing -gt 0 ]]; then
        echo "ℹ️  Note: $existing Claude instance(s) already running"
    fi
    
    return 0
}
```

## Usage Examples

### Basic Spawning

```bash
# Spawn a single agent
spawn_claude_instance "1" "/tmp/agent_1_prompt.md" "/path/to/project"

# Spawn multiple agents from coordination file
spawn_multiple_agents "tasks/multi-agent-coordination.md" "1 2 3"

# Spawn all agents
spawn_multiple_agents
```

### Monitoring and Management

```bash
# Monitor all instances
monitor_instances 5  # Update every 5 seconds

# Check specific instance logs
get_instance_logs "2" 100

# Follow logs in real-time
get_instance_logs "2" 0 true

# Restart an agent
restart_instance "3"

# Stop all agents
stop_all_instances
```

### Integration Script

```bash
#!/bin/bash
source utils/claude-spawner.md

# Verify system is ready
verify_spawn_requirements 4

# Spawn agents
spawn_multiple_agents "tasks/multi-agent-coordination.md" "all"

# Monitor until complete
monitor_instances 30
```

## Best Practices

1. **Resource Management**: Monitor system resources when spawning multiple agents
2. **Error Handling**: Always check spawn success before proceeding
3. **Graceful Shutdown**: Use stop commands rather than killing processes
4. **Log Monitoring**: Regularly check logs for errors or issues
5. **Cleanup**: Run cleanup after multi-agent sessions to free resources