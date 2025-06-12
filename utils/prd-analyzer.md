# PRD Analyzer Utilities

Utilities for analyzing PRDs, comparing against completed work, and generating improvement recommendations.

## Purpose

Provides comprehensive PRD analysis functionality to support iterative development cycles, enabling accurate assessment of project completion and identification of gaps.

## Core Functions

### extract_prd_sections()

Extracts key sections from a PRD file for analysis.

```bash
extract_prd_sections() {
    local prd_file="$1"
    
    if [[ ! -f "$prd_file" ]]; then
        echo "Error: PRD file not found: $prd_file"
        return 1
    fi
    
    # Extract major sections
    local current_section=""
    declare -A sections
    
    while IFS= read -r line; do
        # Detect section headers
        if [[ "$line" =~ ^##[[:space:]](.+)$ ]]; then
            current_section="${BASH_REMATCH[1]}"
            sections["$current_section"]=""
        elif [[ -n "$current_section" ]]; then
            sections["$current_section"]+="$line"$'\n'
        fi
    done < "$prd_file"
    
    # Export sections for use
    for section in "${!sections[@]}"; do
        export "PRD_SECTION_$(echo "$section" | tr ' ' '_' | tr '[:lower:]' '[:upper:]')"="${sections[$section]}"
    done
    
    # List available sections
    echo "Extracted sections:"
    for section in "${!sections[@]}"; do
        echo "  - $section"
    done
}
```

### analyze_goals_completion()

Analyzes how well the completed work meets PRD goals.

```bash
analyze_goals_completion() {
    local prd_file="$1"
    local task_files=("${@:2}")  # All remaining arguments are task files
    
    # Extract goals from PRD
    local goals=()
    local in_goals_section=false
    
    while IFS= read -r line; do
        if [[ "$line" =~ ^##[[:space:]]Goals ]]; then
            in_goals_section=true
        elif [[ "$line" =~ ^##[[:space:]] ]] && [[ "$in_goals_section" == true ]]; then
            break
        elif [[ "$in_goals_section" == true ]] && [[ "$line" =~ ^[0-9]+\.[[:space:]](.+) ]]; then
            goals+=("${BASH_REMATCH[1]}")
        fi
    done < "$prd_file"
    
    # Analyze each goal
    echo "## Goals Completion Analysis"
    echo ""
    
    local goal_num=1
    for goal in "${goals[@]}"; do
        echo "### Goal $goal_num: $goal"
        
        # Search for related completed tasks
        local related_tasks=()
        for task_file in "${task_files[@]}"; do
            if [[ -f "$task_file" ]]; then
                # Extract keywords from goal
                local keywords=$(echo "$goal" | tr '[:upper:]' '[:lower:]' | grep -oE '[a-z]{4,}' | sort -u)
                
                # Search for completed tasks with matching keywords
                while IFS= read -r task_line; do
                    local task_lower=$(echo "$task_line" | tr '[:upper:]' '[:lower:]')
                    local match_count=0
                    
                    for keyword in $keywords; do
                        if [[ "$task_lower" =~ $keyword ]]; then
                            match_count=$((match_count + 1))
                        fi
                    done
                    
                    # If multiple keywords match, consider it related
                    if [[ $match_count -ge 2 ]]; then
                        related_tasks+=("$task_line")
                    fi
                done < <(grep "^[[:space:]]*-[[:space:]]\[x\]" "$task_file" 2>/dev/null)
            fi
        done
        
        # Assess completion
        if [[ ${#related_tasks[@]} -gt 0 ]]; then
            echo "**Status:** Partially Complete"
            echo "**Related completed tasks:**"
            for task in "${related_tasks[@]}"; do
                echo "$task"
            done
        else
            echo "**Status:** Not Started or In Progress"
            echo "**Note:** No completed tasks directly relate to this goal"
        fi
        
        echo ""
        goal_num=$((goal_num + 1))
    done
}
```

### analyze_requirements_coverage()

Checks functional requirements against implemented features.

```bash
analyze_requirements_coverage() {
    local prd_file="$1"
    shift
    local task_files=("$@")
    
    echo "## Functional Requirements Coverage"
    echo ""
    
    # Extract requirements sections
    local current_section=""
    local current_req=""
    declare -A requirements
    
    while IFS= read -r line; do
        if [[ "$line" =~ ^###[[:space:]]([0-9]+\.[[:space:]])?(.+) ]]; then
            current_section="${BASH_REMATCH[2]}"
        elif [[ "$line" =~ ^[0-9]+\.[0-9]+\.[[:space:]](.+) ]]; then
            current_req="${BASH_REMATCH[1]}"
            requirements["$current_section|$current_req"]=1
        fi
    done < <(sed -n '/## Functional Requirements/,/## Non-Goals/p' "$prd_file")
    
    # Check each requirement
    local total_reqs=${#requirements[@]}
    local completed_reqs=0
    
    for req_key in "${!requirements[@]}"; do
        IFS='|' read -r section req <<< "$req_key"
        echo "### $section"
        echo "**Requirement:** $req"
        
        # Search for implementation evidence
        local implemented=false
        for task_file in "${task_files[@]}"; do
            if [[ -f "$task_file" ]]; then
                # Look for completed tasks mentioning this requirement
                if grep -q "^[[:space:]]*-[[:space:]]\[x\].*$section" "$task_file" 2>/dev/null || \
                   grep -q "^[[:space:]]*-[[:space:]]\[x\].*$(echo "$req" | cut -d' ' -f1-3)" "$task_file" 2>/dev/null; then
                    implemented=true
                    completed_reqs=$((completed_reqs + 1))
                    break
                fi
            fi
        done
        
        if [[ "$implemented" == true ]]; then
            echo "**Status:** ✅ Implemented"
        else
            echo "**Status:** ❌ Not Implemented"
        fi
        echo ""
    done
    
    # Summary
    local coverage_percent=$((completed_reqs * 100 / total_reqs))
    echo "### Coverage Summary"
    echo "- Total Requirements: $total_reqs"
    echo "- Implemented: $completed_reqs"
    echo "- Coverage: ${coverage_percent}%"
}
```

### identify_gaps()

Identifies gaps between PRD and implementation.

```bash
identify_gaps() {
    local prd_file="$1"
    shift
    local task_files=("$@")
    
    echo "## Identified Gaps"
    echo ""
    
    # Check for incomplete high-level tasks
    echo "### Incomplete Features"
    local has_incomplete=false
    
    for task_file in "${task_files[@]}"; do
        if [[ -f "$task_file" ]]; then
            local incomplete_count=$(grep -c "^[[:space:]]*-[[:space:]]\[\]" "$task_file" 2>/dev/null || echo 0)
            if [[ $incomplete_count -gt 0 ]]; then
                has_incomplete=true
                echo ""
                echo "**$(basename "$task_file"):** $incomplete_count incomplete tasks"
                grep "^[[:space:]]*-[[:space:]]\[\]" "$task_file" | head -5 | sed 's/\[\]/❌/'
                if [[ $incomplete_count -gt 5 ]]; then
                    echo "  ... and $((incomplete_count - 5)) more"
                fi
            fi
        fi
    done
    
    if [[ "$has_incomplete" == false ]]; then
        echo "None - all planned tasks completed!"
    fi
    
    # Check for missing error handling
    echo ""
    echo "### Missing Components"
    
    # Common patterns that should be implemented
    local patterns=(
        "error handling|error recovery|exception"
        "test|testing|unit test|integration test"
        "documentation|docs|guide"
        "validation|validate|check"
        "logging|log|debug"
    )
    
    for pattern in "${patterns[@]}"; do
        local pattern_name=$(echo "$pattern" | cut -d'|' -f1)
        local found=false
        
        for task_file in "${task_files[@]}"; do
            if [[ -f "$task_file" ]]; then
                if grep -Ei "^[[:space:]]*-[[:space:]]\[x\].*($pattern)" "$task_file" >/dev/null 2>&1; then
                    found=true
                    break
                fi
            fi
        done
        
        if [[ "$found" == false ]]; then
            echo "- ⚠️  Limited $pattern_name implementation detected"
        fi
    done
}
```

### generate_recommendations()

Generates specific recommendations based on analysis.

```bash
generate_recommendations() {
    local mode="$1"
    local coverage_percent="$2"
    local incomplete_count="$3"
    
    echo "## Recommendations"
    echo ""
    
    if [[ "$mode" == "improve" ]]; then
        echo "### Priority Improvements"
        echo ""
        
        # Based on coverage
        if [[ $coverage_percent -lt 50 ]]; then
            echo "1. **Complete Core Features** - Current coverage is only ${coverage_percent}%"
            echo "   - Focus on implementing missing functional requirements"
            echo "   - Prioritize user-facing features over internal optimizations"
        elif [[ $coverage_percent -lt 80 ]]; then
            echo "1. **Fill Feature Gaps** - Coverage at ${coverage_percent}% needs improvement"
            echo "   - Complete remaining functional requirements"
            echo "   - Add missing error handling and validation"
        else
            echo "1. **Polish and Optimize** - Good coverage at ${coverage_percent}%"
            echo "   - Focus on performance improvements"
            echo "   - Enhance user experience"
        fi
        
        # Based on incomplete tasks
        if [[ $incomplete_count -gt 10 ]]; then
            echo "2. **Complete Pending Work** - $incomplete_count tasks remain"
            echo "   - Assign additional resources to incomplete tasks"
            echo "   - Consider splitting large tasks into smaller ones"
        fi
        
        # Standard improvements
        echo "3. **Quality Enhancements**"
        echo "   - Add comprehensive error handling"
        echo "   - Improve test coverage"
        echo "   - Update documentation"
        
    else  # next-phase mode
        echo "### Next Phase Features"
        echo ""
        
        if [[ $coverage_percent -lt 80 ]]; then
            echo "⚠️  **Warning:** Consider completing current phase first (${coverage_percent}% coverage)"
            echo ""
        fi
        
        echo "1. **Advanced Capabilities**"
        echo "   - Multi-user collaboration features"
        echo "   - Workflow templates and customization"
        echo "   - External service integrations"
        echo ""
        
        echo "2. **Enterprise Features**"
        echo "   - Role-based access control"
        echo "   - Audit logging and compliance"
        echo "   - Performance monitoring"
        echo ""
        
        echo "3. **Developer Experience**"
        echo "   - IDE plugins and extensions"
        echo "   - API for custom integrations"
        echo "   - Advanced analytics dashboard"
    fi
}
```

### create_iteration_summary()

Creates a summary of the iteration analysis.

```bash
create_iteration_summary() {
    local prd_file="$1"
    local analysis_file="$2"
    local mode="$3"
    
    cat > "$analysis_file" << EOF
# Iteration Analysis Summary

**Date:** $(date)  
**PRD:** $prd_file  
**Mode:** $mode  

## Executive Summary

This analysis evaluates the current implementation against the Product Requirements Document (PRD) to identify gaps, assess completion status, and recommend next steps.

EOF
    
    # Add sections based on analysis results
    # This would be populated by the other analysis functions
}
```

## Usage Examples

### Basic PRD Analysis

```bash
# Source the utilities
source utils/prd-analyzer.md

# Extract PRD sections
extract_prd_sections "tasks/project-prd.md"

# Access extracted sections
echo "$PRD_SECTION_GOALS"
```

### Goals Completion Analysis

```bash
# Analyze goals across all agent task files
analyze_goals_completion "tasks/project-prd.md" tasks/agent-*-tasks.md
```

### Requirements Coverage

```bash
# Check requirements implementation
analyze_requirements_coverage "tasks/project-prd.md" tasks/agent-*-tasks.md
```

### Gap Analysis

```bash
# Identify what's missing
identify_gaps "tasks/project-prd.md" tasks/agent-*-tasks.md
```

### Generate Recommendations

```bash
# Get improvement recommendations
generate_recommendations "improve" 75 5

# Get next phase recommendations
generate_recommendations "next-phase" 90 0
```

## Integration Notes

- Works with standard PRD format used by `/user:generate-prd`
- Analyzes task files from all agents
- Provides data for `/user:iterate-project` command
- Generates actionable recommendations
- Supports both improvement and next-phase modes