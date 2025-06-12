# Multi-Agent Workflow Example

This example demonstrates how to use multiple Claude agents in parallel for larger projects.

## Scenario

You're building a web application with frontend, backend, database, and testing components. Using multiple agents allows parallel development of these components.

## Step 1: Create Comprehensive Design Document

```markdown
# E-Commerce Platform Design

## Overview
A full-stack e-commerce platform with user authentication, product catalog, shopping cart, and payment processing.

## Architecture
- Frontend: React with TypeScript
- Backend: Node.js with Express
- Database: PostgreSQL
- Authentication: JWT tokens
- Payment: Stripe integration

## Features
1. User Management
   - Registration and login
   - Profile management
   - Password reset
   
2. Product Catalog
   - Product listing with search
   - Categories and filters
   - Product details with images
   
3. Shopping Cart
   - Add/remove items
   - Update quantities
   - Save for later
   
4. Checkout Process
   - Address management
   - Payment processing
   - Order confirmation
   
5. Admin Panel
   - Product management
   - Order management
   - User management

## Non-Functional Requirements
- Response time < 200ms
- Support 1000 concurrent users
- 99.9% uptime
- Mobile responsive
```

Save as `ecommerce-design.md`.

## Step 2: Generate PRD and Initial Tasks

```bash
# Generate PRD
/user:generate-prd ecommerce-design.md

# Generate single-agent task list first
/user:generate-tasks
# Review and type "Go"
```

## Step 3: Distribute Tasks to Multiple Agents

```bash
# Generate multi-agent task distribution
/user:generate-multi-agent-tasks
```

This creates:
- `tasks/agent-1-tasks.md` - Frontend tasks
- `tasks/agent-2-tasks.md` - Backend tasks
- `tasks/agent-3-tasks.md` - Database tasks
- `tasks/agent-4-tasks.md` - Testing tasks
- `tasks/multi-agent-coordination.md` - Coordination info

## Step 4: Review Task Distribution

```bash
# Check task distribution
for i in {1..4}; do
    echo "=== Agent $i ==="
    head -20 tasks/agent-$i-tasks.md
    echo ""
done

# Review coordination file
cat tasks/multi-agent-coordination.md
```

## Step 5: Reorganize Tasks if Needed

If the distribution seems unbalanced:

```bash
# Move 3 tasks from agent 1 to agent 2
/user:reorganize-tasks from=1 to=2 count=3

# Move specific task types
/user:reorganize-tasks from=3 to=4 pattern="test"
```

## Step 6: Execute with Multiple Agents

```bash
# Launch multi-agent execution
/user:execute-multi-agent-tasks max-agents=4
```

This will:
1. Generate prompts for each agent
2. Display instructions for launching new Claude instances
3. Create context files for each agent
4. Update the coordination file

## Step 7: Launch Additional Claude Instances

For each agent (2-4), open a new terminal and:

```bash
# Agent 2
claude --no-conversation-history
# Paste the generated prompt for Agent 2

# Agent 3  
claude --no-conversation-history
# Paste the generated prompt for Agent 3

# Agent 4
claude --no-conversation-history
# Paste the generated prompt for Agent 4
```

## Step 8: Monitor Progress

In the main terminal:

```bash
# Check overall progress
/user:check-progress

# View specific agent status
grep -c "\[x\]" tasks/agent-*-tasks.md

# Check coordination status
tail -20 tasks/multi-agent-coordination.md
```

## Step 9: Handle Dependencies

When an agent is blocked:

```bash
# Agent 2 needs schema from Agent 3
# In Agent 3's terminal, ensure schema task is complete
# Then in Agent 2's terminal:
/user:continue-tasks 2
```

## Step 10: Reassign Tasks from Completed Agents

When an agent finishes early:

```bash
# If Agent 3 completes all tasks
/user:reorganize-tasks from=1 to=3 remaining=true

# Agent 3 can now help Agent 1
```

## Complete Multi-Agent Session

```bash
# Terminal 1 (Main/Agent 1)
mkdir ecommerce-platform
cd ecommerce-platform
git init

# Create design
cat > ecommerce-design.md << 'EOF'
[... design content ...]
EOF

# Generate PRD and tasks
/user:generate-prd ecommerce-design.md
/user:generate-tasks
# Type "Go"

# Distribute to agents
/user:generate-multi-agent-tasks

# Review distribution
cat tasks/multi-agent-coordination.md

# Start execution
/user:execute-multi-agent-tasks max-agents=4

# Terminal 2 (Agent 2)
cd ecommerce-platform
claude --no-conversation-history
# Paste Agent 2 prompt

# Terminal 3 (Agent 3)
cd ecommerce-platform
claude --no-conversation-history
# Paste Agent 3 prompt

# Terminal 4 (Agent 4)
cd ecommerce-platform
claude --no-conversation-history
# Paste Agent 4 prompt
```

## Coordination File Example

```markdown
# Multi-Agent Coordination

## Current Status - [timestamp]

### Agent 1 - Frontend Development
- Status: In Progress
- Current Task: Implementing product listing component
- Completed: 5/12 tasks
- Blocked: No

### Agent 2 - Backend Development
- Status: In Progress
- Current Task: Setting up Express routes
- Completed: 3/10 tasks
- Blocked: Waiting for database schema from Agent 3

### Agent 3 - Database & Infrastructure
- Status: In Progress
- Current Task: Creating PostgreSQL schema
- Completed: 4/8 tasks
- Blocked: No

### Agent 4 - Testing & Documentation
- Status: Waiting
- Current Task: Waiting for components to test
- Completed: 1/9 tasks
- Blocked: Needs components from Agents 1 and 2

## Dependencies
- Agent 2 waiting on Agent 3 for schema (Task 3.2)
- Agent 4 waiting on Agent 1 for components (Task 1.3)
- Agent 4 waiting on Agent 2 for API endpoints (Task 2.4)
```

## Tips for Multi-Agent Success

1. **Clear Boundaries**: Ensure agents have well-defined, non-overlapping responsibilities
2. **Communication**: Use the coordination file for status updates
3. **Dependencies**: Identify and communicate blockers early
4. **Rebalancing**: Reassign tasks as agents complete their work
5. **Context Management**: Monitor context usage per agent

## Common Multi-Agent Commands

| Command | Purpose |
|---------|---------|
| `/user:generate-multi-agent-tasks` | Distribute tasks to agents |
| `/user:reorganize-tasks from=1 to=2` | Move tasks between agents |
| `/user:execute-multi-agent-tasks` | Start parallel execution |
| `/user:check-progress` | Monitor all agents |
| `/user:continue-tasks 2` | Resume specific agent |

## Handling Common Issues

### Agent Blocked on Dependency
```bash
# Check dependency status
grep -A5 "Blocked:" tasks/multi-agent-coordination.md

# In the blocking agent's terminal
# Complete the required task, then update coordination file
```

### Context Overflow
```bash
# If an agent approaches context limit
/user:continue-tasks 2  # This will suggest spawning a sub-agent
```

### Merge Conflicts
```bash
# If agents modify the same file
git status  # Check conflicts
git merge --no-ff agent-2-branch
# Resolve conflicts manually
```

## Next Steps

- [Complex Project Management](complex-project.md) for enterprise-scale projects
- [Continuous Integration](ci-integration.md) for automated workflows
- [Performance Optimization](performance-tuning.md) for large teams