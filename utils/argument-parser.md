# Argument Parser Template

This template provides a reusable pattern for parsing arguments from the `$ARGUMENTS` variable in Claude Code commands.

## Basic Structure

```bash
# Parse arguments from "$ARGUMENTS"
# Format: arg1="value1" arg2="value2" arg3="value3"

# Initialize variables
ARG1=""
ARG2=""
ARG3=""

# Parse the arguments
if [[ -n "$ARGUMENTS" ]]; then
    # Extract arg1
    if [[ "$ARGUMENTS" =~ arg1=\"([^\"]+)\" ]] || [[ "$ARGUMENTS" =~ arg1=([^[:space:]]+) ]]; then
        ARG1="${BASH_REMATCH[1]}"
    fi
    
    # Extract arg2
    if [[ "$ARGUMENTS" =~ arg2=\"([^\"]+)\" ]] || [[ "$ARGUMENTS" =~ arg2=([^[:space:]]+) ]]; then
        ARG2="${BASH_REMATCH[1]}"
    fi
    
    # Extract arg3
    if [[ "$ARGUMENTS" =~ arg3=\"([^\"]+)\" ]] || [[ "$ARGUMENTS" =~ arg3=([^[:space:]]+) ]]; then
        ARG3="${BASH_REMATCH[1]}"
    fi
fi
```

## Advanced Pattern with Validation

```bash
# Parse and validate arguments
parse_arguments() {
    local args="$1"
    
    # Required arguments
    local REQUIRED_ARGS=("design_doc" "output_file")
    
    # Optional arguments with defaults
    local ref_files=""
    local interactive="true"
    
    # Parse design_doc (required)
    if [[ "$args" =~ design_doc=\"([^\"]+)\" ]] || [[ "$args" =~ design_doc=([^[:space:]]+) ]]; then
        design_doc="${BASH_REMATCH[1]}"
    else
        echo "ERROR: Required argument 'design_doc' not found"
        echo "Usage: /user:generate-prd design_doc=\"path/to/doc.md\" [output_file=\"path/to/output.md\"] [ref_files=\"file1.md,file2.md\"] [interactive=\"true|false\"]"
        exit 1
    fi
    
    # Parse output_file (optional with default)
    if [[ "$args" =~ output_file=\"([^\"]+)\" ]] || [[ "$args" =~ output_file=([^[:space:]]+) ]]; then
        output_file="${BASH_REMATCH[1]}"
    else
        output_file="tasks/project-prd.md"
    fi
    
    # Parse ref_files (optional, comma-separated)
    if [[ "$args" =~ ref_files=\"([^\"]+)\" ]] || [[ "$args" =~ ref_files=([^[:space:]]+) ]]; then
        ref_files="${BASH_REMATCH[1]}"
    fi
    
    # Parse interactive (optional boolean)
    if [[ "$args" =~ interactive=\"([^\"]+)\" ]] || [[ "$args" =~ interactive=([^[:space:]]+) ]]; then
        interactive="${BASH_REMATCH[1]}"
    fi
    
    # Validate boolean values
    if [[ "$interactive" != "true" && "$interactive" != "false" ]]; then
        echo "ERROR: 'interactive' must be 'true' or 'false'"
        exit 1
    fi
}

# Call the parser
parse_arguments "$ARGUMENTS"
```

## Multi-Value Arguments Pattern

For handling comma-separated lists or multiple file paths:

```bash
# Parse comma-separated file list
parse_file_list() {
    local file_list="$1"
    local -a files=()
    
    # Split by comma
    IFS=',' read -ra ADDR <<< "$file_list"
    for file in "${ADDR[@]}"; do
        # Trim whitespace
        file=$(echo "$file" | xargs)
        if [[ -n "$file" ]]; then
            files+=("$file")
        fi
    done
    
    # Return array
    echo "${files[@]}"
}

# Usage example
if [[ "$ARGUMENTS" =~ ref_files=\"([^\"]+)\" ]]; then
    ref_files="${BASH_REMATCH[1]}"
    file_array=($(parse_file_list "$ref_files"))
    
    echo "Found ${#file_array[@]} reference files:"
    for file in "${file_array[@]}"; do
        echo "  - $file"
    done
fi
```

## Positional Arguments Pattern

For commands that accept positional arguments:

```bash
# Parse positional arguments (space-separated)
parse_positional() {
    local args="$1"
    local -a positional=()
    
    # Handle quoted strings properly
    while [[ $args =~ ^[[:space:]]*\"([^\"]+)\"(.*)$ ]] || [[ $args =~ ^[[:space:]]*([^[:space:]]+)(.*)$ ]]; do
        positional+=("${BASH_REMATCH[1]}")
        args="${BASH_REMATCH[2]}"
    done
    
    # Assign to variables based on position
    if [[ ${#positional[@]} -ge 1 ]]; then
        TASK_FILE="${positional[0]}"
    fi
    
    if [[ ${#positional[@]} -ge 2 ]]; then
        MAX_TASKS="${positional[1]}"
    fi
    
    if [[ ${#positional[@]} -ge 3 ]]; then
        AGENT_ID="${positional[2]}"
    fi
}

# Usage
parse_positional "$ARGUMENTS"
```

## Complex Arguments with Nested Values

For handling more complex argument structures:

```bash
# Parse complex arguments with nested values
parse_complex_args() {
    local args="$1"
    
    # Parse agent configuration
    if [[ "$args" =~ agents=\[([^\]]+)\] ]]; then
        local agents_config="${BASH_REMATCH[1]}"
        
        # Parse individual agent specs
        IFS=',' read -ra AGENTS <<< "$agents_config"
        for i in "${!AGENTS[@]}"; do
            agent="${AGENTS[$i]}"
            # Extract agent properties
            if [[ "$agent" =~ id:([0-9]+) ]]; then
                agent_ids[$i]="${BASH_REMATCH[1]}"
            fi
            if [[ "$agent" =~ tasks:([0-9]+) ]]; then
                agent_tasks[$i]="${BASH_REMATCH[1]}"
            fi
        done
    fi
}
```

## Error Handling Template

```bash
# Comprehensive error handling for argument parsing
validate_arguments() {
    local errors=()
    
    # Check required arguments
    if [[ -z "$design_doc" ]]; then
        errors+=("Missing required argument: design_doc")
    elif [[ ! -f "$design_doc" ]]; then
        errors+=("File not found: $design_doc")
    fi
    
    # Check optional file arguments
    if [[ -n "$ref_files" ]]; then
        IFS=',' read -ra FILES <<< "$ref_files"
        for file in "${FILES[@]}"; do
            file=$(echo "$file" | xargs)
            if [[ ! -f "$file" ]]; then
                errors+=("Reference file not found: $file")
            fi
        done
    fi
    
    # Check numeric arguments
    if [[ -n "$count" ]] && ! [[ "$count" =~ ^[0-9]+$ ]] && [[ "$count" != "infinite" ]]; then
        errors+=("Invalid count value: $count (must be a number or 'infinite')")
    fi
    
    # Display errors and exit if any
    if [[ ${#errors[@]} -gt 0 ]]; then
        echo "ERROR: Argument validation failed:"
        for error in "${errors[@]}"; do
            echo "  - $error"
        done
        echo ""
        echo "Usage: $USAGE_STRING"
        exit 1
    fi
}
```

## Usage in Command Files

When using this template in a Claude Code command file:

```markdown
**Variables:**
design_doc: $ARGUMENTS
output_file: $ARGUMENTS
ref_files: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `design_doc` - Path to the design document (required)
2. `output_file` - Path for output PRD (optional, default: tasks/project-prd.md)
3. `ref_files` - Comma-separated list of reference files (optional)

[Include the parsing code here]

**EXECUTION:**
With the parsed arguments, proceed to:
1. Validate all file paths exist
2. Process the design document
3. Generate output to specified location
```

## Best Practices

1. **Always validate required arguments** - Exit gracefully with usage information
2. **Provide sensible defaults** - For optional arguments when appropriate
3. **Handle both quoted and unquoted values** - Support flexibility in argument format
4. **Trim whitespace** - Clean up parsed values before use
5. **Validate file paths** - Check existence before processing
6. **Support lists** - Use comma separation for multiple values
7. **Clear error messages** - Include usage examples in error output
8. **Document expected format** - In command file header comments