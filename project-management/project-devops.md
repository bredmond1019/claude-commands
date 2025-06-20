# /user:project-devops [--prompt <prompt>]

## Description

This command will deploy multiple agents to work in parallel to review the project's README.md, codebase, and tests to make sure everything is in order. Then generate development environment setup instructions.

## Arguments

- `--prompt <prompt>`: Any additional prompt to add to the agents.

## Usage

```
/user:project-devops --prompt "Add a new feature to the project"
```

## Process

1. Review the new README.md.
2. Generate multiple agents to work in parallel on the following tasks:

- Review the codebase and make sure we have all the functionality we say we do in the README.md
- Make sure the README.md reflects the current state of the project and what's coming soon in future phases.
- It has a QUICK_START.md file that should be updated to reflect the current state of the project.
- Make sure the DEV_SETUP.md file is updated to reflect the current state of the project.
- Make sure the project-open-source.md file is updated/created to reflect the current state of the project and remaining tasks until open source.
- Make sure the unfinished-tasks.md file is updated/created to reflect the current state of the project.
  - That all the logic is sound and we don't have any stubbed functions.
- Update/Create clear documentation for development env setup, and then you run through it and make sure it works.
- Review the amount of tests, and amount of passing tests.
- If no README.md, create one after going through the codebase.

**Parallel Execution Management:**
Deploy multiple Sub Agents to generate agents in parallel for maximum efficiency and creative
diversity:

- Launch 3-5 Sub Agents simultaneously using Task tool
- Monitor agent progress and completion
- Collect and validate all completed agents
