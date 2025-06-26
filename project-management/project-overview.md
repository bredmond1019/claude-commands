# /user:project-overview [--prompt <prompt>]

## Description

This command creates a short and concise overview of the work that has been completed so far, the most recent work, and what are a few appropriate next steps with the overall project.

The overview includes:

- Summary of completed work
- Most recent work
- Appropriate next steps
- Open questions section
- Potential use cases and future ideas

## Arguments

- `--prompt <prompt>`: Any additional prompt to add to the context.

## Usage

```
/user:project-overview --prompt "Some additional info or instructions"
```

## Process

Create `tasks/project-overview.md` with a short and concise overview containing the following sections:

1. **Work Completed So Far** - Summary of what has been accomplished
2. **Most Recent Work** - Latest developments and progress
3. **Possible Next Steps** - A few possible next actions for the project
4. **Open Questions** - Unresolved issues and questions
5. **Potential Use Cases / Future Ideas** - How this project could be leveraged

## Project Overview File Structure

```md
# Project Overview

Project: <project-name>
Created: <date> <time>
Last Updated: <date> <time>

## Work Completed So Far

- <summary of work completed so far>
- <summary of work completed so far>
- <summary of work completed so far>

## Most Recent Work

- <summary of most recent work>
- <summary of most recent work>

## Possible Next Steps

- <list of possible next steps>
- <list of possible next steps>

## Open Questions

- <list of open questions>
- <list of open questions>

## Potential Use Cases / Future Ideas

- <list of ideas of potential use cases / future ideas>
- <list of ideas of potential use cases / future ideas>
```
