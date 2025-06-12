# Task Distribution Algorithm

Utility for distributing tasks across multiple agents for parallel execution.

## Core Algorithm

```python
class TaskDistributor:
    def __init__(self, task_list, agent_count=None):
        self.tasks = self.parse_task_list(task_list)
        self.agent_count = agent_count or self.auto_detect_agents()
        
    def distribute(self):
        # Step 1: Analyze task characteristics
        task_analysis = self.analyze_tasks()
        
        # Step 2: Group tasks by functional area
        functional_groups = self.group_by_function(task_analysis)
        
        # Step 3: Calculate complexity scores
        complexity_scores = self.calculate_complexity()
        
        # Step 4: Create agent roles based on groups
        agent_roles = self.define_agent_roles(functional_groups)
        
        # Step 5: Balance workload across agents
        assignments = self.balance_workload(complexity_scores, agent_roles)
        
        # Step 6: Identify dependencies
        dependencies = self.identify_dependencies(assignments)
        
        # Step 7: Create execution phases
        phases = self.create_execution_phases(assignments, dependencies)
        
        return {
            'assignments': assignments,
            'dependencies': dependencies,
            'phases': phases,
            'agent_roles': agent_roles
        }
```

## Task Analysis Criteria

### Functional Areas
- **Frontend**: UI components, user interactions, styling, client-side logic
- **Backend**: API endpoints, server logic, authentication, authorization
- **Data**: Database schema, models, migrations, data access layer
- **Integration**: External services, third-party APIs, webhooks
- **Infrastructure**: Deployment, CI/CD, monitoring, configuration
- **Testing**: Unit tests, integration tests, E2E tests, test utilities
- **Documentation**: API docs, user guides, code comments, examples

### Complexity Scoring

```python
def calculate_task_complexity(task):
    score = 1.0  # Base score
    
    # Adjust based on subtask count
    subtask_count = len(task.subtasks)
    if subtask_count > 5:
        score *= 1.5
    elif subtask_count > 3:
        score *= 1.2
    
    # Adjust based on file count
    file_count = len(task.relevant_files)
    if file_count > 5:
        score *= 1.3
    elif file_count > 2:
        score *= 1.1
    
    # Adjust based on keywords
    complex_keywords = ['integration', 'migration', 'refactor', 'architecture']
    for keyword in complex_keywords:
        if keyword in task.title.lower():
            score *= 1.2
    
    return score
```

## Workload Balancing

```python
def balance_workload(tasks, agents):
    # Initialize agent workloads
    agent_workloads = {agent: 0 for agent in agents}
    agent_tasks = {agent: [] for agent in agents}
    
    # Sort tasks by complexity (descending)
    sorted_tasks = sorted(tasks, key=lambda t: t.complexity, reverse=True)
    
    # Assign tasks to least loaded agent that matches functional area
    for task in sorted_tasks:
        suitable_agents = find_suitable_agents(task, agents)
        if suitable_agents:
            # Pick agent with lowest current workload
            selected_agent = min(suitable_agents, key=lambda a: agent_workloads[a])
            agent_tasks[selected_agent].append(task)
            agent_workloads[selected_agent] += task.complexity
    
    return agent_tasks
```

## Dependency Detection

```python
def identify_dependencies(assignments):
    dependencies = []
    
    for agent1, tasks1 in assignments.items():
        for agent2, tasks2 in assignments.items():
            if agent1 != agent2:
                # Check for file dependencies
                files1 = get_all_files(tasks1)
                files2 = get_all_files(tasks2)
                
                # Check if agent2 depends on files created by agent1
                for file in files2:
                    if file in files1 and is_output_file(file, tasks1):
                        dependencies.append({
                            'from': agent1,
                            'to': agent2,
                            'file': file,
                            'type': 'file_dependency'
                        })
                
                # Check for explicit task dependencies
                for task2 in tasks2:
                    for task1 in tasks1:
                        if mentions_task(task2, task1):
                            dependencies.append({
                                'from': agent1,
                                'to': agent2,
                                'task_from': task1.id,
                                'task_to': task2.id,
                                'type': 'task_dependency'
                            })
    
    return dependencies
```

## Execution Phases

```python
def create_execution_phases(assignments, dependencies):
    phases = []
    remaining_tasks = flatten_all_tasks(assignments)
    completed_tasks = set()
    
    while remaining_tasks:
        # Find tasks with no pending dependencies
        ready_tasks = []
        for task in remaining_tasks:
            task_deps = get_task_dependencies(task, dependencies)
            if all(dep in completed_tasks for dep in task_deps):
                ready_tasks.append(task)
        
        if not ready_tasks:
            # Circular dependency detected
            raise Exception("Circular dependency detected")
        
        # Group ready tasks by agent for this phase
        phase_agents = {}
        for task in ready_tasks:
            agent = find_task_agent(task, assignments)
            if agent not in phase_agents:
                phase_agents[agent] = []
            phase_agents[agent].append(task)
        
        phases.append({
            'phase_number': len(phases) + 1,
            'agents': phase_agents,
            'parallel_execution': True
        })
        
        # Mark tasks as completed
        completed_tasks.update(ready_tasks)
        remaining_tasks = [t for t in remaining_tasks if t not in ready_tasks]
    
    return phases
```

## Agent Count Auto-Detection

```python
def auto_detect_agent_count(tasks):
    total_tasks = len(tasks)
    total_complexity = sum(task.complexity for task in tasks)
    functional_areas = count_functional_areas(tasks)
    
    # Rules for agent count
    if total_tasks < 10 or total_complexity < 15:
        return 3  # Simple project
    elif total_tasks < 20 or total_complexity < 30:
        return 4  # Medium project
    else:
        return min(5, functional_areas)  # Complex project, cap at 5
```

## Usage Example

```python
# Parse task list
task_list = parse_markdown_file('tasks/tasks-list.md')

# Create distributor
distributor = TaskDistributor(task_list, agent_count=4)

# Distribute tasks
distribution = distributor.distribute()

# Generate agent files
for agent_id, agent_data in distribution['assignments'].items():
    generate_agent_file(agent_id, agent_data, distribution)

# Generate coordination file
generate_coordination_file(distribution)
```