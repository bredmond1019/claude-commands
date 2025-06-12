# Claude Code Workflow Automation Commands

This directory contains custom Claude Code commands for automating development workflows with support for single and multi-agent task execution.

## Command Structure

All commands follow the naming pattern: `/user:<category>:<command>` or `/user:<command>`

## Installation

Commands are automatically available when placed in this directory with the `.md` extension.

## Available Commands

### PRD Generation & Management

#### `/user:prd:generate`
Generate a Product Requirements Document (PRD) from design documents with optional Q&A interaction.
- **Arguments**: 
  - `<design-doc-path>` - Path to the main design document (required)
  - `[reference-files...]` - Additional reference files for context (optional)
- **Example**: `/user:prd:generate project-design.md docs/api-spec.md`

#### `/user:prd:iterate-project`
Analyze completed work against PRD and generate next iteration of improvements.
- **Arguments**:
  - `--prompt <text>` - Additional prompt for analysis (optional)
  - `--prd-file <path>` - Path to PRD file (default: `tasks/project-prd.md`)
  - `--output-prd <path>` - Path for new PRD (default: `tasks/project-prd-v{N}.md`)
  - `--mode <mode>` - `improve` or `next-phase` (default: improve)
  - `--feedback-file <path>` - Path to feedback file (optional)
- **Example**: `/user:prd:iterate-project --mode next-phase --feedback-file feedback.txt`

### Task Generation

#### `/user:tasks:generate`
Generate a detailed task list from an existing Product Requirements Document (PRD).
- **Arguments**:
  - `[prd-file-path]` - Path to PRD file (optional, defaults to `tasks/project-prd.md`)
  - `--from-file <path>` - Path to main design document for reference (optional)
- **Example**: `/user:tasks:generate tasks/my-custom-prd.md --from-file project-design.md`

#### `/user:tasks:generate-multi-agent`
Distribute an existing task list across 3-5 specialized agents for parallel execution.
- **Arguments**:
  - `<task-list-file>` - Path to completed task list file (required)
  - `--agents <number>` - Number of agents (3-5, default: auto-detect)
  - `--output-dir <path>` - Directory for output files (default: `tasks/`)
- **Example**: `/user:tasks:generate-multi-agent tasks/tasks-list.md --agents 4`

### Task Execution

#### `/user:tasks:execute`
Execute tasks from a task list file with automatic progress tracking and git commits.
- **Arguments**:
  - `<task-list-file>` - Path to task list file (required)
  - `--start-from <task-id>` - Start from specific task ID (optional)
  - `--context-limit <percentage>` - Stop at context usage percentage (default: 80)
- **Example**: `/user:tasks:execute tasks/tasks-list.md --start-from 3.1`

#### `/user:tasks:execute-multi-agent`
Execute tasks in parallel across multiple Claude agents based on task distribution.
- **Arguments**:
  - `--coordination-file <path>` - Path to coordination file (default: `tasks/multi-agent-coordination.md`)
  - `--agents <list>` - Comma-separated list of agent IDs (default: all)
  - `--mode <mode>` - Execution mode - `parallel` or `sequential` (default: parallel)
- **Example**: `/user:tasks:execute-multi-agent --agents 1,3 --mode parallel`

### Task Continuation

#### `/user:tasks:continue`
Resume an interrupted workflow from a saved agent context.
- **Arguments**:
  - `[agent-id]` - Agent identifier to resume (optional)
  - `[task-file]` - Path to specific task file (optional)
  - `[context-file]` - Path to agent context file (optional)
- **Example**: `/user:tasks:continue 2`

#### `/user:tasks:continue-parallel`
Deploy multiple agents to work in parallel to complete previous tasks.
- **Arguments**:
  - `--prompt <text>` - Prompt for agents (optional)
  - `--from-file <path>` - Path to design document (optional)
- **Example**: `/user:tasks:continue-parallel --prompt "Generate detailed tasks"`

### Deployment Commands

#### `/user:deploy:multiple-agents`
Deploy multiple agents to work in parallel to generate a detailed task list.
- **Arguments**:
  - `<prompt>` - Prompt describing the task (required)
  - `--from-file <path>` - Path to main design document (optional)
  - `--coordination-file <path>` - Path to coordination file (optional)
- **Example**: `/user:deploy:multiple-agents "Generate 3 Agents to review each microservice"`

#### `/user:deploy:parallel-tasks`
Deploy multiple agents to work in parallel to generate a detailed task list.
- **Arguments**:
  - `<prompt>` - Prompt describing the task (required)
  - `--from-file <path>` - Path to main design document (optional)
- **Example**: `/user:deploy:parallel-tasks "Generate detailed task list"`

### Task Management

#### `/user:tasks:reorganize`
Reorganize and rebalance tasks between agents or within a single task list.
- **Arguments**:
  - `<source-list>` - Path to source task list (required)
  - `[target-list]` - Path to target task list (optional)
  - `[options]` - Additional options (merge, split, rebalance)
- **Example**: `/user:tasks:reorganize tasks/agent-1-list.md tasks/agent-2-list.md`

#### `/user:tasks-mark-complete`
Mark each agent's respective task list complete.
- **Example**: `/user:tasks-mark-complete`

#### `/user:tasks-commit`
Commit the tasks to the repository.
- **Example**: `/user:tasks-commit`

### Utility Commands

#### `/user:project-devops`
Deploy multiple agents to review project README, codebase, tests, and generate dev environment setup.
- **Example**: `/user:project-devops`

#### `/user:parallel-persist`
Persist and correct parallel task execution when only one agent was running.
- **Example**: `/user:parallel-persist`

## Key Features

- **Parallel Execution**: Most commands support deploying 3-5 agents simultaneously
- **Progress Tracking**: Automatic task completion tracking with checkbox updates
- **Git Integration**: Automatic commits after each subtask completion
- **Context Management**: Monitoring and handling of context usage limits
- **Dependency Management**: Intelligent handling of task dependencies between agents
- **Coordination Files**: Central tracking of multi-agent progress and handoffs
- **Resume Capability**: Ability to continue interrupted workflows from saved contexts

## Subdirectories

### `/ai-dev-tasks/`
Contains reusable rule-based versions of key commands (.mdc files):
- `create-prd.mdc` - PRD generation rule
- `generate-tasks.mdc` - Task generation rule
- `generate-multi-agent-tasks.mdc` - Multi-agent task generation rule
- `process-task-list.mdc` - Task list processing guidelines
- `prompts.md` - Reference prompts for various operations

### `/utils/`
Contains utility modules and helper functions used by the main commands:
- Agent coordination and management utilities
- Context management and monitoring
- Task parsing and distribution
- Git automation helpers
- Error handling and validation

### `/workflows/`
Contains workflow templates and examples:
- `ai-workflow.md` - AI-driven development workflow
- `basic-workflow.md` - Simple task execution workflow
- `iterative-development.md` - Iterative improvement workflow
- `multi-agent-workflow.md` - Multi-agent coordination workflow

## Usage Examples

### Complete Project Workflow
```bash
# 1. Generate PRD from design document
/user:prd:generate design.md

# 2. Generate task list from PRD
/user:tasks:generate

# 3. Distribute tasks to multiple agents
/user:tasks:generate-multi-agent tasks/tasks-list.md --agents 4

# 4. Execute tasks in parallel
/user:tasks:execute-multi-agent

# 5. Review and iterate
/user:prd:iterate-project --mode improve
```

### Single Agent Workflow
```bash
# 1. Generate tasks
/user:tasks:generate tasks/my-prd.md

# 2. Execute tasks
/user:tasks:execute tasks/tasks-list.md

# 3. Continue if interrupted
/user:tasks:continue
```