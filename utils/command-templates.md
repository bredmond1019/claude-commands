# Claude Code Command Templates

This file contains reusable templates for creating Claude Code commands in the workflow automation system.

## Basic Command Template

```markdown
# Command: /user:<command-name>

**Purpose:** Brief description of what this command does

**Variables:**
var1: $ARGUMENTS
var2: $ARGUMENTS
var3: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `var1` - Description (required/optional)
2. `var2` - Description (required/optional, default: value)
3. `var3` - Description (required/optional)

**PHASE 1: VALIDATION**
Validate all arguments and check prerequisites:
- Ensure required arguments are provided
- Verify file paths exist
- Check permissions and access
- Validate argument formats

**PHASE 2: EXECUTION**
Main execution logic:
1. Step 1 description
2. Step 2 description
3. Step 3 description

**PHASE 3: OUTPUT**
Generate output and handle results:
- Save files to specified locations
- Display success messages
- Clean up temporary resources

**ERROR HANDLING:**
- Missing arguments: Display usage and exit
- File not found: Display error with path
- Process failure: Rollback changes if needed
```

## Standard Argument Patterns

### Single File Input
```markdown
**Variables:**
input_file: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `input_file` - Path to input file (required)

Extract input_file from arguments. If not provided, display:
"Usage: /user:command input_file=\"path/to/file.md\""
```

### Multiple File Inputs
```markdown
**Variables:**
design_doc: $ARGUMENTS
ref_files: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `design_doc` - Path to the design document (required)
2. `ref_files` - Comma-separated list of reference files (optional)

Example: /user:command design_doc="design.md" ref_files="ref1.md,ref2.md,ref3.md"
```

### Input/Output Pattern
```markdown
**Variables:**
input_file: $ARGUMENTS
output_dir: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `input_file` - Path to input file (required)
2. `output_dir` - Directory for output (optional, default: ./output)
```

### Task Execution Pattern
```markdown
**Variables:**
task_list: $ARGUMENTS
max_tasks: $ARGUMENTS
agent_id: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `task_list` - Path to task list file (required)
2. `max_tasks` - Maximum tasks to execute (optional, default: all)
3. `agent_id` - Agent identifier for multi-agent mode (optional)
```

## Command Categories

### 1. Generation Commands Template

For commands that generate new files (PRDs, task lists, etc.):

```markdown
# Command: /user:generate-<type>

**Purpose:** Generate a <type> from input specifications

**Variables:**
input_spec: $ARGUMENTS
output_file: $ARGUMENTS
template: $ARGUMENTS
interactive: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `input_spec` - Path to input specification (required)
2. `output_file` - Path for generated output (optional, default: tasks/<type>.md)
3. `template` - Path to custom template (optional)
4. `interactive` - Enable Q&A mode (optional, default: true)

**PHASE 1: LOAD INPUTS**
1. Read input specification from `input_spec`
2. Load template if provided, otherwise use default
3. Prepare output directory

**PHASE 2: INTERACTIVE PROCESSING** (if interactive=true)
1. Analyze input and identify clarifications needed
2. Present questions to user
3. Incorporate responses into generation context

**PHASE 3: GENERATION**
1. Apply template with input data
2. Generate structured output
3. Validate output format

**PHASE 4: SAVE OUTPUT**
1. Create output directory if needed
2. Write generated content to `output_file`
3. Display success message with file location
```

### 2. Execution Commands Template

For commands that execute tasks or run processes:

```markdown
# Command: /user:execute-<type>

**Purpose:** Execute <type> tasks from a task list

**Variables:**
task_list: $ARGUMENTS
context_file: $ARGUMENTS
max_tasks: $ARGUMENTS
continue_from: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `task_list` - Path to task list file (required)
2. `context_file` - Path to context/state file (optional)
3. `max_tasks` - Number of tasks to execute (optional, default: all)
4. `continue_from` - Task ID to continue from (optional)

**PHASE 1: INITIALIZATION**
1. Load task list from file
2. Load previous context if provided
3. Identify starting point (first task or continue_from)

**PHASE 2: TASK EXECUTION LOOP**
For each task (up to max_tasks):
1. Parse task definition and requirements
2. Execute task actions
3. Update task status with [x]
4. Commit changes after each subtask
5. Monitor context capacity

**PHASE 3: STATE MANAGEMENT**
1. Save current context to context_file
2. Update task list with completion markers
3. Log execution summary

**PHASE 4: COMPLETION**
1. Display execution summary
2. Save final state
3. Provide next steps if tasks remain
```

### 3. Coordination Commands Template

For commands that manage multi-agent workflows:

```markdown
# Command: /user:coordinate-<action>

**Purpose:** Coordinate <action> across multiple agents

**Variables:**
coordination_file: $ARGUMENTS
agent_lists: $ARGUMENTS
action: $ARGUMENTS

**ARGUMENTS PARSING:**
Parse the following arguments from "$ARGUMENTS":
1. `coordination_file` - Path to coordination file (required)
2. `agent_lists` - Comma-separated agent task lists (required)
3. `action` - Coordination action to perform (required)

**PHASE 1: LOAD COORDINATION STATE**
1. Read multi-agent-coordination.md
2. Parse current agent assignments
3. Load all agent task lists

**PHASE 2: ANALYZE CURRENT STATE**
1. Check completion status per agent
2. Identify bottlenecks or idle agents
3. Calculate workload distribution

**PHASE 3: PERFORM COORDINATION**
Based on action type:
- redistribute: Balance tasks across agents
- merge: Combine agent lists
- spawn: Create new agent assignments
- sync: Update coordination file

**PHASE 4: UPDATE STATE**
1. Write updated agent task lists
2. Update coordination file
3. Commit all changes
4. Notify affected agents
```

## Integration Patterns

### Git Integration
```markdown
**GIT OPERATIONS:**
After each significant operation:
1. Stage modified files: `git add <files>`
2. Commit with descriptive message: `git commit -m "Type: Description"`
3. Include task reference in commit message
```

### File Validation
```markdown
**FILE VALIDATION:**
Before processing files:
1. Check file exists: `if [[ ! -f "$file" ]]`
2. Verify read permissions: `if [[ ! -r "$file" ]]`
3. Validate file format/extension
4. Check file size limits
```

### Context Management
```markdown
**CONTEXT MONITORING:**
Track context usage throughout execution:
1. Monitor token count periodically
2. Implement checkpoint saves
3. Prepare for agent spawning if needed
4. Save state before context limit
```

## Error Handling Standards

```markdown
**ERROR HANDLING:**
1. **Missing Arguments**
   - Display clear usage examples
   - List all required and optional arguments
   - Exit gracefully

2. **File Errors**
   - File not found: Show attempted path
   - Permission denied: Suggest fixes
   - Format errors: Show expected format

3. **Execution Errors**
   - Save partial progress
   - Log error details
   - Provide recovery instructions
   - Rollback if necessary

4. **Context Overflow**
   - Save current state
   - Display continuation command
   - Prepare handoff to new agent
```

## Usage Examples

### Simple Command
```bash
/user:generate-prd design_doc="specs/feature.md"
```

### Complex Command
```bash
/user:execute-multi-agent-tasks \
  task_lists="tasks/agent-1-list.md,tasks/agent-2-list.md" \
  coordination_file="tasks/multi-agent-coordination.md" \
  max_parallel="3"
```

### Continuation Command
```bash
/user:continue-tasks \
  agent_id="2" \
  context_file="tasks/agent-contexts/agent-2-context.md"
```