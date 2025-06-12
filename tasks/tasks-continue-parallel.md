# Deploy Parallel Tasks Command

Deploy multiple agents to work in parallel to complete the previous tasks.

## Usage

```
/user:tasks:continue-parallel [--prompt <text>] [--from-file <path>]
```

## Arguments

- `--prompt <text>`: Prompt describing the task for the agents that will be generated. (optional)
- `--from-file <path>`: Path to the main design document/prompt to use as a reference (optional)

Example: `/user:tasks:continue-parallel --prompt "Generate a detailed task list for a new project" --from-file project-design.md`

## Process

1. **Review Prompt**: Review the prompt to ensure it is clear and concise.
2. **Generate Multiple Agents**: Generate multiple agents to work in parallel on the `<prompt>`.
3. **Execute Tasks**: Execute the tasks in parallel.

Please continue where the agents left off. Spin up several instances of Claude Code in Parallel to complete tasks from their respective lists. Please make sure their current task doesn't interfere with another agents so they can all work in parallel.
{ <prompt> }
{ <from-file> }

**Parallel Execution Management:**
Deploy multiple Sub Agents to generate agents in parallel for maximum efficiency and creative
diversity:

- Launch 3-5 Sub Agents simultaneously using Task tool
- Monitor agent progress and completion
- Commit Frequent and Detailed Progress
- Collect and validate all completed agents
