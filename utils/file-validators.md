# File Validation Utilities

This file contains reusable utility functions for file path validation and error handling in Claude Code commands.

## Basic File Validation Functions

### Check File Exists
```bash
# Validate that a file exists
validate_file_exists() {
    local file_path="$1"
    local file_description="${2:-file}"
    
    if [[ -z "$file_path" ]]; then
        echo "ERROR: No $file_description path provided"
        return 1
    fi
    
    if [[ ! -f "$file_path" ]]; then
        echo "ERROR: $file_description not found: $file_path"
        return 1
    fi
    
    return 0
}

# Usage
if ! validate_file_exists "$input_file" "input file"; then
    exit 1
fi
```

### Check Directory Exists
```bash
# Validate that a directory exists or create it
validate_directory() {
    local dir_path="$1"
    local create_if_missing="${2:-false}"
    
    if [[ -z "$dir_path" ]]; then
        echo "ERROR: No directory path provided"
        return 1
    fi
    
    if [[ ! -d "$dir_path" ]]; then
        if [[ "$create_if_missing" == "true" ]]; then
            mkdir -p "$dir_path"
            echo "Created directory: $dir_path"
        else
            echo "ERROR: Directory not found: $dir_path"
            return 1
        fi
    fi
    
    return 0
}

# Usage
validate_directory "$output_dir" true  # Create if missing
validate_directory "$input_dir" false  # Error if missing
```

### Validate File Extension
```bash
# Check if file has expected extension
validate_file_extension() {
    local file_path="$1"
    local expected_ext="$2"
    
    local actual_ext="${file_path##*.}"
    
    if [[ "$actual_ext" != "$expected_ext" ]]; then
        echo "ERROR: Expected .$expected_ext file, got .$actual_ext: $file_path"
        return 1
    fi
    
    return 0
}

# Usage
if ! validate_file_extension "$task_file" "md"; then
    exit 1
fi
```

## Advanced Validation Functions

### Validate Multiple Files
```bash
# Validate a comma-separated list of files
validate_file_list() {
    local file_list="$1"
    local file_description="${2:-files}"
    local -a invalid_files=()
    
    if [[ -z "$file_list" ]]; then
        return 0  # Empty list is valid (optional files)
    fi
    
    # Split by comma
    IFS=',' read -ra FILES <<< "$file_list"
    for file in "${FILES[@]}"; do
        # Trim whitespace
        file=$(echo "$file" | xargs)
        
        if [[ ! -f "$file" ]]; then
            invalid_files+=("$file")
        fi
    done
    
    if [[ ${#invalid_files[@]} -gt 0 ]]; then
        echo "ERROR: The following $file_description were not found:"
        for file in "${invalid_files[@]}"; do
            echo "  - $file"
        done
        return 1
    fi
    
    return 0
}

# Usage
if ! validate_file_list "$ref_files" "reference files"; then
    exit 1
fi
```

### Validate File Permissions
```bash
# Check file permissions
validate_file_permissions() {
    local file_path="$1"
    local required_perms="$2"  # r, w, x, or combinations
    
    if [[ ! -f "$file_path" ]]; then
        echo "ERROR: File not found: $file_path"
        return 1
    fi
    
    # Check each permission
    if [[ "$required_perms" == *"r"* ]] && [[ ! -r "$file_path" ]]; then
        echo "ERROR: No read permission for: $file_path"
        return 1
    fi
    
    if [[ "$required_perms" == *"w"* ]] && [[ ! -w "$file_path" ]]; then
        echo "ERROR: No write permission for: $file_path"
        return 1
    fi
    
    if [[ "$required_perms" == *"x"* ]] && [[ ! -x "$file_path" ]]; then
        echo "ERROR: No execute permission for: $file_path"
        return 1
    fi
    
    return 0
}

# Usage
validate_file_permissions "$script_file" "rx"  # Read and execute
validate_file_permissions "$output_file" "w"   # Write
```

### Validate Path Safety
```bash
# Ensure path doesn't contain dangerous patterns
validate_path_safety() {
    local path="$1"
    
    # Check for path traversal attempts
    if [[ "$path" == *".."* ]]; then
        echo "ERROR: Path traversal detected in: $path"
        return 1
    fi
    
    # Check for absolute paths outside project
    if [[ "$path" == /* ]] && [[ "$path" != "$PWD"* ]]; then
        echo "WARNING: Absolute path outside project directory: $path"
        # Optionally return 1 to block
    fi
    
    # Check for special characters that might cause issues
    if [[ "$path" =~ [^a-zA-Z0-9/_.-] ]]; then
        echo "WARNING: Path contains special characters: $path"
    fi
    
    return 0
}
```

## Error Handling Utilities

### Safe File Operations
```bash
# Safely read a file with error handling
safe_read_file() {
    local file_path="$1"
    local content=""
    
    if ! validate_file_exists "$file_path"; then
        return 1
    fi
    
    if ! validate_file_permissions "$file_path" "r"; then
        return 1
    fi
    
    # Read file content
    content=$(<"$file_path" 2>/dev/null)
    local read_status=$?
    
    if [[ $read_status -ne 0 ]]; then
        echo "ERROR: Failed to read file: $file_path"
        return 1
    fi
    
    echo "$content"
    return 0
}

# Usage
if content=$(safe_read_file "$input_file"); then
    echo "File content loaded successfully"
else
    exit 1
fi
```

### Safe File Writing
```bash
# Safely write to a file with backup
safe_write_file() {
    local file_path="$1"
    local content="$2"
    local backup="${3:-true}"
    
    # Ensure directory exists
    local dir_path=$(dirname "$file_path")
    if ! validate_directory "$dir_path" true; then
        return 1
    fi
    
    # Create backup if file exists
    if [[ -f "$file_path" ]] && [[ "$backup" == "true" ]]; then
        cp "$file_path" "${file_path}.bak"
        echo "Backup created: ${file_path}.bak"
    fi
    
    # Write content
    if echo "$content" > "$file_path" 2>/dev/null; then
        echo "File written successfully: $file_path"
        return 0
    else
        echo "ERROR: Failed to write file: $file_path"
        # Restore backup if write failed
        if [[ -f "${file_path}.bak" ]]; then
            mv "${file_path}.bak" "$file_path"
            echo "Restored from backup"
        fi
        return 1
    fi
}
```

## Comprehensive Validation Function

```bash
# All-in-one file validation
validate_file_comprehensive() {
    local file_path="$1"
    local options="$2"  # JSON-like: "exists:true,ext:md,perms:r"
    
    # Parse options
    local check_exists="true"
    local expected_ext=""
    local required_perms=""
    
    if [[ "$options" == *"exists:false"* ]]; then
        check_exists="false"
    fi
    
    if [[ "$options" =~ ext:([^,]+) ]]; then
        expected_ext="${BASH_REMATCH[1]}"
    fi
    
    if [[ "$options" =~ perms:([^,]+) ]]; then
        required_perms="${BASH_REMATCH[1]}"
    fi
    
    # Perform validations
    if [[ "$check_exists" == "true" ]] && ! validate_file_exists "$file_path"; then
        return 1
    fi
    
    if [[ -n "$expected_ext" ]] && ! validate_file_extension "$file_path" "$expected_ext"; then
        return 1
    fi
    
    if [[ -n "$required_perms" ]] && ! validate_file_permissions "$file_path" "$required_perms"; then
        return 1
    fi
    
    if ! validate_path_safety "$file_path"; then
        return 1
    fi
    
    return 0
}

# Usage
validate_file_comprehensive "$input_file" "exists:true,ext:md,perms:r"
```

## Task File Specific Validators

### Validate Task List Format
```bash
# Check if file is a valid task list
validate_task_list() {
    local file_path="$1"
    
    if ! validate_file_exists "$file_path" "task list"; then
        return 1
    fi
    
    # Check for task markers
    if ! grep -q "^- \[[ x]\]" "$file_path"; then
        echo "WARNING: No task items found in: $file_path"
    fi
    
    # Check for required sections
    if ! grep -q "## Tasks" "$file_path"; then
        echo "WARNING: Missing '## Tasks' section in: $file_path"
    fi
    
    return 0
}
```

### Validate PRD Format
```bash
# Check if file is a valid PRD
validate_prd_format() {
    local file_path="$1"
    local required_sections=("Introduction" "Goals" "User Stories" "Functional Requirements")
    local missing_sections=()
    
    if ! validate_file_exists "$file_path" "PRD"; then
        return 1
    fi
    
    # Check for required sections
    for section in "${required_sections[@]}"; do
        if ! grep -q "## $section" "$file_path"; then
            missing_sections+=("$section")
        fi
    done
    
    if [[ ${#missing_sections[@]} -gt 0 ]]; then
        echo "WARNING: PRD missing required sections:"
        for section in "${missing_sections[@]}"; do
            echo "  - $section"
        done
    fi
    
    return 0
}
```

## Error Recovery Functions

```bash
# Attempt to recover from common errors
attempt_recovery() {
    local error_type="$1"
    local context="$2"
    
    case "$error_type" in
        "file_not_found")
            echo "Attempting to locate file..."
            # Try common locations
            local possible_paths=(
                "tasks/$context"
                "./$context"
                "../$context"
            )
            for path in "${possible_paths[@]}"; do
                if [[ -f "$path" ]]; then
                    echo "Found at: $path"
                    echo "$path"
                    return 0
                fi
            done
            ;;
            
        "permission_denied")
            echo "Attempting to fix permissions..."
            if chmod +r "$context" 2>/dev/null; then
                echo "Permissions fixed"
                return 0
            fi
            ;;
            
        "directory_missing")
            echo "Creating missing directory..."
            if mkdir -p "$context" 2>/dev/null; then
                echo "Directory created"
                return 0
            fi
            ;;
    esac
    
    return 1
}
```

## Usage in Commands

When using these validators in Claude Code commands:

```markdown
**FILE VALIDATION:**
Before processing:
1. Validate all input files exist
2. Check file extensions match expected types
3. Verify read/write permissions as needed
4. Ensure output directories exist or can be created

Use the validation utilities:
- validate_file_exists() for required files
- validate_file_list() for comma-separated file arguments
- validate_directory() with create_if_missing for output paths
- safe_read_file() and safe_write_file() for I/O operations
```