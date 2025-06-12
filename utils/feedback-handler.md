# Feedback Handler Utilities

Utilities for collecting, parsing, and incorporating user feedback into project iterations.

## Purpose

Provides standardized feedback handling to ensure user input is properly captured, analyzed, and integrated into project improvements.

## Feedback Format

Standard feedback file structure:

```markdown
# User Feedback

## Date: YYYY-MM-DD

## Feedback Type: [bug|feature|improvement|general]

## Priority: [high|medium|low]

## Summary
Brief description of the feedback

## Details
Detailed explanation of the issue, request, or suggestion

## Expected Behavior
What the user expected to happen (for bugs)

## Actual Behavior
What actually happened (for bugs)

## Suggested Solution
User's proposed solution or approach

## Impact
How this affects the user's workflow
```

## Core Functions

### parse_feedback_file()

Parses a feedback file and extracts structured information.

```bash
parse_feedback_file() {
    local feedback_file="$1"
    
    if [[ ! -f "$feedback_file" ]]; then
        echo "Error: Feedback file not found: $feedback_file"
        return 1
    fi
    
    # Initialize variables
    local date=""
    local type=""
    local priority=""
    local summary=""
    local details=""
    local expected=""
    local actual=""
    local solution=""
    local impact=""
    local current_section=""
    
    # Parse the file
    while IFS= read -r line; do
        if [[ "$line" =~ ^##[[:space:]]Date:[[:space:]](.+) ]]; then
            date="${BASH_REMATCH[1]}"
        elif [[ "$line" =~ ^##[[:space:]]Feedback[[:space:]]Type:[[:space:]](.+) ]]; then
            type="${BASH_REMATCH[1]}"
        elif [[ "$line" =~ ^##[[:space:]]Priority:[[:space:]](.+) ]]; then
            priority="${BASH_REMATCH[1]}"
        elif [[ "$line" =~ ^##[[:space:]](.+)$ ]]; then
            current_section="${BASH_REMATCH[1]}"
        elif [[ -n "$current_section" ]] && [[ -n "$line" ]]; then
            case "$current_section" in
                "Summary")
                    summary+="$line "
                    ;;
                "Details")
                    details+="$line "
                    ;;
                "Expected Behavior")
                    expected+="$line "
                    ;;
                "Actual Behavior")
                    actual+="$line "
                    ;;
                "Suggested Solution")
                    solution+="$line "
                    ;;
                "Impact")
                    impact+="$line "
                    ;;
            esac
        fi
    done < "$feedback_file"
    
    # Export parsed data
    export FEEDBACK_DATE="$date"
    export FEEDBACK_TYPE="$type"
    export FEEDBACK_PRIORITY="$priority"
    export FEEDBACK_SUMMARY="$(echo "$summary" | xargs)"
    export FEEDBACK_DETAILS="$(echo "$details" | xargs)"
    export FEEDBACK_EXPECTED="$(echo "$expected" | xargs)"
    export FEEDBACK_ACTUAL="$(echo "$actual" | xargs)"
    export FEEDBACK_SOLUTION="$(echo "$solution" | xargs)"
    export FEEDBACK_IMPACT="$(echo "$impact" | xargs)"
    
    # Display parsed feedback
    echo "Parsed Feedback:"
    echo "  Date: $date"
    echo "  Type: $type"
    echo "  Priority: $priority"
    echo "  Summary: $(echo "$summary" | xargs)"
}
```

### categorize_feedback()

Categorizes feedback into actionable buckets.

```bash
categorize_feedback() {
    local feedback_dir="${1:-feedback}"
    
    # Initialize categories
    declare -A bugs=()
    declare -A features=()
    declare -A improvements=()
    declare -A general=()
    
    # Process all feedback files
    for feedback_file in "$feedback_dir"/*.md; do
        if [[ -f "$feedback_file" ]]; then
            parse_feedback_file "$feedback_file" >/dev/null
            
            case "$FEEDBACK_TYPE" in
                "bug")
                    bugs["$feedback_file"]="$FEEDBACK_PRIORITY:$FEEDBACK_SUMMARY"
                    ;;
                "feature")
                    features["$feedback_file"]="$FEEDBACK_PRIORITY:$FEEDBACK_SUMMARY"
                    ;;
                "improvement")
                    improvements["$feedback_file"]="$FEEDBACK_PRIORITY:$FEEDBACK_SUMMARY"
                    ;;
                *)
                    general["$feedback_file"]="$FEEDBACK_PRIORITY:$FEEDBACK_SUMMARY"
                    ;;
            esac
        fi
    done
    
    # Display categorized feedback
    echo "## Feedback Categories"
    echo ""
    
    if [[ ${#bugs[@]} -gt 0 ]]; then
        echo "### Bugs (${#bugs[@]})"
        for file in "${!bugs[@]}"; do
            IFS=':' read -r priority summary <<< "${bugs[$file]}"
            echo "- [$priority] $summary"
            echo "  File: $file"
        done
        echo ""
    fi
    
    if [[ ${#features[@]} -gt 0 ]]; then
        echo "### Feature Requests (${#features[@]})"
        for file in "${!features[@]}"; do
            IFS=':' read -r priority summary <<< "${features[$file]}"
            echo "- [$priority] $summary"
            echo "  File: $file"
        done
        echo ""
    fi
    
    if [[ ${#improvements[@]} -gt 0 ]]; then
        echo "### Improvements (${#improvements[@]})"
        for file in "${!improvements[@]}"; do
            IFS=':' read -r priority summary <<< "${improvements[$file]}"
            echo "- [$priority] $summary"
            echo "  File: $file"
        done
        echo ""
    fi
    
    if [[ ${#general[@]} -gt 0 ]]; then
        echo "### General Feedback (${#general[@]})"
        for file in "${!general[@]}"; do
            IFS=':' read -r priority summary <<< "${general[$file]}"
            echo "- [$priority] $summary"
            echo "  File: $file"
        done
    fi
}
```

### generate_feedback_tasks()

Converts feedback into actionable tasks.

```bash
generate_feedback_tasks() {
    local feedback_file="$1"
    local task_prefix="${2:-Fix}"
    
    # Parse the feedback
    parse_feedback_file "$feedback_file" >/dev/null
    
    # Generate task based on type
    local task=""
    case "$FEEDBACK_TYPE" in
        "bug")
            task="$task_prefix bug: $FEEDBACK_SUMMARY"
            if [[ -n "$FEEDBACK_EXPECTED" ]]; then
                task+="\n  - Expected: $FEEDBACK_EXPECTED"
            fi
            if [[ -n "$FEEDBACK_ACTUAL" ]]; then
                task+="\n  - Actual: $FEEDBACK_ACTUAL"
            fi
            ;;
        "feature")
            task="Implement feature: $FEEDBACK_SUMMARY"
            if [[ -n "$FEEDBACK_DETAILS" ]]; then
                task+="\n  - Details: $FEEDBACK_DETAILS"
            fi
            ;;
        "improvement")
            task="Improve: $FEEDBACK_SUMMARY"
            if [[ -n "$FEEDBACK_SOLUTION" ]]; then
                task+="\n  - Suggested approach: $FEEDBACK_SOLUTION"
            fi
            ;;
        *)
            task="Address feedback: $FEEDBACK_SUMMARY"
            ;;
    esac
    
    # Add priority indicator
    case "$FEEDBACK_PRIORITY" in
        "high")
            task="[HIGH PRIORITY] $task"
            ;;
        "low")
            task="[Low Priority] $task"
            ;;
    esac
    
    echo -e "- [ ] $task"
}
```

### create_feedback_summary()

Creates a summary report of all feedback.

```bash
create_feedback_summary() {
    local feedback_dir="${1:-feedback}"
    local output_file="${2:-feedback/summary.md}"
    
    cat > "$output_file" << EOF
# Feedback Summary Report

**Generated:** $(date)  
**Total Feedback Items:** $(find "$feedback_dir" -name "*.md" -not -name "summary.md" | wc -l)

## Overview

This report summarizes all user feedback received for the Claude Code Workflow Automation System.

EOF
    
    # Add categorized feedback
    categorize_feedback "$feedback_dir" >> "$output_file"
    
    # Add priority breakdown
    echo "" >> "$output_file"
    echo "## Priority Breakdown" >> "$output_file"
    echo "" >> "$output_file"
    
    local high_count=0
    local medium_count=0
    local low_count=0
    
    for feedback_file in "$feedback_dir"/*.md; do
        if [[ -f "$feedback_file" ]] && [[ "$feedback_file" != *"summary.md" ]]; then
            parse_feedback_file "$feedback_file" >/dev/null
            case "$FEEDBACK_PRIORITY" in
                "high") high_count=$((high_count + 1)) ;;
                "medium") medium_count=$((medium_count + 1)) ;;
                "low") low_count=$((low_count + 1)) ;;
            esac
        fi
    done
    
    echo "- High Priority: $high_count items" >> "$output_file"
    echo "- Medium Priority: $medium_count items" >> "$output_file"
    echo "- Low Priority: $low_count items" >> "$output_file"
    
    # Add recommended actions
    echo "" >> "$output_file"
    echo "## Recommended Actions" >> "$output_file"
    echo "" >> "$output_file"
    
    if [[ $high_count -gt 0 ]]; then
        echo "### Immediate Actions (High Priority)" >> "$output_file"
        echo "" >> "$output_file"
        
        for feedback_file in "$feedback_dir"/*.md; do
            if [[ -f "$feedback_file" ]] && [[ "$feedback_file" != *"summary.md" ]]; then
                parse_feedback_file "$feedback_file" >/dev/null
                if [[ "$FEEDBACK_PRIORITY" == "high" ]]; then
                    generate_feedback_tasks "$feedback_file" >> "$output_file"
                fi
            fi
        done
    fi
    
    echo "" >> "$output_file"
    echo "### Generated Tasks" >> "$output_file"
    echo "" >> "$output_file"
    echo "To convert all feedback into tasks, run:" >> "$output_file"
    echo '```bash' >> "$output_file"
    echo 'for file in feedback/*.md; do' >> "$output_file"
    echo '    [[ "$file" != *"summary.md" ]] && generate_feedback_tasks "$file"' >> "$output_file"
    echo 'done > tasks/feedback-tasks.md' >> "$output_file"
    echo '```' >> "$output_file"
    
    echo "Summary created: $output_file"
}
```

### incorporate_feedback_into_prd()

Adds feedback considerations to a new PRD iteration.

```bash
incorporate_feedback_into_prd() {
    local feedback_dir="${1:-feedback}"
    local prd_section=""
    
    # Collect high-priority bugs
    local bugs=""
    for feedback_file in "$feedback_dir"/*.md; do
        if [[ -f "$feedback_file" ]] && [[ "$feedback_file" != *"summary.md" ]]; then
            parse_feedback_file "$feedback_file" >/dev/null
            if [[ "$FEEDBACK_TYPE" == "bug" ]] && [[ "$FEEDBACK_PRIORITY" == "high" ]]; then
                bugs+="- $FEEDBACK_SUMMARY\n"
            fi
        fi
    done
    
    # Collect feature requests
    local features=""
    for feedback_file in "$feedback_dir"/*.md; do
        if [[ -f "$feedback_file" ]] && [[ "$feedback_file" != *"summary.md" ]]; then
            parse_feedback_file "$feedback_file" >/dev/null
            if [[ "$FEEDBACK_TYPE" == "feature" ]]; then
                features+="- $FEEDBACK_SUMMARY\n"
            fi
        fi
    done
    
    # Generate PRD section
    prd_section="## User Feedback Integration\n\n"
    
    if [[ -n "$bugs" ]]; then
        prd_section+="### Critical Bugs to Address\n\n"
        prd_section+="$bugs\n"
    fi
    
    if [[ -n "$features" ]]; then
        prd_section+="### Requested Features\n\n"
        prd_section+="$features\n"
    fi
    
    prd_section+="### Feedback Summary\n\n"
    prd_section+="- Total feedback items: $(find "$feedback_dir" -name "*.md" -not -name "summary.md" | wc -l)\n"
    prd_section+="- High priority items require immediate attention\n"
    prd_section+="- Feature requests align with next phase goals\n"
    
    echo -e "$prd_section"
}
```

### create_feedback_template()

Creates a template for new feedback.

```bash
create_feedback_template() {
    local output_file="${1:-feedback/template.md}"
    
    cat > "$output_file" << 'EOF'
# User Feedback

## Date: $(date +%Y-%m-%d)

## Feedback Type: [bug|feature|improvement|general]

## Priority: [high|medium|low]

## Summary
<!-- Brief description of the feedback (one line) -->

## Details
<!-- Detailed explanation of the issue, request, or suggestion -->

## Expected Behavior
<!-- What the user expected to happen (for bugs) -->

## Actual Behavior
<!-- What actually happened (for bugs) -->

## Suggested Solution
<!-- User's proposed solution or approach -->

## Impact
<!-- How this affects the user's workflow -->

## Additional Information
<!-- Any other relevant details, error messages, or context -->

---
*Please save this file as `feedback/[date]-[type]-[brief-description].md`*
EOF
    
    echo "Feedback template created: $output_file"
}
```

## Usage Examples

### Create Feedback Template

```bash
# Source utilities
source utils/feedback-handler.md

# Create a new feedback template
create_feedback_template "feedback/new-issue.md"
```

### Process Single Feedback

```bash
# Parse and display feedback
parse_feedback_file "feedback/2024-01-15-bug-context-overflow.md"

# Generate task from feedback
generate_feedback_tasks "feedback/2024-01-15-bug-context-overflow.md"
```

### Analyze All Feedback

```bash
# Categorize all feedback
categorize_feedback "feedback"

# Create summary report
create_feedback_summary "feedback" "feedback/summary-$(date +%Y%m%d).md"
```

### Incorporate into PRD

```bash
# Generate PRD section with feedback
incorporate_feedback_into_prd "feedback" > prd-feedback-section.md
```

## Integration Notes

- Feedback files should follow the standard format
- Works with `/user:iterate-project` command
- Can generate tasks for any workflow command
- Supports priority-based processing
- Maintains traceability from feedback to implementation