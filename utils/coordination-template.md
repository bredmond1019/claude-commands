# Coordination File Template

This is a simplified template for the multi-agent coordination file that agents can use directly.

## Basic Template

```markdown
# Multi-Agent Coordination: [Project Name]

Generated: [YYYY-MM-DD HH:MM:SS]
Total Agents: [N]
Total Tasks: [N]
Status: active

## Agent Overview

### Agent Count: [N]
**Rationale:** [Why this number of agents was chosen]

### Agent Roles
1. **[Agent 1 Name]:** [Description]
2. **[Agent 2 Name]:** [Description]
3. **[Agent 3 Name]:** [Description]
4. **[Agent 4 Name]:** [Description]

## Task Distribution Summary

### Original Task List Breakdown
- **Agent 1:** Tasks [list]
- **Agent 2:** Tasks [list]
- **Agent 3:** Tasks [list]
- **Agent 4:** Tasks [list]

## Critical Dependencies

### Sequential Dependencies (must happen in order)
1. **[Name]:** Agent [X] task [Y] → Agent [A] task [B]
2. **[Name]:** Agent [X] task [Y] → Agent [A] task [B]

### Parallel Opportunities
- **Phase 1:** Agent 1 works alone on foundation
- **Phase 2:** Agents 2 and 3 can work simultaneously
- **Phase 3:** All agents collaborate on integration

## Execution Status

| Agent | Status | Tasks Complete | Tasks Remaining | Last Update |
|-------|--------|----------------|-----------------|-------------|
| 1     | active | 0/5            | 5               | [timestamp] |
| 2     | idle   | 0/7            | 7               | [timestamp] |
| 3     | idle   | 0/8            | 8               | [timestamp] |
| 4     | idle   | 0/6            | 6               | [timestamp] |

## Integration Milestones

1. **Foundation Complete:** Agent 1 - All templates and utilities ready
   - Status: pending
   - Blocking: Agents 2, 3

2. **Workflow Commands Ready:** Agent 2 - PRD and task generation operational
   - Status: pending
   - Blocking: Agent 3 (multi-agent features)

3. **Execution Framework Ready:** Agent 3 - Task execution operational
   - Status: pending
   - Blocking: Agent 4 (testing)

4. **First Integration Test:** All Agents - End-to-end workflow passes
   - Status: pending
   - Blocking: None

5. **Documentation Complete:** Agent 4 - All guides and examples ready
   - Status: pending
   - Blocking: None

## Communication Protocol

### Handoff Notifications
- Agents update this section when completing dependencies
- Format: **[Timestamp]:** Agent [X] → Agent [Y]: [Message]

### Blocking Issues
- Report any blockers here for team resolution
- Format: **[Timestamp]:** Agent [X]: [Issue description]

## Shared Resources

- **Argument Parsing Template:** `utils/argument-parser.md` (Agent 1 → All)
- **Command Templates:** `utils/command-templates.md` (Agent 1 → All)
- **Agent Prompts:** `utils/agent-prompts.md` (Agent 1 → Agent 3)
- **Coordination Spec:** `docs/coordination-spec.md` (Agent 1 → Agent 3)

## Risk Mitigation

- Each agent monitors context usage (spawn sub-agents at 80%)
- Regular commits after each subtask
- Agents with blocked tasks pivot to documentation/testing
- Integration Agent provides continuous feedback
```

## Quick Reference Functions

```bash
# Quick status update
quick_status_update() {
    local agent_id="$1"
    local complete="$2"
    local remaining="$3"
    local status="${4:-active}"
    
    echo "| $agent_id | $status | $complete/$((complete + remaining)) | $remaining | $(date '+%Y-%m-%d %H:%M:%S') |"
}

# Add communication entry
add_communication() {
    local from="$1"
    local to="$2"
    local message="$3"
    
    echo "- **$(date '+%Y-%m-%d %H:%M:%S'):** Agent $from → $to: $message"
}

# Check if milestone complete
check_milestone() {
    local milestone_name="$1"
    local status="$2"  # pending|in-progress|completed
    
    echo "   - Status: $status"
}
```

## Usage Examples

### Creating Initial Coordination File
```bash
# Generate coordination file for 4 agents with 26 total tasks
cat > tasks/multi-agent-coordination.md <<'EOF'
# Multi-Agent Coordination: Claude Code Workflow Automation System

Generated: 2024-01-15 10:00:00
Total Agents: 4
Total Tasks: 26
Status: active

## Agent Overview

### Agent Count: 4
**Rationale:** Four agents provide optimal parallelization while maintaining clear ownership boundaries.

### Agent Roles
1. **Command Architecture Agent:** Core infrastructure and templates
2. **Workflow Commands Agent:** PRD and task generation commands
3. **Execution & Coordination Agent:** Task execution and multi-agent logic
4. **Integration & Documentation Agent:** Testing and documentation

[Rest of template...]
EOF
```

### Updating Agent Status
```bash
# Agent 1 completes a task
sed -i '' 's/| 1     | active | 0\/5 | 5 |/| 1     | active | 1\/5 | 4 |/' tasks/multi-agent-coordination.md
```

### Adding Communication
```bash
# Agent 1 notifies others about completed dependency
echo "- **$(date '+%Y-%m-%d %H:%M:%S'):** Agent 1 → All: Argument parsing template complete (task 1.2)" >> tasks/multi-agent-coordination.md
```