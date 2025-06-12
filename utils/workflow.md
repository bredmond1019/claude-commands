### Initial Planning Prompt

Use @create-prd.mdc
Here's the feature I want to build: [Describe your feature in detail]
Reference these files to help you: [Optional: @file1.py @file2.ts]

### Generate Tasks Prompt

Now take @MyFeature-PRD.md and create tasks using @generate-tasks-from-prd.mdc

### Process Tasks Prompt

Please start on task 1.1 and use @process-task-list.mdc


##########################
####### MULTI AGENTS #####
##########################

### Split Tasks

- Make a new document called tasks/parallel_tasks-final.md.
- Review all of the tasks in 6.0, 7.0, and 8.0
- Reorganize all of the tasks into two separate lists
- Each list is for an agent running in parallel completing tasks without interfering with the other agent

  - Intent: Create a structured document to organize parallel work

### Generate Agent Prompts

Now generate a small and concise prompt for each Agent A and Agent B so they can understand exactly what they need to do given this new task list.

### Initialize Agent on Tasks

    ##########################
    ####### AGENT A #####
    ##########################

You are Agent A working on the AI System project. Your task is to complete all items in the "Agent A Task List" section of @tasks/parallel-tasks-final.md. Use @tasks/ai-dev-tasks/process-task-list.mdc

Docs to Reference:

- `Original Task List` : @tasks/tasks-project-design-doc.md
- `Parallel Task List` : @tasks/parallel-tasks-final.md

You are responsible for backend infrastructure and deployment. Your focus areas are:

1. **Model Management** - Integrate all AI model providers (OpenAI, Anthropic, Bedrock, Ollama) with consistent interfaces, cost tracking, and fallback mechanisms
2. **Backend Services** - Build the API gateway, agent monitoring, execution viewer, and user management systems
3. **Testing Infrastructure** - Create comprehensive test scenarios, conduct integration/load/security testing
4. **Deployment** - Set up Kubernetes manifests, CI/CD pipeline, monitoring alerts, and production deployment

Work in these directories: `/src/core/model_management/`, `/src/api/`, `/src/monitoring/`, `/kubernetes/`, `/tests/`

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them on both the `Original Task List` & `Parallel Tasks List`

Start with task 6.1 Complete model provider integration with consistent interfaces

    ##########################
    ####### AGENT B #####
    ##########################

You are Agent B working on the AI System project. Your task is to complete all items in the "Agent B Task List" section of @tasks/parallel-tasks-final.md. Use @tasks/ai-dev-tasks/process-task-list.mdc

Docs to Reference:

- `Original Task List` : @tasks/tasks-project-design-doc.md
- `Parallel Task List` : @tasks/parallel-tasks-final.md

You are responsible for frontend UI and documentation. Your focus areas are:

1. **Testing Tools** - Build benchmarking suite, A/B testing visualization, cost prediction tools, and workflow templates
2. **UI Development** - Create the monitoring dashboard, workflow visualizer, debugging interface, analytics charts, and agent configuration UI
3. **Documentation** - Write user guides, migration docs, SDK documentation, troubleshooting guides, and architecture diagrams
4. **Examples** - Create example workflows, runbooks, and interactive API documentation

Work in these directories: `/frontend/`, `/docs/`, `/examples/`, `/scripts/`

Design the UI to consume Agent A's API endpoints. Create documentation that references actual implemented features. All UI components should have clear information hierarchy and helpful tooltips.

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them on both the `Original Task List` & `Parallel Tasks List`

Start with task 6.6 Create benchmarking suite with interpretable metrics

##########################################
####### MULTI AGENTS | CONTINUE TASK #####
##########################################

### Continue Tasks Prompts | Multiple Agents

- Go back and check off all completed tasks by Agent B thus far.
- Then continue onto the next task for Agent B on @tasks/parallel-tasks-final.md
- After each subtask is completed, git commit for only the files you've modifies/created
- Then mark the subtask complete on the tasks file.

### Compact Chat History

/compact provide only enough context to understand the main idea of the project and what is needed to complete the next few tasks, and which agent you are.

### When one agent finishes all their tasks

- Review remaining tasks for Agent B @tasks/parallel-tasks-final.md
- continue onto the next task for Agent B that won't interfere with Agent B.
- Move that Item to Agent B's list
- Then start on that task
- After each subtask is completed, git commit for only the files you've modifies/created
- Then Repeat the process
