# iterate-project

Deploy multiple agents to work in parallel to generate a detailed task list. Analyze completed work against the PRD and generate next iteration of improvements or features.

## Arguments

- `--prompt <text>` (optional): Additional prompt to add to the analysis.
- `--prd-file <path>` (optional): Path to the PRD file. Defaults to `tasks/project-prd.md`
- `--output-prd <path>` (optional): Path for new PRD. Defaults to `tasks/project-prd-v{N}.md`
- `--mode <mode>` (optional): `improve` (refine existing features) or `next-phase` (add new features). Defaults to `improve`
- `--feedback-file <path>` (optional): Path to the feedback file.

## Usage

```bash
/user:prd:iterate-project --prompt "Add a new feature to the project" --prd-file tasks/custom-prd.md --output-prd tasks/prd-v2.md --mode next-phase --feedback-file feedback.txt
```

<prompt-default>
I think this product is a great idea and I really want it to be successful. So let's review the project with the lens of really making sure everything has sound logic, no compile issues, no stubbed code or functions that still need logic, etc. If there is, let's make note.  Also, make note of areas we could use improvement to meet the high standards we set our goals out for!
</prompt-default>

## Workflow

Deploy multiple agents to work in parallel to generate a detailed task list.

**Parallel Execution Management:**
Deploy multiple Sub Agents to generate agents in parallel for maximum efficiency and creative diversity:

- Launch 3-5 Sub Agents simultaneously using Task tool
- Monitor agent progress and completion
- Collect and validate all completed agents

1. **Analyze Completed Work**: Review all completed tasks across all agents
2. **Compare Against PRD**: Identify gaps, improvements, or next phase opportunities
3. **Gather Feedback**: Incorporate user feedback if provided
4. **Generate Analysis Report**: Document findings and recommendations
5. **Create New PRD**: Generate updated PRD for next iteration
6. **Update Documentation**: Reflect changes in project documentation
7. **Generate New Tasks**: Optionally create task list for next iteration

## Implementation

```bash
#!/bin/bash

# Parse arguments
PRD_FILE="tasks/project-prd.md"
FEEDBACK_FILE=""
OUTPUT_PRD=""
MODE="improve"
PROMPT="I think this product is a great idea and I really want it to be successful. So let's review the project with the lens of really making sure everything has sound logic, no compile issues, no stubbed code or functions that still need logic, etc. If there is, let's make note.  Also, make note of areas we could use improvement to meet the high standards we set our goals out for!"

# Parse named arguments
IFS=' ' read -ra ARGS <<< "$ARGUMENTS"
for arg in "${ARGS[@]}"; do
    if [[ "$arg" =~ ^--prd-file=(.+)$ ]]; then
        PRD_FILE="${BASH_REMATCH[1]}"
    elif [[ "$arg" =~ ^--feedback-file=(.+)$ ]]; then
        FEEDBACK_FILE="${BASH_REMATCH[1]}"
    elif [[ "$arg" =~ ^--output-prd=(.+)$ ]]; then
        OUTPUT_PRD="${BASH_REMATCH[1]}"
    elif [[ "$arg" =~ ^--mode=(.+)$ ]]; then
        MODE="${BASH_REMATCH[1]}"
    elif [[ "$arg" =~ ^--prompt=(.+)$ ]]; then
        PROMPT="${BASH_REMATCH[1]}"
    fi
done

# Determine output PRD filename if not specified
if [[ -z "$OUTPUT_PRD" ]]; then
    # Find the next version number
    VERSION=2
    while [[ -f "tasks/project-prd-v${VERSION}.md" ]]; do
        VERSION=$((VERSION + 1))
    done
    OUTPUT_PRD="tasks/project-prd-v${VERSION}.md"
fi

# Validate PRD exists
if [[ ! -f "$PRD_FILE" ]]; then
    echo "Error: PRD file not found: $PRD_FILE"
    exit 1
fi

# Create analysis report
ANALYSIS_FILE="tasks/iteration-analysis-$(date +%Y%m%d_%H%M%S).md"

echo "=== Project Iteration Analysis ==="
echo "Current PRD: $PRD_FILE"
echo "Mode: $MODE"
echo "Output PRD: $OUTPUT_PRD"
if [[ -n "$PROMPT" ]]; then
    echo "Additional Prompt: $PROMPT"
fi
echo ""

# Analyze completed work
echo "Analyzing completed tasks..."

cat > "$ANALYSIS_FILE" << 'EOF'
# Project Iteration Analysis

## Analysis Date
$(date)

## Current PRD
$PRD_FILE

## Completed Work Summary

### By Agent
EOF

# Analyze each agent's completed tasks
for agent_file in tasks/agent-*-tasks.md; do
    if [[ -f "$agent_file" ]]; then
        agent_name=$(basename "$agent_file" .md)
        total_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[.\]" "$agent_file" 2>/dev/null || echo 0)
        completed_tasks=$(grep -c "^[[:space:]]*-[[:space:]]\[x\]" "$agent_file" 2>/dev/null || echo 0)

        echo "" >> "$ANALYSIS_FILE"
        echo "#### $agent_name" >> "$ANALYSIS_FILE"
        echo "- Completion: $completed_tasks/$total_tasks tasks ($(( completed_tasks * 100 / (total_tasks + 1) ))%)" >> "$ANALYSIS_FILE"
        echo "- Completed items:" >> "$ANALYSIS_FILE"
        grep "^[[:space:]]*-[[:space:]]\[x\]" "$agent_file" | sed 's/\[x\]//' | sed 's/^/  /' >> "$ANALYSIS_FILE" 2>/dev/null || echo "  None" >> "$ANALYSIS_FILE"
    fi
done

# Analyze against PRD goals
echo "" >> "$ANALYSIS_FILE"
echo "## PRD Goals Analysis" >> "$ANALYSIS_FILE"

# Extract goals from PRD
if grep -q "## Goals" "$PRD_FILE"; then
    echo "" >> "$ANALYSIS_FILE"
    echo "### Original Goals" >> "$ANALYSIS_FILE"
    sed -n '/## Goals/,/^##[^#]/p' "$PRD_FILE" | grep -E "^[0-9]+\." | sed 's/^/- /' >> "$ANALYSIS_FILE"
fi

# Analyze functional requirements completion
echo "" >> "$ANALYSIS_FILE"
echo "## Functional Requirements Status" >> "$ANALYSIS_FILE"

# Extract and analyze each requirement section
grep "^### " "$PRD_FILE" | while read -r requirement; do
    echo "" >> "$ANALYSIS_FILE"
    echo "$requirement" >> "$ANALYSIS_FILE"
    # This is where you'd add specific requirement checking logic
    echo "- Status: [Requires manual review]" >> "$ANALYSIS_FILE"
done

# Include user feedback if provided
if [[ -n "$FEEDBACK_FILE" ]] && [[ -f "$FEEDBACK_FILE" ]]; then
    echo "" >> "$ANALYSIS_FILE"
    echo "## User Feedback" >> "$ANALYSIS_FILE"
    cat "$FEEDBACK_FILE" >> "$ANALYSIS_FILE"
fi

if [[ -n "$PROMPT" ]]; then
    echo "" >> "$ANALYSIS_FILE"
    echo "## Additional Analysis Prompt" >> "$ANALYSIS_FILE"
    echo "$PROMPT" >> "$ANALYSIS_FILE"
fi

# Generate recommendations
echo "" >> "$ANALYSIS_FILE"
echo "## Recommendations" >> "$ANALYSIS_FILE"

if [[ "$MODE" == "improve" ]]; then
    cat >> "$ANALYSIS_FILE" << 'EOF'

### Improvement Opportunities
1. **Code Quality**
   - Add comprehensive error handling where missing
   - Improve test coverage for edge cases
   - Refactor complex functions for better maintainability

2. **Performance**
   - Optimize file operations for large repositories
   - Add caching for frequently accessed data
   - Implement parallel processing where applicable

3. **User Experience**
   - Add progress indicators for long-running operations
   - Improve error messages with actionable suggestions
   - Create interactive mode for complex commands

4. **Documentation**
   - Add more examples for common use cases
   - Create video tutorials for visual learners
   - Improve troubleshooting guide with real issues
EOF
else
    cat >> "$ANALYSIS_FILE" << 'EOF'

### Next Phase Features
1. **Advanced Workflows**
   - Template system for common project types
   - Integration with CI/CD pipelines
   - Custom workflow definitions

2. **Collaboration Features**
   - Multi-user task assignment
   - Real-time progress tracking
   - Agent performance analytics

3. **AI Enhancements**
   - Smart task prioritization
   - Automatic dependency detection
   - Context-aware suggestions

4. **Integration Expansion**
   - GitHub/GitLab integration
   - Slack/Discord notifications
   - IDE plugins for popular editors
EOF
fi

# Get unfinished business
echo "" >> "$ANALYSIS_FILE"
echo "## Incomplete Items" >> "$ANALYSIS_FILE"

for agent_file in tasks/agent-*-tasks.md; do
    if [[ -f "$agent_file" ]]; then
        pending=$(grep -c "^[[:space:]]*-[[:space:]]\[\]" "$agent_file" 2>/dev/null || echo 0)
        if [[ $pending -gt 0 ]]; then
            echo "" >> "$ANALYSIS_FILE"
            echo "### $(basename "$agent_file" .md)" >> "$ANALYSIS_FILE"
            grep "^[[:space:]]*-[[:space:]]\[\]" "$agent_file" | sed 's/\[\]//' | sed 's/^/- /' >> "$ANALYSIS_FILE"
        fi
    fi
done

# Display analysis summary
echo "Analysis complete. Report saved to: $ANALYSIS_FILE"
echo ""
echo "=== Key Findings ==="

# Calculate overall completion
total_all=0
completed_all=0
for agent_file in tasks/agent-*-tasks.md; do
    if [[ -f "$agent_file" ]]; then
        total=$(grep -c "^[[:space:]]*-[[:space:]]\[.\]" "$agent_file" 2>/dev/null || echo 0)
        completed=$(grep -c "^[[:space:]]*-[[:space:]]\[x\]" "$agent_file" 2>/dev/null || echo 0)
        total_all=$((total_all + total))
        completed_all=$((completed_all + completed))
    fi
done

echo "Overall Completion: $completed_all/$total_all tasks ($(( completed_all * 100 / (total_all + 1) ))%)"
echo ""

# Generate new PRD
echo "Generating new PRD..."
echo ""

# Create new PRD based on analysis
cat > "$OUTPUT_PRD" << EOF
# Product Requirements Document: Claude Code Workflow Automation System
# Version: $(basename "$OUTPUT_PRD" .md | sed 's/.*-v//')
# Generated: $(date)

## Iteration Summary

This PRD represents the $(echo "$MODE" | sed 's/improve/improvement iteration/;s/next-phase/next phase/') of the Claude Code Workflow Automation System.

### Changes from Previous Version
- Generated from: $PRD_FILE
- Analysis report: $ANALYSIS_FILE
- Mode: $MODE

## Introduction/Overview

$(sed -n '/## Introduction\/Overview/,/^##[^#]/p' "$PRD_FILE" | tail -n +2 | head -n -1)

$(if [[ "$MODE" == "next-phase" ]]; then
    echo ""
    echo "### Phase $(basename "$OUTPUT_PRD" .md | sed 's/.*-v//') Additions"
    echo ""
    echo "This phase extends the system with advanced features for enterprise use cases, enhanced collaboration, and deeper AI integration."
fi)

## Goals

$(sed -n '/## Goals/,/^##[^#]/p' "$PRD_FILE" | tail -n +2 | head -n -1)

$(if [[ "$MODE" == "improve" ]]; then
    echo ""
    echo "### Improvement Goals for This Iteration"
    echo ""
    echo "6. Enhance error handling and recovery mechanisms across all commands"
    echo "7. Improve performance for large-scale projects with many tasks"
    echo "8. Strengthen test coverage to ensure reliability"
else
    echo ""
    echo "### Additional Goals for Phase $(basename "$OUTPUT_PRD" .md | sed 's/.*-v//')"
    echo ""
    echo "6. Enable multi-user collaboration on shared projects"
    echo "7. Provide advanced analytics and insights on development velocity"
    echo "8. Support custom workflow templates for different project types"
    echo "9. Integrate with external tools and services"
fi)

## User Stories

$(sed -n '/## User Stories/,/^##[^#]/p' "$PRD_FILE" | tail -n +2 | head -n -1)

$(if [[ "$MODE" == "next-phase" ]]; then
    echo ""
    echo "### New User Stories for Phase $(basename "$OUTPUT_PRD" .md | sed 's/.*-v//')"
    echo ""
    echo "7. **As a team lead**, I want to assign tasks to specific team members and track their progress, so that I can manage team workload effectively."
    echo ""
    echo "8. **As a developer**, I want to use predefined workflow templates, so that I can quickly set up common project types without manual configuration."
    echo ""
    echo "9. **As a project manager**, I want to receive notifications about project progress, so that I can stay informed without constant checking."
fi)

## Functional Requirements

$(sed -n '/## Functional Requirements/,/^## Non-Goals/p' "$PRD_FILE" | tail -n +2 | head -n -1)

$(if [[ "$MODE" == "improve" ]]; then
    echo ""
    echo "### 7. Improvement Requirements"
    echo ""
    echo "7.1. **Enhanced Error Handling**"
    echo "- The system must provide detailed error messages with suggested fixes"
    echo "- The system must implement retry logic for transient failures"
    echo "- The system must create error logs for debugging"
    echo ""
    echo "7.2. **Performance Optimization**"
    echo "- The system must handle task lists with 1000+ items efficiently"
    echo "- The system must implement caching for repeated operations"
    echo "- The system must optimize git operations for large repositories"
else
    echo ""
    echo "### 7. Multi-User Collaboration"
    echo ""
    echo "7.1. The system must support task assignment to specific users"
    echo "7.2. The system must track task ownership and completion by user"
    echo "7.3. The system must prevent conflicts when multiple users work simultaneously"
    echo "7.4. The system must provide task handoff capabilities between users"
    echo ""
    echo "### 8. Workflow Templates"
    echo ""
    echo "8.1. The system must provide built-in templates for common project types"
    echo "8.2. The system must allow custom template creation and sharing"
    echo "8.3. The system must support template parameters for customization"
    echo "8.4. The system must validate template compatibility before use"
fi)

## Non-Goals (Out of Scope)

$(sed -n '/## Non-Goals/,/^##[^#]/p' "$PRD_FILE" | tail -n +2 | head -n -1)

## Design Considerations

$(sed -n '/## Design Considerations/,/^##[^#]/p' "$PRD_FILE" | tail -n +2 | head -n -1)

## Technical Considerations

$(sed -n '/## Technical Considerations/,/^##[^#]/p' "$PRD_FILE" | tail -n +2 | head -n -1)

$(if [[ "$MODE" == "next-phase" ]]; then
    echo ""
    echo "### Additional Technical Considerations"
    echo ""
    echo "7. **Multi-User Architecture**: Implement file locking and conflict resolution for concurrent access"
    echo "8. **Template Engine**: Use a flexible template system that supports conditionals and loops"
    echo "9. **Notification System**: Integrate with webhooks for external service notifications"
    echo "10. **Analytics Storage**: Implement efficient data storage for usage metrics and insights"
fi)

## Success Metrics

$(sed -n '/## Success Metrics/,/^##[^#]/p' "$PRD_FILE" | tail -n +2 | head -n -1)

$(if [[ "$MODE" == "improve" ]]; then
    echo ""
    echo "### Improvement Metrics"
    echo ""
    echo "6. **Error Reduction**: 50% decrease in user-reported issues"
    echo "7. **Performance**: 2x faster execution for large task lists"
    echo "8. **Test Coverage**: Achieve 90% code coverage across all commands"
else
    echo ""
    echo "### Phase $(basename "$OUTPUT_PRD" .md | sed 's/.*-v//') Metrics"
    echo ""
    echo "6. **Collaboration**: Support 5+ concurrent users without conflicts"
    echo "7. **Template Usage**: 80% of new projects use templates"
    echo "8. **Integration Adoption**: 60% of users enable external integrations"
fi)

## Open Questions

$(if [[ "$MODE" == "improve" ]]; then
    echo "1. What specific error scenarios are most common and need priority handling?"
    echo "2. Should we implement automatic error recovery or always require user intervention?"
    echo "3. What performance benchmarks should we target for enterprise use?"
    echo "4. How detailed should error logs be without compromising security?"
else
    echo "1. Which collaboration platforms should we prioritize for integration?"
    echo "2. How should we handle conflicts when multiple agents modify the same files?"
    echo "3. What template categories would provide the most value?"
    echo "4. Should templates be versioned and updatable?"
    echo "5. What analytics would provide the most actionable insights?"
fi)

## Implementation Notes

- Previous PRD: $PRD_FILE
- Analysis Report: $ANALYSIS_FILE
- Iteration Mode: $MODE
$(if [[ -n "$FEEDBACK_FILE" ]]; then
    echo "- User Feedback: $FEEDBACK_FILE"
fi)
EOF

echo "New PRD generated: $OUTPUT_PRD"
echo ""

# Update project documentation
DOCS_DIR="docs"
if [[ -d "$DOCS_DIR" ]]; then
    # Update version history
    VERSION_HISTORY="$DOCS_DIR/version-history.md"
    if [[ ! -f "$VERSION_HISTORY" ]]; then
        cat > "$VERSION_HISTORY" << EOF
# Version History

## Overview
This document tracks the evolution of the Claude Code Workflow Automation System through its various iterations.

## Versions
EOF
    fi

    # Append new version entry
    cat >> "$VERSION_HISTORY" << EOF

### $(basename "$OUTPUT_PRD" .md | sed 's/project-prd-//')
- Date: $(date +%Y-%m-%d)
- Mode: $MODE
- PRD: $OUTPUT_PRD
- Analysis: $ANALYSIS_FILE
- Key Changes:
$(if [[ "$MODE" == "improve" ]]; then
    echo "  - Enhanced error handling and recovery"
    echo "  - Performance optimizations"
    echo "  - Improved test coverage"
else
    echo "  - Added multi-user collaboration"
    echo "  - Introduced workflow templates"
    echo "  - Implemented external integrations"
fi)
EOF

    echo "Updated version history: $VERSION_HISTORY"
fi

# Prompt for next steps
echo ""
echo "=== Next Steps ==="
echo "1. Review the analysis report: $ANALYSIS_FILE"
echo "2. Review the new PRD: $OUTPUT_PRD"
echo "3. Generate new task list: /user:generate-tasks prd=$OUTPUT_PRD"
echo "4. Distribute tasks to agents: /user:generate-multi-agent-tasks"
echo "5. Execute the next iteration: /user:execute-multi-agent-tasks"
```

## Analysis Report Format

The iteration analysis report includes:

1. **Completed Work Summary** - What each agent accomplished
2. **PRD Goals Analysis** - How well the original goals were met
3. **Functional Requirements Status** - Which requirements are complete
4. **User Feedback** - Incorporated feedback if provided
5. **Recommendations** - Specific improvements or new features
6. **Incomplete Items** - Tasks that weren't finished

## PRD Evolution

The new PRD maintains the structure of the original while:

- **Improvement Mode**: Adds refinements to existing features
- **Next Phase Mode**: Introduces entirely new feature sets
- Preserving successful elements from the previous version
- Incorporating lessons learned from implementation
- Addressing feedback and identified gaps

## Error Handling

- Validates PRD file exists before analysis
- Handles missing feedback files gracefully
- Creates version numbers automatically to avoid conflicts
- Provides clear error messages for missing dependencies

## Integration Points

- Reads completed task status from all agent task files
- Can incorporate external feedback files
- Updates project documentation automatically
- Prepares for next task generation cycle
- Works with all other workflow commands

## Agent Implementation

Deploy multiple agents to work in parallel to generate a detailed task list.

**Parallel Execution Management:**
Deploy multiple Sub Agents to generate agents in parallel for maximum efficiency and creative diversity:

- Launch 3-5 Sub Agents simultaneously using Task tool
- Monitor agent progress and completion
- Collect and validate all completed agents
