# 🤖 Claude Code Workflow Automation Commands

> **Automate your development workflow** with custom Claude Code commands supporting single and multi-agent task execution

---

## 📁 Command Structure

All commands follow the naming pattern: `/user:<category>:<command>` or `/user:<command>`

## 🚀 Installation

Commands are automatically available when placed in this directory with the `.md` extension.

---

## 📋 Available Commands

### 📄 PRD Generation & Management

#### 🔧 `/user:prd:generate`
**Generate a Product Requirements Document (PRD) from design documents with optional Q&A interaction.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `<design-doc-path>` | **Required** | Path to the main design document |
| `[reference-files...]` | *Optional* | Additional reference files for context |

**📝 Example:** `/user:prd:generate project-design.md docs/api-spec.md`

#### 🔄 `/user:prd:iterate-project`
**Analyze completed work against PRD and generate next iteration of improvements.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `--prompt <text>` | *Optional* | Additional prompt for analysis |
| `--prd-file <path>` | *Optional* | Path to PRD file (default: `tasks/project-prd.md`) |
| `--output-prd <path>` | *Optional* | Path for new PRD (default: `tasks/project-prd-v{N}.md`) |
| `--mode <mode>` | *Optional* | `improve` or `next-phase` (default: improve) |
| `--feedback-file <path>` | *Optional* | Path to feedback file |

**📝 Example:** `/user:prd:iterate-project --mode next-phase --feedback-file feedback.txt`

---

### 📋 Task Generation

#### ✏️ `/user:tasks:generate`
**Generate a detailed task list from an existing Product Requirements Document (PRD).**

| Parameter | Type | Description |
|-----------|------|-------------|
| `[prd-file-path]` | *Optional* | Path to PRD file (defaults to `tasks/project-prd.md`) |
| `--from-file <path>` | *Optional* | Path to main design document for reference |

**📝 Example:** `/user:tasks:generate tasks/my-custom-prd.md --from-file project-design.md`

#### 🤖 `/user:tasks:generate-multi-agent`
**Distribute an existing task list across 3-5 specialized agents for parallel execution.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `<task-list-file>` | **Required** | Path to completed task list file |
| `--agents <number>` | *Optional* | Number of agents (3-5, default: auto-detect) |
| `--output-dir <path>` | *Optional* | Directory for output files (default: `tasks/`) |

**📝 Example:** `/user:tasks:generate-multi-agent tasks/tasks-list.md --agents 4`

---

### ⚡ Task Execution

#### 🚀 `/user:tasks:execute`
**Execute tasks from a task list file with automatic progress tracking and git commits.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `<task-list-file>` | **Required** | Path to task list file |
| `--start-from <task-id>` | *Optional* | Start from specific task ID |
| `--context-limit <percentage>` | *Optional* | Stop at context usage percentage (default: 80) |

**📝 Example:** `/user:tasks:execute tasks/tasks-list.md --start-from 3.1`

#### 🔄 `/user:tasks:execute-multi-agent`
**Execute tasks in parallel across multiple Claude agents based on task distribution.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `--coordination-file <path>` | *Optional* | Path to coordination file (default: `tasks/multi-agent-coordination.md`) |
| `--agents <list>` | *Optional* | Comma-separated list of agent IDs (default: all) |
| `--mode <mode>` | *Optional* | Execution mode - `parallel` or `sequential` (default: parallel) |

**📝 Example:** `/user:tasks:execute-multi-agent --agents 1,3 --mode parallel`

---

### 🔄 Task Continuation

#### ▶️ `/user:tasks:continue`
**Resume an interrupted workflow from a saved agent context.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `[agent-id]` | *Optional* | Agent identifier to resume |
| `[task-file]` | *Optional* | Path to specific task file |
| `[context-file]` | *Optional* | Path to agent context file |

**📝 Example:** `/user:tasks:continue 2`

#### ⚡ `/user:tasks:continue-parallel`
**Deploy multiple agents to work in parallel to complete previous tasks.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `--prompt <text>` | *Optional* | Prompt for agents |
| `--from-file <path>` | *Optional* | Path to design document |

**📝 Example:** `/user:tasks:continue-parallel --prompt "Generate detailed tasks"`

---

### 🚀 Deployment Commands

#### 🤖 `/user:deploy:multiple-agents`
**Deploy multiple agents to work in parallel to generate a detailed task list.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `<prompt>` | **Required** | Prompt describing the task |
| `--from-file <path>` | *Optional* | Path to main design document |
| `--coordination-file <path>` | *Optional* | Path to coordination file |

**📝 Example:** `/user:deploy:multiple-agents "Generate 3 Agents to review each microservice"`

#### ⚡ `/user:deploy:parallel-tasks`
**Deploy multiple agents to work in parallel to generate a detailed task list.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `<prompt>` | **Required** | Prompt describing the task |
| `--from-file <path>` | *Optional* | Path to main design document |

**📝 Example:** `/user:deploy:parallel-tasks "Generate detailed task list"`

---

### 🔧 Task Management

#### ⚖️ `/user:tasks:reorganize`
**Reorganize and rebalance tasks between agents or within a single task list.**

| Parameter | Type | Description |
|-----------|------|-------------|
| `<source-list>` | **Required** | Path to source task list |
| `[target-list]` | *Optional* | Path to target task list |
| `[options]` | *Optional* | Additional options (merge, split, rebalance) |

**📝 Example:** `/user:tasks:reorganize tasks/agent-1-list.md tasks/agent-2-list.md`

#### ✅ `/user:tasks-mark-complete`
**Mark each agent's respective task list complete.**

**📝 Example:** `/user:tasks-mark-complete`

#### 💾 `/user:tasks-commit`
**Commit the tasks to the repository.**

**📝 Example:** `/user:tasks-commit`

---

### 🛠️ Utility Commands

#### 🔍 `/user:project-devops`
**Deploy multiple agents to review project README, codebase, tests, and generate dev environment setup.**

**📝 Example:** `/user:project-devops`

#### 🔄 `/user:parallel-persist`
**Persist and correct parallel task execution when only one agent was running.**

**📝 Example:** `/user:parallel-persist`

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| ⚡ **Parallel Execution** | Most commands support deploying 3-5 agents simultaneously |
| 📊 **Progress Tracking** | Automatic task completion tracking with checkbox updates |
| 🔀 **Git Integration** | Automatic commits after each subtask completion |
| 🧠 **Context Management** | Monitoring and handling of context usage limits |
| 🔗 **Dependency Management** | Intelligent handling of task dependencies between agents |
| 📋 **Coordination Files** | Central tracking of multi-agent progress and handoffs |
| 🔄 **Resume Capability** | Ability to continue interrupted workflows from saved contexts |

---

## 📁 Subdirectories

### 🤖 `/ai-dev-tasks/`
**Contains reusable rule-based versions of key commands (.mdc files):**

| File | Purpose |
|------|---------|
| 📄 `create-prd.mdc` | PRD generation rule |
| 📋 `generate-tasks.mdc` | Task generation rule |
| 🤖 `generate-multi-agent-tasks.mdc` | Multi-agent task generation rule |
| 📝 `process-task-list.mdc` | Task list processing guidelines |
| 💬 `prompts.md` | Reference prompts for various operations |

### 🔧 `/utils/`
**Contains utility modules and helper functions used by the main commands:**
- 🤝 Agent coordination and management utilities
- 🧠 Context management and monitoring
- 📋 Task parsing and distribution
- 🔀 Git automation helpers
- ⚠️ Error handling and validation

### 🔄 `/workflows/`
**Contains workflow templates and examples:**

| File | Purpose |
|------|---------|
| 🤖 `ai-workflow.md` | AI-driven development workflow |
| 📋 `basic-workflow.md` | Simple task execution workflow |
| 🔄 `iterative-development.md` | Iterative improvement workflow |
| 🤖 `multi-agent-workflow.md` | Multi-agent coordination workflow |

---

## 📖 Usage Examples

### 🎯 Complete Project Workflow
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

### 👤 Single Agent Workflow
```bash
# 1. Generate tasks
/user:tasks:generate tasks/my-prd.md

# 2. Execute tasks
/user:tasks:execute tasks/tasks-list.md

# 3. Continue if interrupted
/user:tasks:continue
```

---

## 🚀 Getting Started

1. 📄 **Create a design document** describing your project
2. ⌨️ **Run the workflow commands** to generate and distribute tasks
3. 🤖 **Deploy agents** to work in parallel
4. 📊 **Monitor progress** through coordination files
5. ✅ **Complete and iterate** on your project

> 💡 **Tip:** Start with the Complete Project Workflow above for the best experience with multi-agent development!

---

### 🎯 Quick Links

- 📖 [Multi-Agent Development Guide](Multi-Agent-Development-Guide.md) - Complete introduction to multi-agent development
- 🔬 [Multi-Agent Workflow Tutorial](Multi-Agent-Workflow-Explained.md) - Step-by-step explanation of the workflow system
- 🚀 [Multi-Agent Quickstart Guide](Multi-Agent-Quickstart-Guide.md) - Get up and running quickly

**🤖 Transform your development with AI agents working in parallel!**