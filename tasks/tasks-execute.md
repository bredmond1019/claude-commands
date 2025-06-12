# /user:execute-tasks

Execute tasks from a task list file with automatic progress tracking and git commits.

## Usage

```
/user:tasks:execute <task-list-file> [--start-from <task-id>] [--context-limit <percentage>]
```

## Arguments

- `<task-list-file>`: Path to the task list file (e.g., `tasks/tasks-list.md` or `tasks/agent-1-tasks.md`)
- `--start-from <task-id>`: Optional. Start execution from a specific task ID (e.g., `2.3`)
- `--context-limit <percentage>`: Optional. Stop when context usage reaches this percentage (default: 80)

## Process

1. **Parse Task List**: Read and parse the markdown task list to extract all tasks
2. **Find Starting Point**: Identify uncompleted tasks or start from specified task
3. **Execute Tasks**: Work through tasks sequentially, updating checkboxes as completed
4. **Git Commits**: Automatically commit after each subtask completion
5. **Context Monitoring**: Track context usage and warn when approaching limits
6. **Progress Updates**: Update task file with completion markers in real-time

## Task Execution Flow

```python
def execute_tasks(task_list_file, start_from=None, context_limit=80):
    # 1. Parse the task list
    tasks = parse_task_list(task_list_file)

    # 2. Find starting point
    current_task = find_next_task(tasks, start_from)

    # 3. Main execution loop
    while current_task and not context_exceeded(context_limit):
        # Execute the task
        execute_single_task(current_task)

        # Mark task complete
        mark_task_complete(task_list_file, current_task)

        # Commit changes
        if is_subtask(current_task):
            commit_progress(current_task)

        # Find next task
        current_task = find_next_task(tasks)

    # 4. Final status report
    generate_completion_report(tasks)
```

## Task Parsing

The command parses markdown task lists in this format:

```markdown
- [ ] 1.0 Parent Task
  - [ ] 1.1 Subtask one
  - [x] 1.2 Subtask two (completed)
  - [ ] 1.3 Subtask three
- [x] 2.0 Another Parent (completed)
```

## Checkbox Tracking

Updates task completion status by modifying checkboxes:

- `[ ]` → `[x]` when task is completed
- Preserves existing completed tasks
- Updates file in-place without losing formatting

## Git Commit Automation

After each subtask completion:

```bash
git add <modified-files>
git commit -m "Complete task <task-id>: <task-description>

- <summary of changes>
- <files modified>

🤖 Generated with Claude Code (https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

## Context Monitoring

```python
def monitor_context():
    current_usage = get_current_context_usage()
    percentage = (current_usage / MAX_CONTEXT) * 100

    if percentage >= 90:
        print("⚠️  Context usage critical (90%+)")
        return "CRITICAL"
    elif percentage >= 80:
        print("⚠️  Context usage high (80%+)")
        return "WARNING"
    elif percentage >= 70:
        print("ℹ️  Context usage moderate (70%+)")
        return "INFO"

    return "OK"
```

## Progress Reporting

Real-time progress updates:

```
📋 Task Execution Progress
========================
Total Tasks: 15
Completed: 7 (46%)
In Progress: 1
Remaining: 7

Current Task: 3.2 - Implement user authentication
Context Usage: 65% (13,000 / 20,000 tokens)

Recent Completions:
✅ 3.1 - Set up database models
✅ 2.4 - Create API endpoints
```

## Error Handling

- **File Not Found**: Clear error if task list doesn't exist
- **Parse Errors**: Report line numbers for malformed task lists
- **Git Conflicts**: Pause execution and prompt for resolution
- **Context Overflow**: Save progress and provide resumption instructions

## Resumption Support

If execution is interrupted:

```bash
# Resume from where you left off
/user:execute-tasks tasks/tasks-list.md --start-from 3.3

# Or use the continue command
/user:continue-tasks
```

## Example Usage

```bash
# Execute all tasks in the main list
/user:execute-tasks tasks/tasks-list.md

# Execute agent-specific tasks
/user:execute-tasks tasks/agent-2-tasks.md

# Start from a specific task
/user:execute-tasks tasks/tasks-list.md --start-from 3.1

# Set conservative context limit
/user:execute-tasks tasks/tasks-list.md --context-limit 70
```

## Integration with Multi-Agent Workflow

When used in multi-agent scenarios:

1. Each agent executes their own task list
2. Coordination file tracks overall progress
3. Dependencies are checked before task execution
4. Handoff points trigger notifications

## Notes

- Tasks are executed in order, respecting dependencies
- Parent tasks are marked complete only after all subtasks finish
- Files are saved after each task to preserve progress
- Context monitoring helps prevent mid-task interruptions
- Git commits provide rollback capability if needed
