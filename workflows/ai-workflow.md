# AI Workflow

## Goal

Generate a series of markdown files that can be used to guide an AI assistant in a common workflow.

## Usage

```
/user:ai-workflow <prompt> [--design-doc <path>] [--auto-execute]
```

## Arguments

- `<prompt>`: Prompt describing the task for the agents that will be generated (required)
- `--design-doc <path>`: Path to the design document (optional)
- `--auto-execute`: Automatically execute the tasks (optional)

Example: `/user:ai-workflow "Generate a detailed task list for a new project" --design-doc project-design.md --auto-execute`

## Process

1. Generate a PRD based on a design doc or prompt
2. Generate a list of tasks (single or multi-agent)
3. Execute the tasks (single or multi-agent in parallel)
4. Analyze the codebase based on the results

## Output

- `project-prd.md` - A PRD to the Project
- `tasks/tasks-list.md` - A list of tasks for a single or multiple agents.
- `tasks/agent-<X: int>-list.md` - A list of tasks for a single agent.
- `tasks/multi-agent-coordination.md` - A coordination file for multiple agents.

## Workflow

### Generating a PRD

1. Review the design doc or prompt
2. Generate a PRD using @ai-dev-tasks/generate-prd.mdc

#### Notes

- LLM will ask the user for questions.
- If `--auto-execute` is provided, the LLM should answer his own questions.
- If `--auto-execute` is not provided, the user should respond to the questions in the chat.
- LLM will generate a document called `project-prd.md`

### Generating Tasks from a PRD

1. If `--auto-execute` is not provided, pause for the user to review the PRD.
2. Review the PRD in `tasks/project-prd.md`
3. Generate a list of tasks using @ai-dev-tasks/generate-tasks.mdc
4. If `--auto-execute` is provided, the LLM should answer his own questions.
5. If `--auto-execute` is not provided, the user should respond to the questions in the chat.
6. LLM will generate a file called single list of tasks: `tasks/tasks-list.md`
7. At first only higher level tasks are created.
8. Then the user is asked to review then type 'Go'
   8a. If `--auto-execute` is provided, the LLM should continue to the next step.
9. Then the LLM will create a list of subtasks for each task.

### Generating Multi-Agent Tasks

1. Review `tasks/project-prd.md`
2. Using `tasks/tasks-list.md`, create a list of tasks for each agent.
3. Create tasks using @ai-dev-tasks/generate-multi-agent-tasks.mdc

#### Notes

- If `--auto-execute` is provided, the LLM should continue to the next step.
- If `--auto-execute` is not provided, the user should respond to the questions in the chat.
- LLM will generate a file called `tasks/multi-agent-coordination.md`

### Executing Tasks

1. Generating Agent Prompts

- Should generate a prompt for each agent based on how many task lists are provided
- Prompt Structure:

```markdown
<agent-prompt-structure>

You are Agent <X: int> responsible for <your-focus-area>. Complete these <Y> tasks from @tasks/agent-<X: int>-list.md.

Docs to Reference:

- `Project Design Doc` : @tasks/project-design-doc.md
- `Original Task List` : @tasks/tasks-list.md
- `Agent Task List` : @tasks/agent-<X: int>-list.md
- `Multi-Agent Coordination` : @tasks/multi-agent-coordination.md

**Tasks:**

**Your focus areas:**

**Key requirements:**

**Dependencies:**

**Success criteria:**

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them on both:
  - the `Original Task List`
  - the `Agent Task List`

</agent-prompt-structure>
```

2. Initialize Agents

- For each Agent:
  - Spin up a new instance of claude with approving all permissions.
  - Take each Agent Prompt, and feed it to the new instance of claude.
  - Agents should be aware of context_capacity,
  - Spinning up new agents if needed to avoid too much context in the chat window.

3. Continue Tasks

- Each Agent needs to be provided context from one session to the next for the same Agent.
- Each Agent should be able to continue their tasks from the last session.
- Continue Tasks Prompt:

```markdown
<continue-prompt>
- Go back and check off all completed tasks by Agent <X: int> thus far on @tasks/agent-<X: int>-list.md
- Then continue onto the next task for Agent <X: int> on @tasks/agent-<X: int>-list.md
- After each subtask is completed, git commit for only the files you've modifies/created
- Then mark the subtask complete on both:
  - `Original Task List` : @tasks/tasks-list.md
  - `Agent Task List` : @tasks/agent-<X: int>-list.md
</continue-prompt>
```

4. Agent <X: int> Completes All Tasks

- Send Agent <X: int> to review all other agents' task lists.
- Determine which agent has the most tasks remaining -> Agent <Y: int>
- Review remaining tasks for Agent <Y: int> on @tasks/agent-<Y: int>-list.md
- Determine a few tasks of Agent <Y: int> that can be moved to Agent <X: int>'s list
  - These tasks should not interfere with Agent <Y: int>'s current tasks.
- Move the tasks to Agent <X: int>'s list and remove from Agent <Y: int>'s list.
- Send Agent <X: int> to continue their tasks.

5. All Agents Complete All Tasks

- Review all agents' task lists
- Determine if any tasks are left:
  - If so, send the task list back to the LLM to be reorganized
  - If not, the project is complete

6. Project is Complete

- Review the project-prd.md
- Determine if the project is complete
  - If so, the project is complete
  - If not, Generate a new PRD for missng features or next phases
- Update the README.md and other important project documentation files to match all the updates.
