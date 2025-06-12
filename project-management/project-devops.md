# Project DevOps

## Description

This command will deploy multiple agents to work in parallel to review the project's README.md, codebase, and tests to make sure everything is in order. Then generate development environment setup instructions.

## Usage

```
/user:project-devops
```

## Process

1. Review the new README.md.
2. Generate multiple agents to work in parallel on the following tasks:

- Review the codebase and make sure we have all the functionality we say we do in the README.md
- That all the logic is sound and we don't have any stubbed functions
- Update/Create clear documentation for development env setup, and then you run through it and make sure it works.
- Review the amount of tests, and amount of passing tests.

**Parallel Execution Management:**
Deploy multiple Sub Agents to generate agents in parallel for maximum efficiency and creative
diversity:

- Launch 3-5 Sub Agents simultaneously using Task tool
- Monitor agent progress and completion
- Collect and validate all completed agents
