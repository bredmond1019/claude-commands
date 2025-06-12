##########################
####### MULTI AGENTS #####
##########################

### Step 1 - Split Tasks

- Make a new document called tasks/parallel-tasks-list.md.
- Review all of the remaining tasks in @tasks/tasks-prd-climbr-mvp-phase1
- Reorganize all of the tasks into three separate lists
- Each list is for an agent running in parallel completing tasks without interfering with the other agent

  - Intent: Create a structured document to organize parallel work

### Step 2 - Generate Agent Prompts

Now generate a small and concise prompt for each Agent 1, Agent 2, and Agent 3 so they can understand exactly what they need to do given this new task list.

Use the following propmt structure for the agent prompts:

<agent-prompt-structure>

You are Agent <X> responsible for <your-focus-area>. Complete these <X> tasks from @tasks/parallel-tasks-list.md. Use @tasks/ai-dev-tasks/process-task-list.mdc

Docs to Reference:

- `Project Design Doc` : @tasks/project-design-doc.md
- `Original Task List` : @tasks/tasks-prd-climbr-mvp-phase1.md
- `Parallel Task List` : @tasks/parallel-tasks-list.md

**Tasks:**

**Your focus areas:**

**Key requirements:**

**Dependencies:**

**Success criteria:**

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them on both the `Original Task List` & `Parallel Tasks List`

</agent-prompt-structure>

### Step 3 - Initialize Agent on Tasks

#############################################
####### AGENT INITIAL INSTRUCTIONS ##########
#############################################

Take each prompt, and feed it to a separate instance of claude with approving all permissions.

##########################################
############# CONTINUE TASK ##############
##########################################

### Step 4 - Continue Tasks Prompts | Multiple Agents

<continue-prompt>
- Go back and check off all completed tasks by Agent <X> thus far.
- Then continue onto the next task for Agent B on @tasks/parallel-tasks-list.md
- After each subtask is completed, git commit for only the files you've modifies/created
- Then mark the subtask complete on the tasks file.
</continue-prompt>

### Compact Chat History

/compact provide only enough context to understand the main idea of the project and what is needed to complete the next few tasks, and which agent you are.

### When one agent finishes all their tasks

- Review remaining tasks for Agent B @tasks/parallel-tasks-list.md
- continue onto the next task for Agent B that won't interfere with Agent B.
- Move that Item to Agent B's list
- Then start on that task
- After each subtask is completed, git commit for only the files you've modifies/created
- Then Repeat the process

### PYTHON SERVICE

- First Review @tasks/parallel-task-list
- Then, Let's make a few updates to the application as follows

  - Now that you know what the task list file looks like, update our application to better fit that format for keeping track of tasks completed.

  - The application should start and focus on just a single instance of claude. I will start up three separate terminals with the application running for each agent, instead of one application running three instances of Claude. Here is a sample initial prompt:
    <initial-prompt>
    You are Agent 1 responsible for Search & Discovery Features. Complete these 5 tasks from @tasks/parallel-tasks-list.md. Use @tasks/ai-dev-tasks/process-task-list.mdc

    Docs to Reference:

    - `Project Design Doc` : @tasks/product-design-doc.md
    - `Original Task List` : @tasks/tasks-prd-climbr-mvp-phase1.md
    - `Parallel Task List` : @tasks/parallel-tasks-list.md

    **Tasks:**

    - 3.1 Create search filters for location, skill level, and preferences
    - 3.2 Implement geospatial queries using PostGIS
    - 3.3 Build advanced search with multiple filter combinations
    - 3.4 Add search result pagination (20 results per page)
    - 3.5 Implement saved search preferences functionality

    **Your focus areas:**

    - Partner search functionality in backend/matching/
    - Geospatial queries with PostGIS
    - Search optimization and filtering

    **Key requirements:**

    - Use descriptive class/method names (e.g., LocationRadiusFilter, find_climbers_within_radius)
    - Implement clear pagination with has_next_page, total_results_count
    - Create SavedSearch model with clear field names

    **Dependencies:**

    - Django models and PostGIS already configured
    - User and ClimbingProfile models exist
    - No dependencies on other agents

    **Success criteria:**

    - All search filters work with clear parameter names
    - Geospatial queries use proper PostGIS methods
    - Pagination returns exactly 20 results per page
    - Users can save and load search preferences

    For each task:

    - Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
    - Commit after each completed subtask with only the files you've touched
    - Mark tasks complete with [x] as you finish them on both the `Original Task List` & `Parallel Tasks 
  List`
      </initial-prompt>

  - Everytime Claude finishes and returns a response, we should restart claude code with the prompt of:
    <continue-prompt>
    - Go back and check off all completed tasks by Agent 2 thus far.
    - Then continue onto the next task for Agent 2 on @tasks/parallel-tasks-list.md
    - After each subtask is completed, git commit for only the files you've modifies/created
    - Then mark the subtask complete on the tasks file.
      </continue-prompt>

<initial-prompt>
  You are Agent 1 responsible for Search & Discovery Features. Complete these 5 tasks from @tasks/parallel-tasks-list.md. Use @tasks/ai-dev-tasks/process-task-list.mdc

Docs to Reference:

- `Project Design Doc` : @tasks/product-design-doc.md
- `Original Task List` : @tasks/tasks-prd-climbr-mvp-phase1.md
- `Parallel Task List` : @tasks/parallel-tasks-list.md

**Tasks:**

- 3.1 Create search filters for location, skill level, and preferences
- 3.2 Implement geospatial queries using PostGIS
- 3.3 Build advanced search with multiple filter combinations
- 3.4 Add search result pagination (20 results per page)
- 3.5 Implement saved search preferences functionality

**Your focus areas:**

- Partner search functionality in backend/matching/
- Geospatial queries with PostGIS
- Search optimization and filtering

**Key requirements:**

- Use descriptive class/method names (e.g., LocationRadiusFilter, find_climbers_within_radius)
- Implement clear pagination with has_next_page, total_results_count
- Create SavedSearch model with clear field names

**Dependencies:**

- Django models and PostGIS already configured
- User and ClimbingProfile models exist
- No dependencies on other agents

**Success criteria:**

- All search filters work with clear parameter names
- Geospatial queries use proper PostGIS methods
- Pagination returns exactly 20 results per page
- Users can save and load search preferences

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them on both the `Original Task List` & `Parallel Tasks 
  List`
  </initial-prompt>

⏺ Agent 2 Prompt: AI & Matching System

You are Agent 2 responsible for AI & Matching System. Complete these 8 tasks from
@tasks/parallel-tasks-list.md. Use @tasks/ai-dev-tasks/process-task-list.mdc

Docs to Reference:

- `Project Design Doc` : @tasks/product-design-doc.md
- `Original Task List` : @tasks/tasks-prd-climbr-mvp-phase1.md
- `Parallel Task List` : @tasks/parallel-tasks-list.md

**Tasks:**

- 4.1 Integrate OpenAI API for natural language search processing
- 4.2 Develop compatibility scoring algorithm considering grades, styles, location
- 4.3 Implement daily partner suggestion system (3-5 matches)
- 4.4 Create feedback system for AI suggestions (thumbs up/down)
- 4.5 Build learning mechanism to improve suggestions from feedback
- 4.6 Add transparent explanation for why partners are suggested
- 4.7 Implement fallback to traditional search if AI fails
- 4.8 Create Celery tasks for async AI processing

**Your focus areas:**

- AI integration in backend/ai_services/
- Compatibility scoring algorithms
- Matching suggestion system
- Asynchronous task processing with Celery

**Key requirements:**

- Create NaturalLanguageSearchParser with descriptive methods
- Build CompatibilityScorer with clear weight constants (LOCATION_WEIGHT, SKILL_MATCH_WEIGHT)
- Generate 3-5 daily partner suggestions
- Implement thumbs up/down feedback with learning mechanism

**Dependencies:**

- User and ClimbingProfile models exist
- Celery and Redis already configured
- No dependencies on other agents' work

**Success criteria:**

- OpenAI API processes natural language queries
- Compatibility scores are transparent and explainable
- Daily suggestions are personalized and improving
- Fallback to traditional search works seamlessly

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them on both the `Original Task List` & `Parallel Tasks
List`

⏺ Agent 3 Prompt: Messaging & Frontend

You are Agent 3 responsible for Messaging & Frontend. Complete these 17 tasks from
@tasks/parallel-tasks-list.md. Use @tasks/ai-dev-tasks/process-task-list.mdc

Docs to Reference:

- `Project Design Doc` : @tasks/product-design-doc.md
- `Original Task List` : @tasks/tasks-prd-climbr-mvp-phase1.md
- `Parallel Task List` : @tasks/parallel-tasks-list.md

**Tasks:**

- 5.1-5.6: Complete messaging system (DirectMessage model, real-time messaging, notifications,
  content filtering, blocking)
- 6.1-6.10: Build entire frontend (React setup, Tailwind CSS, auth components, profile UI, search UI,
  messaging UI, responsive design)

**Your focus areas:**

- Direct messaging system in backend/messaging/
- Real-time WebSocket communication with Django Channels
- Complete React frontend with TypeScript
- GraphQL integration with Apollo Client

**Key requirements:**

- Create DirectMessage model with clear methods (mark_as_read, is_read_by_recipient)
- Implement WebSocket events ('message.new', 'message.read', 'user.typing')
- Build responsive React components with Tailwind CSS
- Use descriptive component names (LoginForm, PartnerSuggestionCard)

**Dependencies:**

- Django Channels already configured
- User and ClimbingProfile models exist
- Coordinate with Agent 1 for search UI integration

**Success criteria:**

- Real-time messaging works with read receipts
- Frontend is fully responsive (mobile, tablet, desktop)
- All components have proper loading states and error handling
- GraphQL types are properly generated

For each task:

- Follow the process in @tasks/ai-dev-tasks/process-task-list.mdc
- Commit after each completed subtask with only the files you've touched
- Mark tasks complete with [x] as you finish them on both the `Original Task List` & `Parallel Tasks 
List`
