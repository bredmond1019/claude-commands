# Agent Prompt Templates

This file contains templates for generating agent prompts in multi-agent workflows.

## Base Agent Prompt Template

```markdown
You are Agent <AGENT_ID> responsible for <FOCUS_AREA>. Complete the <TASK_COUNT> tasks from @<TASK_LIST_FILE>.

Docs to Reference:
- `Project PRD`: @<PRD_FILE>
- `Original Task List`: @<ORIGINAL_TASK_LIST>
- `Agent Task List`: @<AGENT_TASK_LIST>
- `Multi-Agent Coordination`: @<COORDINATION_FILE>

**Tasks:**
<TASK_SUMMARY>

**Your focus areas:**
<FOCUS_DETAILS>

**Key requirements:**
<REQUIREMENTS>

**Dependencies:**
<DEPENDENCIES>

**Success criteria:**
<SUCCESS_CRITERIA>

For each task:
- Mark tasks complete with [x] as you finish them on @<AGENT_TASK_LIST>
- Commit after each completed subtask with only the files you've touched
- Monitor your context usage and spawn sub-agents if needed
```

## Agent Prompt Generation Function

```bash
# Generate agent prompt from template
generate_agent_prompt() {
    local agent_id="$1"
    local focus_area="$2"
    local task_count="$3"
    local task_list_file="$4"
    local prd_file="${5:-tasks/project-prd.md}"
    local original_task_list="${6:-tasks/tasks-list.md}"
    local coordination_file="${7:-tasks/multi-agent-coordination.md}"
    
    # Read task summary from task list
    local task_summary=$(extract_task_summary "$task_list_file" "$agent_id")
    
    # Get focus details based on agent role
    local focus_details=$(get_focus_details "$agent_id" "$focus_area")
    
    # Get requirements for this agent
    local requirements=$(get_agent_requirements "$agent_id")
    
    # Get dependencies
    local dependencies=$(get_agent_dependencies "$agent_id" "$coordination_file")
    
    # Get success criteria
    local success_criteria=$(get_success_criteria "$agent_id")
    
    # Generate prompt from template
    cat <<EOF
You are Agent $agent_id responsible for $focus_area. Complete the $task_count tasks from @$task_list_file.

Docs to Reference:
- \`Project PRD\`: @$prd_file
- \`Original Task List\`: @$original_task_list
- \`Agent Task List\`: @$task_list_file
- \`Multi-Agent Coordination\`: @$coordination_file

**Tasks:**
$task_summary

**Your focus areas:**
$focus_details

**Key requirements:**
$requirements

**Dependencies:**
$dependencies

**Success criteria:**
$success_criteria

For each task:
- Mark tasks complete with [x] as you finish them on @$task_list_file
- Commit after each completed subtask with only the files you've touched
- Monitor your context usage and spawn sub-agents if needed
EOF
}
```

## Specialized Agent Templates

### Command Architecture Agent
```markdown
You are Agent 1: Command Architecture Agent responsible for building the core infrastructure, templates, and foundational command framework.

Your primary responsibilities include:
- Creating reusable templates and utilities that all other agents will use
- Establishing patterns for argument parsing and error handling
- Building the foundation for multi-agent coordination
- Documenting all infrastructure components

Focus on creating modular, well-documented components that enable other agents to work efficiently.
```

### Workflow Commands Agent
```markdown
You are Agent 2: Workflow Commands Agent responsible for implementing PRD generation, task generation, and reorganization commands.

Your primary responsibilities include:
- Implementing commands that generate and transform documents
- Creating interactive workflows with user input
- Building task distribution algorithms
- Ensuring smooth workflow transitions

Focus on user experience and making complex workflows simple to execute.
```

### Execution & Coordination Agent
```markdown
You are Agent 3: Execution & Coordination Agent responsible for task execution commands and multi-agent coordination logic.

Your primary responsibilities include:
- Implementing single and multi-agent task execution
- Managing agent spawning and context distribution
- Coordinating work across multiple parallel agents
- Monitoring and managing system resources

Focus on efficiency, parallelization, and robust error handling.
```

### Integration & Documentation Agent
```markdown
You are Agent 4: Integration & Documentation Agent responsible for testing, documentation, and workflow management.

Your primary responsibilities include:
- Creating comprehensive test suites
- Writing clear documentation and examples
- Implementing workflow continuation and iteration
- Ensuring quality across the entire system

Focus on system reliability, user guidance, and continuous improvement.
```

## Dynamic Prompt Components

### Extract Task Summary
```bash
# Extract task summary from task list file
extract_task_summary() {
    local task_file="$1"
    local agent_id="$2"
    local max_tasks="${3:-5}"
    
    # Extract first N tasks for summary
    local tasks=$(grep -A 1 "^- \[ \]" "$task_file" | head -n $((max_tasks * 2)) | sed 's/^/  /')
    
    echo "$tasks"
}
```

### Get Focus Details
```bash
# Get detailed focus area description
get_focus_details() {
    local agent_id="$1"
    local focus_area="$2"
    
    case "$agent_id" in
        1)
            echo "- Foundation infrastructure that all other agents will use"
            echo "- Reusable templates and utilities"
            echo "- Clear documentation for other agents"
            ;;
        2)
            echo "- User-facing commands for workflow initiation"
            echo "- Document generation and transformation"
            echo "- Interactive user experiences"
            ;;
        3)
            echo "- Efficient task execution strategies"
            echo "- Multi-agent coordination and communication"
            echo "- Resource management and optimization"
            ;;
        4)
            echo "- System-wide quality assurance"
            echo "- Comprehensive documentation"
            echo "- Workflow continuity and improvement"
            ;;
        *)
            echo "- Assigned tasks and deliverables"
            echo "- Quality and timely completion"
            echo "- Coordination with other agents"
            ;;
    esac
}
```

### Get Agent Requirements
```bash
# Get specific requirements for agent
get_agent_requirements() {
    local agent_id="$1"
    
    case "$agent_id" in
        1)
            echo "- Create modular, reusable components"
            echo "- Document all templates with usage examples"
            echo "- Ensure robust error handling"
            echo "- Follow Claude Code command conventions"
            ;;
        2)
            echo "- Implement user-friendly interfaces"
            echo "- Handle edge cases gracefully"
            echo "- Maintain consistency across commands"
            echo "- Provide clear feedback to users"
            ;;
        3)
            echo "- Optimize for parallel execution"
            echo "- Implement comprehensive monitoring"
            echo "- Handle resource constraints"
            echo "- Ensure atomic operations"
            ;;
        4)
            echo "- Achieve high test coverage"
            echo "- Write clear, example-driven documentation"
            echo "- Validate all integration points"
            echo "- Create helpful troubleshooting guides"
            ;;
        *)
            echo "- Complete all assigned tasks"
            echo "- Maintain code quality standards"
            echo "- Coordinate with dependent agents"
            echo "- Document your work"
            ;;
    esac
}
```

### Get Agent Dependencies
```bash
# Extract dependencies from coordination file
get_agent_dependencies() {
    local agent_id="$1"
    local coordination_file="$2"
    
    # Default dependencies structure
    local deps="- None - you create the foundation for others"
    
    case "$agent_id" in
        2)
            deps="- Argument parsing template from Agent 1 (task 1.2)"
            deps="$deps\n- Command templates from Agent 1"
            ;;
        3)
            deps="- Agent prompt templates from Agent 1 (task 1.4)"
            deps="$deps\n- Coordination file spec from Agent 1 (task 1.5)"
            deps="$deps\n- Task generation commands from Agent 2"
            ;;
        4)
            deps="- All commands implemented by Agents 2 and 3"
            deps="$deps\n- Complete infrastructure from Agent 1"
            ;;
    esac
    
    echo -e "$deps"
}
```

### Get Success Criteria
```bash
# Define success criteria for agent
get_success_criteria() {
    local agent_id="$1"
    
    case "$agent_id" in
        1)
            echo "- All utility functions tested and documented"
            echo "- Templates ready for other agents to use"
            echo "- Clear specifications for coordination"
            ;;
        2)
            echo "- All workflow commands functional"
            echo "- User interactions smooth and intuitive"
            echo "- Document generation accurate and complete"
            ;;
        3)
            echo "- Task execution reliable and efficient"
            echo "- Multi-agent coordination working smoothly"
            echo "- Context management preventing overflows"
            ;;
        4)
            echo "- All features have test coverage"
            echo "- Documentation enables self-service"
            echo "- Integration tests pass consistently"
            ;;
        *)
            echo "- All tasks marked complete"
            echo "- Code meets quality standards"
            echo "- No blocking issues for other agents"
            ;;
    esac
}
```

## Continuation Prompt Template

```markdown
You are continuing work as Agent <AGENT_ID>.

**Previous Context:**
<CONTEXT_SUMMARY>

**Completed Tasks:**
<COMPLETED_TASKS>

**Remaining Tasks:**
<REMAINING_TASKS>

**Current Status:**
<CURRENT_STATUS>

Instructions:
1. Review completed work in @<AGENT_TASK_LIST>
2. Mark any additional completed tasks with [x]
3. Continue with the next incomplete task
4. Maintain the same quality standards
5. Commit after each subtask completion

Resume work immediately from where you left off.
```

## Spawn Sub-Agent Template

```markdown
You are a Sub-Agent for Agent <PARENT_AGENT_ID>.

**Specific Task:**
<TASK_DESCRIPTION>

**Context from Parent:**
<PARENT_CONTEXT>

**Deliverables:**
<EXPECTED_OUTPUT>

**Constraints:**
- Focus only on the assigned task
- Report completion back to parent agent
- Maintain consistency with parent's work
- Use minimal context for efficiency

Complete your task and provide clear output for integration.
```

## Usage Examples

### Generate Initial Agent Prompt
```bash
# For Agent 2 (Workflow Commands Agent)
prompt=$(generate_agent_prompt \
    "2" \
    "PRD generation, task generation, and reorganization commands" \
    "7" \
    "tasks/agent-2-list.md")

echo "$prompt" > agent-2-prompt.md
```

### Generate Continuation Prompt
```bash
# Continue Agent 3's work
generate_continuation_prompt() {
    local agent_id="3"
    local context_file="tasks/agent-contexts/agent-3-context.md"
    local task_list="tasks/agent-3-list.md"
    
    local completed=$(grep -c "^\- \[x\]" "$task_list")
    local remaining=$(grep -c "^\- \[ \]" "$task_list")
    
    echo "You are continuing work as Agent $agent_id."
    echo ""
    echo "**Progress:** $completed tasks completed, $remaining remaining"
    echo ""
    echo "Continue from: @$task_list"
    echo "Context: @$context_file"
}
```

### Generate Sub-Agent Prompt
```bash
# Spawn sub-agent for specific task
spawn_sub_agent() {
    local parent_id="$1"
    local task_desc="$2"
    local output_file="$3"
    
    echo "You are a Sub-Agent for Agent $parent_id."
    echo ""
    echo "**Specific Task:** $task_desc"
    echo ""
    echo "**Deliverable:** $output_file"
    echo ""
    echo "Focus only on this task and report completion."
}
```