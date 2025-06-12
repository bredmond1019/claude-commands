# Basic Workflow Example

This example demonstrates the simplest workflow for a single developer working on a small project.

## Scenario

You have a design document for a TODO application and want to implement it using the Claude workflow automation system.

## Step 1: Create Your Design Document

First, create a design document describing your project:

```markdown
# TODO Application Design

## Overview
A simple command-line TODO application that helps users manage their daily tasks.

## Features
1. Add new tasks with descriptions
2. Mark tasks as complete
3. List all tasks with filters
4. Delete tasks
5. Save tasks to a file

## Technical Requirements
- Written in Python
- Uses JSON for data storage
- Command-line interface
- Unit tests for core functionality
```

Save this as `todo-design.md`.

## Step 2: Generate PRD

Generate a formal PRD from your design document:

```bash
/user:generate-prd todo-design.md
```

This will:
- Read your design document
- Optionally ask clarifying questions
- Generate a structured PRD at `tasks/project-prd.md`

## Step 3: Generate Task List

Create a task list from the PRD:

```bash
/user:generate-tasks
```

When prompted, review the high-level tasks and type "Go" to generate subtasks.

Output: `tasks/tasks-list.md`

## Step 4: Execute Tasks

Execute all tasks with a single agent:

```bash
/user:execute-tasks
```

The system will:
- Read tasks from `tasks/tasks-list.md`
- Execute each task in order
- Mark completed tasks with [x]
- Commit changes after each subtask
- Monitor context usage

## Step 5: Check Progress

If you need to stop and resume later:

```bash
# Check current status
cat tasks/tasks-list.md | grep -c "\[x\]"  # Completed
cat tasks/tasks-list.md | grep -c "\[ \]"  # Remaining

# Resume work
/user:continue-tasks
```

## Step 6: Iterate and Improve

After completing the initial implementation:

```bash
/user:iterate-project
```

This will:
- Analyze completed work against the PRD
- Generate recommendations
- Create an updated PRD for the next iteration

## Complete Example Session

```bash
# 1. Setup project
mkdir todo-app
cd todo-app
git init

# 2. Create design document
cat > todo-design.md << 'EOF'
# TODO Application Design
[... design content ...]
EOF

# 3. Generate PRD
/user:generate-prd todo-design.md

# 4. Generate tasks
/user:generate-tasks
# Type "Go" when prompted

# 5. Execute tasks
/user:execute-tasks

# 6. Check progress
ls -la  # See created files
git log --oneline  # See commits

# 7. Iterate for improvements
/user:iterate-project mode=improve
```

## Expected Output Structure

After running the workflow:

```
todo-app/
├── todo-design.md          # Your original design
├── tasks/
│   ├── project-prd.md      # Generated PRD
│   ├── tasks-list.md       # Task list with progress
│   └── iteration-analysis-*.md  # Analysis reports
├── src/
│   ├── todo.py             # Main application
│   ├── storage.py          # Data storage module
│   └── cli.py              # CLI interface
├── tests/
│   ├── test_todo.py        # Unit tests
│   └── test_storage.py
└── README.md               # Generated documentation
```

## Tips for Basic Workflow

1. **Start Small**: Begin with a simple design document
2. **Review Generated Content**: Always review the PRD and task list before execution
3. **Commit Often**: The system commits after each subtask automatically
4. **Monitor Progress**: Check task completion regularly
5. **Iterate**: Use the iteration command to improve your project

## Common Commands

| Command | Purpose |
|---------|---------|
| `/user:generate-prd design.md` | Create PRD from design |
| `/user:generate-tasks` | Generate task list |
| `/user:execute-tasks` | Run all tasks |
| `/user:continue-tasks` | Resume interrupted work |
| `/user:iterate-project` | Analyze and improve |

## Next Steps

Once comfortable with the basic workflow, explore:
- [Multi-Agent Workflow](multi-agent-workflow.md) for larger projects
- [Iterative Development](iterative-development.md) for continuous improvement
- [Team Collaboration](team-collaboration.md) for working with others