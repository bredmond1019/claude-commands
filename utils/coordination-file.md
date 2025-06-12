# Coordination File Management Utilities

Utilities for managing multi-agent coordination through shared files.

## Core Functions

### update_coordination_status()

Updates agent progress in the coordination file.

```bash
update_coordination_status() {
    local coordination_file="$1"
    local agent_id="$2"
    local completed_tasks="$3"
    local total_tasks="$4"
    local status="${5:-active}"
    
    # Calculate percentage
    local percentage=0
    if [[ $total_tasks -gt 0 ]]; then
        percentage=$(( (completed_tasks * 100) / total_tasks ))
    fi
    
    # Create status line
    local status_line="- **Agent $agent_id**: $completed_tasks/$total_tasks tasks (${percentage}%)"
    
    # Add status emoji
    case "$status" in
        complete)
            status_line="$status_line ✅ Complete"
            ;;
        active)
            status_line="$status_line 🚧 Active"
            ;;
        waiting)
            status_line="$status_line ⏸️ Waiting"
            ;;
        blocked)
            status_line="$status_line 🚫 Blocked"
            ;;
    esac
    
    # Update or add status in coordination file
    if grep -q "Agent $agent_id.*tasks" "$coordination_file"; then
        # Update existing line
        sed -i '' "s/- \*\*Agent $agent_id\*\*:.*/$status_line/" "$coordination_file" 2>/dev/null || \
        sed -i "s/- \*\*Agent $agent_id\*\*:.*/$status_line/" "$coordination_file"
    else
        # Add new status section if needed
        if ! grep -q "## Agent Status" "$coordination_file"; then
            echo -e "\n## Agent Status (Last Updated: $(date))\n\n### Progress" >> "$coordination_file"
        fi
        echo "$status_line" >> "$coordination_file"
    fi
    
    # Update timestamp
    update_coordination_timestamp "$coordination_file"
}
```

### record_handoff()

Records completion of a dependency handoff.

```bash
record_handoff() {
    local coordination_file="$1"
    local from_agent="$2"
    local to_agent="$3"
    local deliverable="$4"
    local timestamp="${5:-$(date +%H:%M)}"
    
    # Find or create handoff section
    if ! grep -q "### Recent Handoffs" "$coordination_file"; then
        echo -e "\n### Recent Handoffs" >> "$coordination_file"
    fi
    
    # Add handoff record
    local handoff_line="- ✅ Agent $from_agent → Agent $to_agent: $deliverable ($timestamp)"
    
    # Add after the Recent Handoffs header
    sed -i '' "/### Recent Handoffs/a\\
$handoff_line" "$coordination_file" 2>/dev/null || \
    awk '/### Recent Handoffs/ {print; print "'"$handoff_line"'"; next} {print}' "$coordination_file" > "${coordination_file}.tmp" && \
    mv "${coordination_file}.tmp" "$coordination_file"
}
```

### check_dependencies_met()

Checks if all dependencies for an agent are satisfied.

```bash
check_dependencies_met() {
    local coordination_file="$1"
    local agent_id="$2"
    
    # Extract dependencies section for this agent
    local deps_section=$(awk "/Agent $agent_id.*Prerequisites/,/^$/" "$coordination_file" | \
                        grep "From.*:" | \
                        sed 's/.*From \(Agent [0-9]\+\).*: \(.*\)/\1:\2/')
    
    if [[ -z "$deps_section" ]]; then
        echo "No dependencies found for Agent $agent_id"
        return 0
    fi
    
    local all_met=true
    local unmet_deps=""
    
    while IFS=: read -r from_agent dependency; do
        # Check if this dependency is marked in handoffs
        if ! grep -q "$from_agent.*$dependency.*✅" "$coordination_file"; then
            all_met=false
            unmet_deps="${unmet_deps}\n  - Waiting on $from_agent: $dependency"
        fi
    done <<< "$deps_section"
    
    if [[ "$all_met" == "true" ]]; then
        echo "✅ All dependencies satisfied for Agent $agent_id"
        return 0
    else
        echo -e "⏸️ Agent $agent_id has unmet dependencies:$unmet_deps"
        return 1
    fi
}
```

### create_blocker_entry()

Creates an entry in the blockers file for cross-agent issues.

```bash
create_blocker_entry() {
    local blocker_file="${1:-tasks/blockers.md}"
    local agent_id="$2"
    local issue="$3"
    local severity="${4:-medium}"  # low, medium, high, critical
    local blocking_agents="${5:-}"  # comma-separated list
    
    # Create blockers file if it doesn't exist
    if [[ ! -f "$blocker_file" ]]; then
        cat > "$blocker_file" << 'EOF'
# Active Blockers

This file tracks blocking issues that need resolution across agents.

## Format
- **ID**: Unique identifier (BLOCK-XXX)
- **Reporter**: Agent that discovered the issue
- **Severity**: low | medium | high | critical
- **Blocking**: Agents affected by this issue
- **Status**: active | investigating | resolved

---

EOF
    fi
    
    # Generate blocker ID
    local last_id=$(grep -o "BLOCK-[0-9]\+" "$blocker_file" | tail -1 | cut -d- -f2)
    local new_id=$((${last_id:-0} + 1))
    local blocker_id="BLOCK-$(printf "%03d" $new_id)"
    
    # Add blocker entry
    cat >> "$blocker_file" << EOF

## $blocker_id: $issue

- **Reporter**: Agent $agent_id
- **Severity**: $severity
- **Blocking**: ${blocking_agents:-None specified}
- **Status**: active
- **Created**: $(date)

### Description
$issue

### Impact
Agents affected: ${blocking_agents:-To be determined}

### Notes
_Add investigation notes here_

---
EOF
    
    echo "Created blocker $blocker_id"
    
    # Update coordination file with blocker reference
    if [[ -n "$blocking_agents" ]]; then
        update_coordination_status "$1" "$agent_id" "-" "-" "blocked"
    fi
}
```

### sync_coordination_state()

Synchronizes coordination state across all agent contexts.

```bash
sync_coordination_state() {
    local coordination_file="${1:-tasks/multi-agent-coordination.md}"
    local sync_dir="${2:-tasks/agent-contexts}"
    
    mkdir -p "$sync_dir"
    
    echo "🔄 Synchronizing coordination state..."
    
    # Get current state from all agents
    local agent_count=$(grep -c "^[0-9]\+\. \*\*.*Agent:" "$coordination_file" || echo 0)
    
    for agent_id in $(seq 1 "$agent_count"); do
        local task_file="tasks/agent-${agent_id}-tasks.md"
        
        if [[ -f "$task_file" ]]; then
            # Get task stats
            local completed=$(grep -c "\[x\]" "$task_file" || echo 0)
            local total=$(grep -c "\[.\]" "$task_file" || echo 0)
            
            # Determine status
            local status="active"
            if [[ $completed -eq $total ]] && [[ $total -gt 0 ]]; then
                status="complete"
            elif ! check_dependencies_met "$coordination_file" "$agent_id" > /dev/null; then
                status="waiting"
            fi
            
            # Update coordination file
            update_coordination_status "$coordination_file" "$agent_id" "$completed" "$total" "$status"
            
            # Save agent state snapshot
            save_agent_state "$agent_id" "$sync_dir"
        fi
    done
    
    echo "✅ Coordination state synchronized"
}
```

### save_agent_state()

Saves current agent state for recovery and handoff.

```bash
save_agent_state() {
    local agent_id="$1"
    local state_dir="$2"
    local state_file="$state_dir/agent-${agent_id}-state.md"
    
    cat > "$state_file" << EOF
# Agent $agent_id State Snapshot

Generated: $(date)

## Task Progress
$(grep "\[.\]" "tasks/agent-${agent_id}-tasks.md" | head -20)

## Recent Commits
$(git log --oneline -5 --grep="Agent $agent_id" 2>/dev/null || echo "No agent-specific commits found")

## Current Context
- Working Directory: $(pwd)
- Active Branch: $(git branch --show-current)
- Modified Files: $(git status --porcelain | wc -l) files

## Last Activity
$(tail -5 "/tmp/claude_agent_${agent_id}.log" 2>/dev/null || echo "No activity log found")
EOF
}
```

### monitor_coordination_conflicts()

Monitors for and resolves coordination conflicts.

```bash
monitor_coordination_conflicts() {
    local coordination_file="$1"
    local interval="${2:-30}"
    
    echo "👀 Monitoring for coordination conflicts..."
    
    local last_hash=""
    
    while true; do
        # Check for file modifications
        local current_hash=$(md5sum "$coordination_file" 2>/dev/null | cut -d' ' -f1 || \
                           md5 -q "$coordination_file" 2>/dev/null)
        
        if [[ -n "$last_hash" ]] && [[ "$current_hash" != "$last_hash" ]]; then
            # File changed, check for conflicts
            if grep -q "<<<<<<< HEAD" "$coordination_file"; then
                echo "⚠️  Merge conflict detected in coordination file!"
                resolve_coordination_conflict "$coordination_file"
            fi
            
            # Check for concurrent updates
            local update_count=$(grep -c "Last Updated:" "$coordination_file")
            if [[ $update_count -gt 1 ]]; then
                echo "⚠️  Multiple update timestamps detected"
                consolidate_updates "$coordination_file"
            fi
        fi
        
        last_hash="$current_hash"
        sleep "$interval"
    done
}
```

### update_coordination_timestamp()

Updates the last modified timestamp in coordination file.

```bash
update_coordination_timestamp() {
    local coordination_file="$1"
    local timestamp="$(date)"
    
    # Update existing timestamp or add new one
    if grep -q "Last Updated:" "$coordination_file"; then
        sed -i '' "s/Last Updated:.*/Last Updated: $timestamp)/" "$coordination_file" 2>/dev/null || \
        sed -i "s/Last Updated:.*/Last Updated: $timestamp)/" "$coordination_file"
    else
        # Add timestamp after status header
        sed -i '' "/## Agent Status/s/$/\\n(Last Updated: $timestamp)/" "$coordination_file" 2>/dev/null || \
        sed -i "/## Agent Status/s/$/\n(Last Updated: $timestamp)/" "$coordination_file"
    fi
}
```

### generate_coordination_report()

Generates a summary report from coordination data.

```bash
generate_coordination_report() {
    local coordination_file="${1:-tasks/multi-agent-coordination.md}"
    local output_file="${2:-tasks/coordination-report.md}"
    
    echo "📊 Generating coordination report..."
    
    # Gather statistics
    local total_agents=$(grep -c "^[0-9]\+\. \*\*.*Agent:" "$coordination_file" || echo 0)
    local active_agents=$(grep -c "🚧 Active" "$coordination_file" || echo 0)
    local completed_agents=$(grep -c "✅ Complete" "$coordination_file" || echo 0)
    local blocked_agents=$(grep -c "🚫 Blocked" "$coordination_file" || echo 0)
    
    # Calculate overall progress
    local total_tasks=0
    local completed_tasks=0
    
    for agent_id in $(seq 1 "$total_agents"); do
        if [[ -f "tasks/agent-${agent_id}-tasks.md" ]]; then
            local agent_total=$(grep -c "\[.\]" "tasks/agent-${agent_id}-tasks.md" || echo 0)
            local agent_completed=$(grep -c "\[x\]" "tasks/agent-${agent_id}-tasks.md" || echo 0)
            total_tasks=$((total_tasks + agent_total))
            completed_tasks=$((completed_tasks + agent_completed))
        fi
    done
    
    local overall_percentage=0
    if [[ $total_tasks -gt 0 ]]; then
        overall_percentage=$(( (completed_tasks * 100) / total_tasks ))
    fi
    
    # Generate report
    cat > "$output_file" << EOF
# Multi-Agent Coordination Report

Generated: $(date)

## Executive Summary

- **Total Agents**: $total_agents
- **Active**: $active_agents
- **Completed**: $completed_agents
- **Blocked**: $blocked_agents

## Overall Progress

**$completed_tasks / $total_tasks tasks completed (${overall_percentage}%)**

$(printf '%.0s█' $(seq 1 $((overall_percentage / 5))))$(printf '%.0s░' $(seq 1 $(((100 - overall_percentage) / 5))))

## Agent Details

$(grep "^- \*\*Agent" "$coordination_file" || echo "No agent status found")

## Recent Activity

### Last 5 Handoffs
$(grep "✅ Agent.*→" "$coordination_file" | tail -5 || echo "No handoffs recorded")

### Active Blockers
$(if [[ -f "tasks/blockers.md" ]]; then
    grep -A2 "Status.*active" "tasks/blockers.md" | grep -B2 "BLOCK-" || echo "No active blockers"
else
    echo "No blocker tracking file found"
fi)

## Recommendations

$(generate_coordination_recommendations "$overall_percentage" "$blocked_agents")

EOF
    
    echo "✅ Report saved to: $output_file"
}
```

### generate_coordination_recommendations()

Generates recommendations based on coordination state.

```bash
generate_coordination_recommendations() {
    local progress="$1"
    local blocked_count="$2"
    
    local recommendations=""
    
    if [[ $blocked_count -gt 0 ]]; then
        recommendations="${recommendations}- **Priority**: Resolve $blocked_count blocking issues\n"
    fi
    
    if [[ $progress -lt 25 ]]; then
        recommendations="${recommendations}- Consider increasing parallelization\n"
        recommendations="${recommendations}- Review task distribution for balance\n"
    elif [[ $progress -lt 50 ]]; then
        recommendations="${recommendations}- Monitor for upcoming dependencies\n"
        recommendations="${recommendations}- Prepare for integration phase\n"
    elif [[ $progress -lt 75 ]]; then
        recommendations="${recommendations}- Begin integration testing\n"
        recommendations="${recommendations}- Review completed work for consistency\n"
    else
        recommendations="${recommendations}- Focus on final integration\n"
        recommendations="${recommendations}- Prepare deployment checklist\n"
    fi
    
    echo -e "$recommendations"
}
```

## Usage Examples

### Basic Coordination Updates

```bash
# Update agent progress
update_coordination_status "tasks/multi-agent-coordination.md" "2" 5 10 "active"

# Record a handoff
record_handoff "tasks/multi-agent-coordination.md" "1" "3" "API templates complete"

# Check dependencies
check_dependencies_met "tasks/multi-agent-coordination.md" "3"
```

### Blocker Management

```bash
# Create a blocker
create_blocker_entry "tasks/blockers.md" "2" "Database schema conflicts with existing tables" "high" "2,3,4"

# Monitor for conflicts
monitor_coordination_conflicts "tasks/multi-agent-coordination.md" &
```

### Synchronization and Reporting

```bash
# Sync all agent states
sync_coordination_state

# Generate progress report
generate_coordination_report

# Save agent state for handoff
save_agent_state "2" "tasks/agent-contexts"
```

## Best Practices

1. **Atomic Updates**: Use file locking when multiple agents update simultaneously
2. **Regular Syncs**: Sync coordination state at least every 5 minutes
3. **Clear Communication**: Use descriptive handoff messages
4. **Blocker Priority**: Address blockers immediately to prevent cascading delays
5. **State Preservation**: Save agent state before context switches