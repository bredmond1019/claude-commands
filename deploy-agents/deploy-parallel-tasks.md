# Deploy Parallel Tasks Command

Deploy multiple agents to work in parallel to generate a detailed task list.

## Usage

```
/user:deploy:parallel-tasks <prompt> [--from-file <path>]
```

## Arguments

- `<prompt>`: Prompt describing the task for the agents that will be generated (required)
- `--from-file <path>`: Path to the main design document/prompt to use as a reference (optional)

Example: `/user:deploy:parallel-tasks "Generate a detailed task list for a new project" --from-file project-design.md`

## Process

1. **Review Prompt**: Review the prompt to ensure it is clear and concise.
2. **Generate Multiple Agents**: Generate multiple agents to work in parallel on the `<prompt>`.

{ <prompt> }
{ <from-file> }

**Parallel Execution Management:**
Deploy multiple Sub Agents to generate agents in parallel for maximum efficiency and creative
diversity:

- Launch 3-5 Sub Agents simultaneously using Task tool
- Monitor agent progress and completion
- Collect and validate all completed agents
