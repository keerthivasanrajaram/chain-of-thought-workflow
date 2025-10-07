

This workflow breaks down the migration into three primary phases, starting from the data layer and moving up to the presentation layer. Each phase consists of subtasks with sequential prompts designed to build upon each other.

Phase 1: Data & Foundation Layer Migration

This phase focuses on understanding and planning the migration of the core data structures: datasources, questions, and tasks (which define the application's forms).

Subtask 1: Datasource Migration

Objective: Analyze all v3 datasources (pneumocare-fp_datasource.json), identify dependencies and potential breaking changes, and create a migration plan for a v4-compatible format.

** chained-prompts/subtask-1-datasources **

**➡️ Prompt 1.1: Datasource Inventory Analysis**

**System Instruction:** You are a system migration analyst. Your task is to perform an initial inventory of all data sources from the provided v3 JSON file.

**User Prompt:**
"Analyze the provided `pneumocare-fp_datasource.json` file. Create a Markdown table that summarizes each datasource. The columns should be: `ID`, `Label`, `Type` (QUERY/SHEET), `Sheet ID` (if applicable), and `Parameters Used` (list all keys from the `querySettings.mappings` array, noting their default values)."

---
 chained-output/1.1-datasource-inventory.md 
---

**➡️ Prompt 1.2: SQL Syntax & Pattern Recognition**

**System Instruction:** You are a database migration specialist. Your task is to analyze SQL queries for non-standard syntax and common patterns that may require special handling in a new system.

**User Prompt:**
"Using the full content of `pneumocare-fp_datasource.json` and the inventory from `chained-output/1.1-datasource-inventory.md`, perform a detailed analysis of each `QUERY` type datasource. Identify and list the following:
1.  **Non-Standard SQL:** List any datasources using platform-specific syntax like `::date`, `AT TIME ZONE`, or array operators like `ANY(teams)`.
2.  **Parameter Handling:** Note how parameters (e.g., `{{Username}}`) are used, especially in `WHERE` clauses (e.g., `... OR {{Username}} = 'All'`). This pattern often needs to be handled differently in v4.
3.  **Common Patterns:** Identify datasources that follow a `SELECT 'All' UNION ALL SELECT DISTINCT ...` pattern, as these are typically used for populating filter dropdowns."

---
 chained-output/1.2-sql-analysis.md 
---

**➡️ Prompt 1.3: Generate V4 Migration Strategy**

**System Instruction:** You are a cloud solutions architect specializing in data systems. You have knowledge of both v3 and v4 architectures. Your task is to create a detailed, actionable migration plan.

**User Prompt:**
"Based on the datasource inventory (`1.1-datasource-inventory.md`) and the SQL analysis (`1.2-sql-analysis.md`), create a comprehensive migration plan. Assume the following v4 constraints:
- SQL syntax must be ANSI SQL compliant.
- Parameters are strongly typed and nulls must be handled explicitly (e.g., using `COALESCE` or `IS NULL`). The `'All'` pattern should be handled with a `CASE` statement or logic outside the query if possible.
- `SHEET` type datasources are deprecated and must be migrated to either a database table or a dedicated API endpoint.

For each datasource, create a new Markdown table with these columns: `V3 ID`, `V3 Label`, `V4 Migration Strategy`, `Rewritten V4 Query / Action`, and `Potential Issues & Notes`."

---
 chained-output/1.3-datasource-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 1**

**User Prompt:**
"Review the generated plan in `1.3-datasource-migration-plan.md`.
- Are the rewritten SQL queries valid and efficient?
- Does the strategy for `SHEET` datasources make sense?
- For the `'All'` parameter pattern, is the proposed solution robust?

Refine the query for datasource ID `7` (`Attendance`) to ensure it correctly handles cases where a user might exist but has no activity in the selected date range. The query should still return the user's name with zero counts."

*(The model would then regenerate `1.3-datasource-migration-plan.md` with the refined query.)*
Subtask 2: Task & Question (Form) Migration

Objective: Convert v3 task definitions and their associated questions (pneumocare-tasks.json, pneumocare-questions.json) into a structured plan for creating v4 forms.

** chained-prompts/subtask-2-forms **

**➡️ Prompt 2.1: Task and Question Inventory**

**System Instruction:** You are a data model analyst. Your task is to map relationships between tasks and the questions they contain.

**User Prompt:**
"Analyze `pneumocare-questions.json` and `pneumocare-tasks.json`.
1.  First, create a comprehensive dictionary mapping every `questionId` to an object containing its `title` and `type` from `pneumocare-questions.json`.
2.  Then, iterate through each task in `pneumocare-tasks.json`. For each task, create a summary in a Markdown list. Include the `taskId`, `taskName`, and a nested list of the `title` and `type` for each question it contains, using the dictionary you created."

---
 chained-output/2.1-task-question-inventory.md 
---

**➡️ Prompt 2.2: Identify Complex Form Logic**

**System Instruction:** You are a front-end component specialist. Your task is to identify complex UI components and logic that will require custom implementation or careful mapping.

**User Prompt:**
"Using the full `pneumocare-questions.json` file, identify all questions with complex configurations that will need special attention during migration. Create a 'Form Complexity Watchlist' table with columns for `Question ID`, `Question Title`, `Type`, and `Complexity Description`. Pay special attention to:
- `SUBGROUP` and `SUBFORM` types.
- `DROPDOWN` fields where `fetchFrom` is `'SHEET'`.
- Any conditional logic defined in a `settings.conditions` array."

---
 chained-output/2.2-form-complexity-watchlist.md 
---

**➡️ Prompt 2.3: Generate V4 Form Migration Plan**

**System Instruction:** You are a senior software engineer planning a UI migration. You need to create clear, actionable instructions for a development team.

**User Prompt:**
"Using the task inventory (`2.1-task-question-inventory.md`), the complexity watchlist (`2.2-form-complexity-watchlist.md`), and the datasource migration plan (`1.3-datasource-migration-plan.md`), generate a detailed migration plan for each task. Assume v4 has a modern form builder. For each v3 task, provide:
- **V4 Form Name:** [v3 taskName]
- **V4 Form Fields (in a table):**
  - `V3 Question Title`
  - `V3 Question Type`
  - `Recommended V4 Component` (e.g., TextInput, DatePicker, Select, LocationPicker, FieldGroup)
  - `Configuration Notes` (For dropdowns, specify the **V4 Datasource** to link to. For SUBFORMs/SUBGROUPs, describe the nested structure.)"

---
 chained-output/2.3-form-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 2**

**User Prompt:**
"Review the form migration plan in `2.3-form-migration-plan.md`. In the plan for the 'Add Hospital' task, the `SUBGROUP` for 'Critical Care Department' is nested. Refine the plan to suggest creating a reusable 'Department Details' component in the v4 system for this and similar questions in the task."

*(The model regenerates `2.3-form-migration-plan.md` with the refined component strategy.)*
Phase 2: Business Logic Migration

This phase translates the v3 process flows, which combine user tasks with backend automations, into a modern, declarative workflow structure.

Subtask 3: Process & Workflow Migration

Objective: Analyze the v3 processes (pneumocare-process.json), deconstruct their flow and automation logic, and create a migration plan for a v4-compatible workflow engine.

** chained-prompts/subtask-3-processes **

**➡️ Prompt 3.1: Visualize Process Flows**

**System Instruction:** You are a business process analyst. Your task is to visualize complex workflows from a JSON definition.

**User Prompt:**
"Analyze the `draftState` of each process in `pneumocare-process.json`. For each process, generate a MermaidJS graph definition (`graph TD`) to visualize the flow.
- Represent nodes for `start`, `task`, `label`, and `reaction`.
- Use the node's `id` and `data.label` for clarity.
- Connect the nodes based on the `links` array.
- For links pointing to a `taskNode` or `labelNode`, add the allocation type (e.g., 'All Users', 'From Previous Sheet') as text on the link."

---
 chained-output/3.1-process-flow-diagrams.md 
---

**➡️ Prompt 3.2: Extract Automation (Reaction) Logic**

**System Instruction:** You are an automation specialist. Your task is to extract and document all backend automation rules from a process definition.

**User Prompt:**
"Examine all `reactionNode` definitions within `pneumocare-process.json`. Create a master table of all automations with the following columns: `Process Name`, `Trigger Node (Task/Form ID)`, `Reaction Type` (e.g., CREATE_ROW, UPDATE_LABEL), `Target Sheet ID`, and `Field Mappings Summary` (briefly describe what is being mapped, e.g., 'Maps 5 question fields and 2 task metadata fields')."

---
 chained-output/3.2-automation-rules-inventory.md 
---

**➡️ Prompt 3.3: V4 Declarative Workflow Plan**

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
    - **Action:** [e.g., 'Create Row in Sheet `enquiry_form`']
    - **Mappings:** [Reference the mappings from the automation inventory]
  - **3. [Action Type]:** [e.g., 'Show Task from Label']
    - **Source Label:** [e.g., 'Enquiry Form']
    - **Allocation:** [e.g., 'User from `assigned_person` column']
    - **Task:** [Task Name]
- **Conditional Logic:** [Describe any branching logic, e.g., 'If `Update hospital` task response is 'Yes', proceed to...']"

---
 chained-output/3.3-workflow-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 3**

**User Prompt:**
"Review the workflow plan in `3.3-workflow-migration-plan.md`. The 'Hospital' process (`process-95d5da3a-cb70-4a3e-b98b-a13902e4129f`) has multiple tasks branching from 'Check In'. The plan should explicitly state that these tasks ('Training', 'Visit Details', etc.) should be made available in parallel to the user after check-in."

*(The model updates the plan to clarify the parallel task allocation.)*
Phase 3: Presentation Layer Migration

This final phase handles the user-facing components: the pages, widgets, and overall application navigation.

Subtask 4: Page & Widget Migration

Objective: Plan the reconstruction of v3 pages (pneumocare-fp_page.json) and their widgets within a v4 UI framework, ensuring they connect to the newly planned v4 datasources.

** chained-prompts/subtask-4-pages **

**➡️ Prompt 4.1: Page and Widget Inventory**

**System Instruction:** You are a UI/UX analyst. Your task is to inventory all pages and their constituent components from a JSON definition.

**User Prompt:**
"Analyze `pneumocare-fp_page.json`. Create a hierarchical list of all pages. For each page, provide:
- **Page Title:**
- **Page ID:**
- **Widgets:**
  - A table with columns: `Widget ID`, `Widget Type`, `Label`, `V3 DataSource ID`."

---
 chained-output/4.1-page-widget-inventory.md 
---

**➡️ Prompt 4.2: V4 Widget-Datasource Mapping**

**System Instruction:** You are a data integration specialist. Your task is to link front-end components to their corresponding backend data sources.

**User Prompt:**
"Using the page inventory (`4.1-page-widget-inventory.md`) and the datasource migration plan (`1.3-datasource-migration-plan.md`), create a master mapping table. The table should have columns: `Page Title`, `Widget ID`, `Widget Type`, `V3 DataSource ID`, and `V4 DataSource ID`. This will be the definitive guide for connecting v4 widgets to their data."

---
 chained-output/4.2-widget-datasource-map.md 
---

**➡️ Prompt 4.3: V4 Page Reconstruction Guide**

**System Instruction:** You are a front-end developer writing technical specifications for rebuilding a UI.

**User Prompt:**
"Using the complete analysis so far, generate a reconstruction guide for each page in `pneumocare-fp_page.json`. For each page, provide:
1.  **Page Name:** [Page Title]
2.  **V4 Route:** [pageRoute]
3.  **V4 Layout:** Describe the grid layout based on the `layout` array (e.g., 'Top row: 3-column DatePicker, 3-column SearchBar. Second row: 12-column Table').
4.  **Widget Configuration Guide (Table):**
    - `Widget Type`
    - `Label`
    - `V4 DataSource ID` (from `4.2-widget-datasource-map.md`)
    - `Key Configurations` (For tables, list the columns. For maps, list the marker settings like `locationColId`. For filters like `DATE_RANGE_PICKER`, specify `targetKey` and `toDateKey`)."

---
 chained-output/4.3-page-reconstruction-guide.md 
---

**🔄 Feedback & Refinement Loop 4**

**User Prompt:**
"Review the guide in `4.3-page-reconstruction-guide.md`. The configuration for the map widget `MAP_CONTAINER-itemJEwhTaCQ` on the 'Distributor Details' page is incomplete. Update the 'Key Configurations' to explicitly include the `markers` array details: `locationColId`, `primaryColumn`, `markerStyle`, and `listSettings`."

*(The model regenerates the guide with more detailed widget configuration instructions.)*
Subtask 5: Proxy & Navigation Migration

Objective: Recreate the main application shell, including routing and the primary navigation menu.

** chained-prompts/subtask-5-navigation **

**➡️ Prompt 5.1: Navigation Structure Analysis**

**System Instruction:** You are an information architect. Your task is to document the navigation structure of an application.

**User Prompt:**
"Analyze `pneumocare-fp_proxy.json`.
1.  Identify the main proxy and its `route`.
2.  List the navigation links from `meta.navBar.links`. Use the `meta.referanceMap` to find the title for each page ID.
3.  Identify the `defaultPage` for the proxy.
4.  Summarize this information in a clear, concise list."

---
 chained-output/5.1-navigation-summary.md 
---

**➡️ Prompt 5.2: V4 Navigation Configuration Plan**

**System Instruction:** You are a developer configuring an application's navigation shell.

**User Prompt:**
"Based on the navigation summary from `5.1-navigation-summary.md`, create a simple, actionable plan for configuring the v4 navigation menu. Assume the v4 configuration requires a list of items with a label, icon, and route. Provide a Markdown list of navigation items to create and specify which page should be the default route. Use the page titles from `pneumocare-fp_page.json` for labels."

---
 chained-output/5.2-v4-navigation-plan.md 
---

**🔄 Feedback & Refinement Loop 5**

**User Prompt:**
"Review the navigation plan. Are all links from the v3 proxy accounted for? Is the default page correctly identified? The plan looks correct and complete."

*(No further refinement needed for this step.)*

This structured workflow ensures that every component of the v3 system is analyzed, mapped to its v4 equivalent, and has a clear, actionable migration plan. The chained nature of the prompts ensures context is carried through each step, leading to a cohesive and comprehensive final plan.
