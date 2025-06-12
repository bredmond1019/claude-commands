# Multi-Agent Workflow Example

This example demonstrates how to use the updated multi-agent commands for parallel development of larger projects.

## Scenario

You're building a web application with frontend, backend, database, and testing components. Using multiple agents allows parallel development of these components with automatic coordination.

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
# Generate PRD from design document
/user:prd:generate ecommerce-design.md

# Generate comprehensive task list from PRD
/user:tasks:generate tasks/project-prd.md
```

## Step 3: Distribute Tasks to Multiple Agents

```bash
# Automatically distribute tasks across 4 agents
/user:tasks:generate-multi-agent tasks/tasks-list.md --agents 4
```

This creates:
- `tasks/agent-1-tasks.md` - Frontend development tasks
- `tasks/agent-2-tasks.md` - Backend API tasks
- `tasks/agent-3-tasks.md` - Database and infrastructure tasks
- `tasks/agent-4-tasks.md` - Testing and DevOps tasks
- `tasks/multi-agent-coordination.md` - Central coordination file

## Step 4: Review Task Distribution

```bash
# Check task distribution for each agent
for i in {1..4}; do
    echo "=== Agent $i Tasks ==="
    head -20 tasks/agent-$i-tasks.md
    echo ""
done

# Review coordination and dependencies
cat tasks/multi-agent-coordination.md
```

## Step 5: Execute with Multiple Agents

```bash
# Launch all agents in parallel
/user:tasks:execute-multi-agent
```

This automatically:
1. Analyzes task dependencies
2. Generates specialized prompts for each agent
3. Creates coordination tracking
4. Deploys all agents simultaneously
5. Monitors progress and context usage

Alternative execution options:

```bash
# Deploy specific agents only
/user:tasks:execute-multi-agent --agents 1,3 --mode parallel

# Custom deployment for specialized scenarios
/user:deploy:multiple-agents "Build e-commerce platform components" --from-file ecommerce-design.md
```

## Step 6: Monitor Progress

```bash
# Check real-time progress of all agents
cat tasks/multi-agent-coordination.md

# Monitor specific agent progress
cat tasks/agent-1-tasks.md | grep -c "\[x\]"

# Watch progress continuously
watch cat tasks/multi-agent-coordination.md
```

## Step 7: Handle Interruptions and Dependencies

If an agent stops due to context limits or dependencies:

```bash
# Continue a specific agent
/user:tasks:continue 2

# Continue all stopped agents
/user:tasks:continue-parallel

# Reorganize tasks if needed
/user:tasks:reorganize tasks/agent-1-tasks.md tasks/agent-3-tasks.md
```

## Step 8: Complete the Project

```bash
# Mark all agents' work as complete
/user:tasks-mark-complete

# Commit all changes
/user:tasks-commit

# Generate next iteration if needed
/user:prd:iterate-project --mode next-phase
```

## Complete Workflow Example

```bash
# Set up project
mkdir ecommerce-platform
cd ecommerce-platform
git init

# Create design document
cat > ecommerce-design.md << 'EOF'
# E-Commerce Platform Design
[... design content ...]
EOF

# Execute complete multi-agent workflow
/user:prd:generate ecommerce-design.md
/user:tasks:generate
/user:tasks:generate-multi-agent tasks/tasks-list.md --agents 4
/user:tasks:execute-multi-agent

# Monitor in another terminal
watch cat tasks/multi-agent-coordination.md

# Handle any interruptions
/user:tasks:continue-parallel

# Complete the project
/user:tasks-mark-complete
/user:tasks-commit
```

## Coordination File Example

The system automatically maintains a coordination file like this:

```markdown
# Multi-Agent Task Coordination

## Agent Status - 2024-01-15 14:30:00
- Agent 1: Active (Frontend) - 8/12 tasks complete (65%), 45% context used
- Agent 2: Active (Backend) - 6/10 tasks complete (60%), 52% context used  
- Agent 3: Waiting (Database) - 7/8 tasks complete (87%), 78% context used
- Agent 4: Active (Testing) - 3/9 tasks complete (33%), 25% context used

## Task Dependencies
- Agent 2 waiting for Agent 3 to complete database migrations
- Agent 4 waiting for Agent 1 to complete component library
- No other blockers

## Recent Handoffs
- [x] Agent 3 → Agent 2: Database schema completed
- [x] Agent 1 → Agent 4: Core components ready for testing
- [ ] Agent 2 → Agent 4: API endpoints for integration tests

## Progress Summary
Total Tasks: 39
Completed: 24 (62%)
In Progress: 8
Blocked: 2
Remaining: 5

## Next Actions
- Agent 3 approaching context limit - may need continuation
- Agent 4 can proceed with component testing
- Agent 2 ready to continue once migration completes
```

## Advanced Multi-Agent Features

### Project DevOps Review
Deploy specialized agents to review your entire project:

```bash
/user:project-devops
```

Creates agents to:
- Review and improve README
- Analyze codebase architecture  
- Review and enhance tests
- Generate development environment setup

### Custom Agent Deployment
For specialized scenarios:

```bash
/user:deploy:multiple-agents "Optimize performance across all microservices" \
  --from-file architecture.md \
  --coordination-file tasks/optimization-coordination.md
```

### Parallel Task Deployment
Quick deployment for specific work:

```bash
/user:deploy:parallel-tasks "Add real-time features to the platform" \
  --from-file realtime-requirements.md
```

## Best Practices for Multi-Agent Success

### 1. Optimal Agent Distribution
- **3 agents**: Small to medium projects
- **4 agents**: Most full-stack applications (recommended)
- **5 agents**: Complex projects with multiple domains

### 2. Task Organization
- Let the system analyze and distribute tasks automatically
- The AI understands dependencies better than manual distribution
- Each agent gets cohesive, related tasks

### 3. Monitoring and Coordination
- Check coordination file regularly for status updates
- Don't interrupt agents unless they've stopped
- Use continuation commands when agents pause

### 4. Dependency Management
- The system automatically handles most dependencies
- Agents know when to wait for prerequisites
- Handoffs are tracked in the coordination file

## Troubleshooting Common Issues

### Agent Context Overflow
```bash
# System automatically pauses agents at 80% context
/user:tasks:continue 2  # Continue specific agent
/user:tasks:continue-parallel  # Continue all stopped agents
```

### Unbalanced Task Distribution
```bash
# Move tasks between agents
/user:tasks:reorganize tasks/agent-1-tasks.md tasks/agent-3-tasks.md
```

### Only One Agent Running
```bash
# Correct parallel execution state
/user:parallel-persist
```

### Checking Agent Progress
```bash
# List all agent task files
ls tasks/agent-*-tasks.md

# Check completion counts
grep -c "\[x\]" tasks/agent-*-tasks.md

# Review coordination status
tail -20 tasks/multi-agent-coordination.md
```

## Updated Command Reference

| Command | Purpose |
|---------|---------|
| `/user:prd:generate` | Generate PRD from design document |
| `/user:tasks:generate` | Create comprehensive task list |
| `/user:tasks:generate-multi-agent` | Distribute tasks to agents |
| `/user:tasks:execute-multi-agent` | Deploy all agents in parallel |
| `/user:tasks:continue` | Resume specific agent |
| `/user:tasks:continue-parallel` | Resume all stopped agents |
| `/user:tasks:reorganize` | Rebalance tasks between agents |
| `/user:deploy:multiple-agents` | Custom agent deployment |
| `/user:deploy:parallel-tasks` | Quick parallel deployment |
| `/user:project-devops` | Deploy DevOps review agents |
| `/user:tasks-mark-complete` | Mark all work complete |
| `/user:tasks-commit` | Commit all changes |
| `/user:prd:iterate-project` | Generate next iteration |

## Key Improvements Over Manual Approach

1. **Automated Coordination**: No manual prompt generation or agent management
2. **Smart Distribution**: AI analyzes tasks and creates optimal groupings
3. **Dependency Tracking**: Built-in understanding of task prerequisites
4. **Progress Monitoring**: Real-time tracking of all agents
5. **Easy Continuation**: Simple commands to resume interrupted work
6. **Conflict Prevention**: Agents automatically avoid file conflicts
7. **Context Management**: Automatic pausing and resumption at context limits

## Next Steps

After completing your multi-agent project:

- Use `/user:prd:iterate-project` to plan the next development phase
- Deploy `/user:project-devops` to review and optimize the entire project
- Consider the workflow for your next project using lessons learned

The multi-agent system transforms development from sequential task execution to coordinated parallel development, dramatically improving both speed and quality.