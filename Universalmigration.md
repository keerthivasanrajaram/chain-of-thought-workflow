

Universal V3 to V4 System Migration Workflow

This workflow is designed to be executed by a Large Language Model (LLM) to analyze legacy system components and generate a comprehensive, actionable migration plan.

Prerequisites:

Before starting, gather the following components of your V3 system and rename them as follows for consistency throughout this workflow:

Datasources Definition: v3_datasources.json

Tasks/Form Definitions: v3_tasks.json

Questions/Field Definitions: v3_questions.json

Process/Workflow Logic: v3_processes.json

UI Pages & Widgets: v3_pages.json

Navigation/Application Shell: v3_navigation.json

Phase 1: Data & Foundation Layer Migration

This phase establishes the new data foundation by analyzing and planning the migration of datasources and the data-capture structures (forms).

Subtask 1: Datasource Migration

Objective: Analyze all V3 datasources, identify dependencies and breaking changes, and create a migration plan for a robust, V4-compatible data access layer.

** chained-prompts/subtask-1-datasources **

code
Code
download
content_copy
expand_less
**➡️ Prompt 1.1: Datasource Inventory Analysis**

**System Instruction:** You are a system migration analyst. Your task is to perform an initial inventory of all data sources from the provided V3 JSON file.

**User Prompt:**
"Analyze the provided `v3_datasources.json` file. Create a Markdown table that summarizes each datasource. The columns should be: `ID`, `Label`, `Type` (e.g., QUERY, SHEET), `Primary Source` (e.g., Sheet ID, Table Name), and `Parameters Used` (list all parameter keys from the configuration, noting their default values)."

---
 chained-output/1.1-datasource-inventory.md 
---

**➡️ Prompt 1.2: Query Syntax & Pattern Recognition**

**System Instruction:** You are a database migration specialist. Your task is to analyze SQL queries for non-standard syntax and common patterns that may require special handling in a new system.

**User Prompt:**
"Using the full content of `v3_datasources.json` and the inventory from `chained-output/1.1-datasource-inventory.md`, perform a detailed analysis of each `QUERY` type datasource. Identify and list the following:
1.  **Platform-Specific SQL:** List any datasources using non-standard syntax (e.g., `::date` for casting, platform-specific date functions, proprietary array operators like `ANY()`).
2.  **Dynamic Filtering Patterns:** Note how parameters are used, especially common patterns for optional filters like `... WHERE (column = {{param}} OR {{param}} = 'All')`.
3.  **UI-Specific Patterns:** Identify queries that follow a `SELECT 'All' UNION ALL SELECT DISTINCT ...` pattern, typically used for populating filter dropdowns."

---
 chained-output/1.2-sql-analysis.md 
---

**➡️ Prompt 1.3: Generate V4 Datasource Migration Strategy**

**System Instruction:** You are a cloud solutions architect specializing in data systems. You have knowledge of both V3 and V4 architectures. Your task is to create a detailed, actionable migration plan.

**User Prompt:**
"Based on the datasource inventory (`1.1-datasource-inventory.md`) and the SQL analysis (`1.2-sql-analysis.md`), create a comprehensive migration plan. Assume the V4 system has the following constraints, which you can modify if needed:
- SQL syntax must be ANSI SQL compliant.
- Parameters are strongly typed and nulls must be handled explicitly (e.g., using `COALESCE` or `IS NULL`).
- Legacy `SHEET` type datasources must be migrated to a database table or a dedicated API endpoint.

For each datasource, create a new Markdown table with these columns: `V3 ID`, `V3 Label`, `V4 Migration Strategy`, `Rewritten V4 Query / Action`, and `Potential Issues & Notes`."

---
 chained-output/1.3-datasource-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 1**

**User Prompt:**
"Review the generated plan in `1.3-datasource-migration-plan.md`.
- Are the rewritten SQL queries valid and efficient?
- Does the strategy for `SHEET` datasources make sense?
- For dynamic filtering patterns, is the proposed solution robust?

Provide a refinement prompt for a specific datasource, for example: 'Refine the query for datasource ID `[Example_ID]` (`[Example_Label]`) to improve performance by replacing the subquery with a Common Table Expression (CTE).'"
Subtask 2: Form & Field Migration

Objective: Convert V3 task and question definitions into a structured plan for creating modern, component-based V4 forms.

** chained-prompts/subtask-2-forms **

code
Code
download
content_copy
expand_less
**➡️ Prompt 2.1: Task and Question Inventory**

**System Instruction:** You are a data model analyst. Your task is to map the relationships between high-level tasks (forms) and the questions (fields) they contain.

**User Prompt:**
"Analyze `v3_questions.json` and `v3_tasks.json`.
1.  First, create a comprehensive dictionary mapping every `questionId` to an object containing its `title` and `type` from `v3_questions.json`.
2.  Then, iterate through each task in `v3_tasks.json`. For each task, create a summary in a Markdown list. Include the `taskId`, `taskName`, and a nested list of the `title` and `type` for each question it contains, using the dictionary you created."

---
 chained-output/2.1-task-question-inventory.md 
---

**➡️ Prompt 2.2: Identify Complex Form Logic & Components**

**System Instruction:** You are a UI/UX component specialist. Your task is to identify complex UI components and logic that will require custom implementation or careful mapping.

**User Prompt:**
"Using the full `v3_questions.json` file, identify all questions with complex configurations that will need special attention during migration. Create a 'Form Complexity Watchlist' table with columns for `Question ID`, `Question Title`, `Type`, and `Complexity Description`. Pay special attention to:
- Nested structures like `SUBGROUP` and `SUBFORM`.
- `DROPDOWN` fields that fetch data from legacy sources (`fetchFrom: 'SHEET'`).
- Any conditional visibility or logic defined in a `settings.conditions` array."

---
 chained-output/2.2-form-complexity-watchlist.md 
---

**➡️ Prompt 2.3: Generate V4 Form Migration Plan**

**System Instruction:** You are a senior software engineer planning a UI migration. You need to create clear, actionable instructions for a development team.

**User Prompt:**
"Using the task inventory (`2.1-task-question-inventory.md`), the complexity watchlist (`2.2-form-complexity-watchlist.md`), and the datasource migration plan (`1.3-datasource-migration-plan.md`), generate a detailed migration plan for each task. Assume V4 has a modern, component-based form builder. For each V3 task, provide:
- **V4 Form Name:** [v3 taskName]
- **V4 Form Fields (in a table):**
  - `V3 Question Title`
  - `V3 Question Type`
  - `Recommended V4 Component` (e.g., TextInput, DatePicker, Select, LocationPicker, FieldGroup, FileUpload)
  - `Configuration Notes` (For dropdowns, specify the **V4 Datasource** to link to. For nested structures, describe the nesting or suggest a reusable component strategy.)"

---
 chained-output/2.3-form-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 2**

**User Prompt:**
"Review the form migration plan in `2.3-form-migration-plan.md`. For a task with complex nested logic (e.g., 'Add Customer'), provide a refinement prompt like: 'In the plan for the `[Example_Task_Name]` task, the `[Complex_Field_Type]` for `[Field_Title]` is used multiple times. Refine the plan to suggest creating a single reusable component in the V4 system for this logic.'"
Phase 2: Business Logic & Automation Migration

This phase translates the V3 process flows, which combine user tasks with backend automations, into a modern, declarative workflow structure.

Subtask 3: Process & Workflow Migration

Objective: Analyze the V3 processes, deconstruct their flow and automation logic, and create a migration plan for a V4-compatible workflow engine.

** chained-prompts/subtask-3-processes **

code
Code
download
content_copy
expand_less
**➡️ Prompt 3.1: Visualize Process Flows**

**System Instruction:** You are a business process analyst. Your task is to visualize complex workflows from a JSON definition to make them easily understandable.

**User Prompt:**
"Analyze the `draftState` of each process in `v3_processes.json`. For each process, generate a MermaidJS graph definition (`graph TD`) to visualize the flow.
- Represent nodes for `start`, `task`, `label`, `form`, and `reaction`.
- Use the node's `id` and `data.label` for clarity.
- Connect the nodes based on the `links` array.
- For links connecting to a `taskNode` or `labelNode`, add the allocation type (e.g., 'All Users', 'From Previous Sheet') as text on the link."

---
 chained-output/3.1-process-flow-diagrams.md 
---

**➡️ Prompt 3.2: Extract Automation (Reaction) Logic**

**System Instruction:** You are an automation specialist. Your task is to extract and document all backend automation rules from a process definition.

**User Prompt:**
"Examine all `reactionNode` definitions within `v3_processes.json`. Create a master table of all automations with the following columns: `Process Name`, `Trigger Node (Task/Form ID)`, `Reaction Type` (e.g., CREATE_ROW, UPDATE_LABEL, WEB_HOOK), `Target Sheet/Endpoint`, and `Field Mappings Summary` (briefly describe what is being mapped, e.g., 'Maps 5 question fields and 2 task metadata fields to the target sheet')."

---
 chained-output/3.2-automation-rules-inventory.md 
---

**➡️ Prompt 3.3: Generate V4 Declarative Workflow Plan**

**System Instruction:** You are a workflow design expert. Your task is to translate a legacy process model into a modern, declarative trigger-and-action workflow definition.

**User Prompt:**
"Using the process diagrams (`3.1-process-flow-diagrams.md`) and the automation inventory (`3.2-automation-rules-inventory.md`), create a V4 migration plan for each process. Assume V4 workflows are defined by a trigger and a sequence of steps. For each process, provide a plan in the following format:

**Process:** [Process Name]
- **Trigger:** [e.g., 'Manual Start by any user']
- **Steps:**
  - **1. [Action Type]:** [e.g., 'Show Form']
    - **Form:** [Form Name from `2.3-form-migration-plan.md`]
    - **Allocation:** [e.g., 'All assigned users']
  - **2. [Action Type]:** [e.g., 'On Form Submission (Automation)']
    - **Action:** [e.g., 'Create Row in Sheet `[sheet_name]`']
    - **Mappings:** [Reference the mappings from the automation inventory]
  - **3. [Action Type]:** [e.g., 'Show Task from Label']
    - **Source Label:** [e.g., 'Customer Orders']
    - **Allocation:** [e.g., 'User from `assigned_technician` column']
    - **Task:** [Task Name]
- **Conditional Logic:** [Describe any branching logic, e.g., 'If `Task_A` response for `question_X` is 'Yes', proceed to Step 4; otherwise, proceed to Step 5.']"

---
 chained-output/3.3-workflow-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 3**

**User Prompt:**
"Review the workflow plan in `3.3-workflow-migration-plan.md`. The plan for `[Example_Process_Name]` shows several tasks branching from a single start point. Clarify whether these tasks should be allocated sequentially or in parallel."
Phase 3: Presentation Layer Migration

This final phase handles the user-facing components: the pages, widgets, and overall application navigation.

Subtask 4: Page & Widget Migration

Objective: Plan the reconstruction of V3 pages and their widgets within a V4 UI framework, ensuring they connect to the newly planned V4 datasources.

** chained-prompts/subtask-4-pages **

code
Code
download
content_copy
expand_less
**➡️ Prompt 4.1: Page and Widget Inventory**

**System Instruction:** You are a UI/UX analyst. Your task is to inventory all pages and their constituent components from a JSON definition.

**User Prompt:**
"Analyze `v3_pages.json`. Create a hierarchical list of all pages. For each page, provide:
- **Page Title:**
- **Page ID:**
- **Widgets:**
  - A table with columns: `Widget ID`, `Widget Type`, `Label`, `V3 DataSource ID`."

---
 chained-output/4.1-page-widget-inventory.md 
---

**➡️ Prompt 4.2: Generate V4 Widget-to-Datasource Mapping**

**System Instruction:** You are a data integration specialist. Your task is to create a definitive mapping guide for linking front-end components to their corresponding backend data sources.

**User Prompt:**
"Using the page inventory (`4.1-page-widget-inventory.md`) and the datasource migration plan (`1.3-datasource-migration-plan.md`), create a master mapping table. The table should have columns: `Page Title`, `Widget ID`, `Widget Type`, `V3 DataSource ID`, and `V4 DataSource ID`. This will be the guide for connecting v4 widgets to their data."

---
 chained-output/4.2-widget-datasource-map.md 
---

**➡️ Prompt 4.3: Generate V4 Page Reconstruction Guide**

**System Instruction:** You are a front-end developer writing technical specifications for rebuilding a user interface.

**User Prompt:**
"Using all previous analysis, generate a reconstruction guide for each page in `v3_pages.json`. For each page, provide:
1.  **Page Name:** [Page Title]
2.  **V4 Route:** [pageRoute]
3.  **V4 Layout:** Describe the grid layout based on the `layout` array (e.g., 'Top row: 3-column DatePicker, 3-column SearchBar. Second row: 12-column Table').
4.  **Widget Configuration Guide (Table):**
    - `Widget Type`
    - `Label`
    - `V4 DataSource ID` (from `4.2-widget-datasource-map.md`)
    - `Key Configurations` (For tables, list columns. For maps, list marker settings like `locationColId`. For filters like `DATE_RANGE_PICKER`, specify `targetKey` and `toDateKey`)."

---
 chained-output/4.3-page-reconstruction-guide.md 
---

**🔄 Feedback & Refinement Loop 4**

**User Prompt:**
"Review the guide in `4.3-page-reconstruction-guide.md`. The configuration for `[Example_Widget_ID]` on the `[Example_Page_Name]` page is missing important details. Update the 'Key Configurations' to explicitly include [e.g., the `markers` array details: `locationColId`, `primaryColumn`, `markerStyle`, etc.]."
Subtask 5: Application Shell & Navigation Migration

Objective: Recreate the main application shell, including routing and the primary navigation menu.

** chained-prompts/subtask-5-navigation **

code
Code
download
content_copy
expand_less
**➡️ Prompt 5.1: Navigation Structure Analysis**

**System Instruction:** You are an information architect. Your task is to document the navigation structure of an application from its configuration file.

**User Prompt:**
"Analyze `v3_navigation.json` (or the equivalent file defining the main application shell).
1.  Identify the main application container and its base `route`.
2.  List the navigation links defined in the configuration (e.g., in `meta.navBar.links`). Use any reference maps provided to find the descriptive title for each navigation item ID.
3.  Identify the `defaultPage` for the application.
4.  Summarize this information in a clear, concise list."

---
 chained-output/5.1-navigation-summary.md 
---

**➡️ Prompt 5.2: V4 Navigation Configuration Plan**

**System Instruction:** You are a developer configuring an application's navigation shell.

**User Prompt:**
"Based on the navigation summary from `5.1-navigation-summary.md`, create a simple, actionable plan for configuring the V4 navigation menu. Assume the V4 configuration requires a list of items, each with a label, icon, and route. Provide a Markdown list of navigation items to create and specify which page should be set as the default/home route. Use the page titles from `v3_pages.json` for the labels."

---
 chained-output/5.2-v4-navigation-plan.md 
---

**🔄 Feedback & Refinement Loop 5**

**User Prompt:**
"Review the navigation plan. Are all links from the V3 application accounted for? Is the default page correctly identified? Confirm if the plan is complete or requires adjustments."
