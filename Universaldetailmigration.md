

Universal V3 to V4 System Migration Workflow

This workflow synthesizes pre-analyzed legacy system data into a set of actionable, detailed migration plans for a modern V4 architecture.

Prerequisites:

Before starting, ensure you have the analysis documents generated from an initial inventory phase. This workflow assumes these files are available as context for the prompts:

1.1-datasource-inventory.md: A summary of all V3 datasources.

1.2-sql-analysis.md: A report on SQL syntax, patterns, and potential issues.

2.1-task-question-inventory.md: A mapping of V3 tasks (forms) to their questions (fields).

2.2-form-complexity-watchlist.md: A list of complex form fields requiring special attention.

3.1-process-flow-diagrams.md: Visual representations of all V3 business processes.

3.2-automation-rules-inventory.md: A catalog of all backend automation logic (reactions).

4.1-page-widget-inventory.md: An inventory of all UI pages and their widgets.

5.1-navigation-summary.md: A summary of the application's navigation structure.

Phase 1: Generating the Data & Foundation Layer Plan
Subtask 1: Generate the Datasource Migration Plan

Objective: Create the definitive plan for migrating each V3 datasource, including rewritten queries and strategies for handling deprecated types.

** chained-prompts/subtask-1-datasource-plan **

code
Code
download
content_copy
expand_less
**➡️ Prompt 1.1: Generate V4 Datasource Migration Strategy**

**System Instruction:** You are a cloud solutions architect specializing in data systems. Your task is to create a detailed and actionable migration plan for a development team based on pre-existing analysis.

**User Prompt:**
"Using the provided datasource inventory in `1.1-datasource-inventory.md` and the SQL analysis in `1.2-sql-analysis.md`, create a comprehensive migration plan. Adhere to the following V4 system constraints (these can be modified if your target system differs):
- All SQL syntax must be ANSI SQL compliant.
- Parameters are strongly typed; nulls must be handled explicitly (e.g., using `COALESCE` or `IS NULL`).
- Dynamic filtering patterns like `(column = {{param}} OR {{param}} = 'All')` must be rewritten using modern, portable syntax (e.g., `CASE` statements or handling the logic in the application layer).
- Legacy `SHEET` or file-based datasources are deprecated and must be migrated to a database table or a dedicated API endpoint.

For each datasource, create a new Markdown table with these columns: `V3 ID`, `V3 Label`, `V4 Migration Strategy`, `Rewritten V4 Query / Action`, and `Potential Issues & Notes`."

---
 chained-output/1.1-datasource-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 1**

**User Prompt:**
"Review the generated plan in `1.1-datasource-migration-plan.md`. The rewritten query for datasource `[Example_ID]` could be more performant. Refine the plan by rewriting its query to use a Common Table Expression (CTE) instead of a nested subquery and add a note about indexing the `[column_name]` column in the `Potential Issues & Notes` section."
Subtask 2: Generate the Form & Field Migration Plan

Objective: Create a detailed blueprint for reconstructing each V3 form using modern V4 components, including data bindings and complex logic handling.

** chained-prompts/subtask-2-form-plan **

code
Code
download
content_copy
expand_less
**➡️ Prompt 2.1: Generate V4 Form Migration Plan**

**System Instruction:** You are a senior software engineer writing technical specifications for a UI migration. Your goal is to provide clear, component-based instructions for a development team.

**User Prompt:**
"Using the task inventory (`2.1-task-question-inventory.md`), the complexity watchlist (`2.2-form-complexity-watchlist.md`), and the finalized datasource migration plan (`1.1-datasource-migration-plan.md`), generate a detailed form migration plan. Assume V4 uses a modern, component-based form builder.

For each V3 task (form), provide:
- **V4 Form Name:** [V3 Task Name]
- **V4 Form Fields (in a table):**
  - `V3 Field Title`
  - `V3 Field Type`
  - `Recommended V4 Component` (e.g., TextInput, DatePicker, Select, LocationPicker, FieldGroup, FileUpload)
  - `Configuration Notes` (For dropdowns, specify the **V4 Datasource ID** to link to. For nested structures like SUBFORM/SUBGROUP, describe the required nested fields or recommend creating a reusable component.)"

---
 chained-output/2.1-form-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 2**

**User Prompt:**
"Review the form migration plan in `2.1-form-migration-plan.md`. In the plan for the `[Example_Form_Name]` form, the nested `[Complex_Field_Type]` for `[Field_Title]` is used multiple times. Refine the plan to suggest creating a single reusable 'Address' component in the V4 system and reference it in the 'Recommended V4 Component' column for all relevant fields."
Phase 2: Generating the Business Logic Plan
Subtask 3: Generate the Workflow Migration Plan

Objective: Translate the legacy process flows and automations into modern, declarative V4 workflow definitions.

** chained-prompts/subtask-3-workflow-plan **

code
Code
download
content_copy
expand_less
**➡️ Prompt 3.1: Generate V4 Declarative Workflow Plan**

**System Instruction:** You are a workflow design expert. Your task is to translate an implicit, procedural process model into a modern, declarative trigger-and-action workflow definition.

**User Prompt:**
"Using the process diagrams (`3.1-process-flow-diagrams.md`) and the automation inventory (`3.2-automation-rules-inventory.md`), create a V4 migration plan for each process. Assume V4 workflows are defined by a trigger and a sequence of steps.

For each process, provide a plan in the following format:

**Process:** [Process Name]
- **Trigger:** [e.g., 'Manual Start by any user', 'On Row Creation in `sheet_name`']
- **Steps:**
  - **1. [Action Type]:** [e.g., 'Show Form']
    - **Form:** [Form Name from `2.1-form-migration-plan.md`]
    - **Allocation:** [e.g., 'All assigned users', 'User who triggered the workflow']
  - **2. [Action Type]:** [e.g., 'On Form Submission (Automation)']
    - **Action:** [e.g., 'Create Row in Table `[table_name]`']
    - **Mappings:** [Reference the mappings from the automation inventory file.]
  - **3. [Action Type]:** [e.g., 'Assign Task from Data']
    - **Source Data:** [e.g., 'Row from Step 2']
    - **Allocation Logic:** [e.g., 'Assign to user specified in the `assigned_technician` column']
    - **Task/Form:** [Form Name]
- **Conditional Logic:** [Describe any branching logic in plain English, e.g., 'If `Task_A` response for `question_X` is 'Yes', proceed to Step 4; otherwise, proceed to Step 5.']"

---
 chained-output/3.1-workflow-migration-plan.md 
---

**🔄 Feedback & Refinement Loop 3**

**User Prompt:**
"Review the workflow plan in `3.1-workflow-migration-plan.md`. In the `[Example_Process_Name]` workflow, the allocation logic for Step 3 is ambiguous. Refine this to specify the exact column (`[column_name]`) from the source data from which the user ID should be read for task assignment."
Phase 3: Generating the Presentation Layer Plan
Subtask 4: Generate the Page & Widget Reconstruction Guide

Objective: Create a detailed guide for developers to rebuild the V3 pages and widgets, ensuring they are correctly connected to the migrated V4 datasources.

** chained-prompts/subtask-4-page-plan **

code
Code
download
content_copy
expand_less
**➡️ Prompt 4.1: Generate V4 Widget-to-Datasource Mapping**

**System Instruction:** You are a data integration specialist. Your task is to create the definitive mapping guide for linking front-end components to their corresponding backend data sources.

**User Prompt:**
"Using the page inventory (`4.1-page-widget-inventory.md`) and the datasource migration plan (`1.1-datasource-migration-plan.md`), create a master mapping table. This table will serve as the single source of truth for connecting V4 widgets to data. The columns should be: `Page Title`, `Widget ID`, `Widget Type`, `V3 DataSource ID`, and `V4 DataSource ID`."

---
 chained-output/4.1-widget-datasource-map.md 
---

**➡️ Prompt 4.2: Generate V4 Page Reconstruction Guide**

**System Instruction:** You are a senior front-end developer writing technical specifications for rebuilding a user interface. Your instructions must be precise and easy for other developers to follow.

**User Prompt:**
"Using the widget-datasource map (`4.1-widget-datasource-map.md`) and the original page inventory (`4.1-page-widget-inventory.md`), generate a reconstruction guide for each V3 page.

For each page, provide:
1.  **Page Name:** [Page Title]
2.  **V4 Route:** [pageRoute]
3.  **V4 Layout:** Describe the grid layout based on the `layout` array (e.g., 'Top row: 3-column DatePicker, 3-column SearchBar. Second row: 12-column Table').
4.  **Widget Configuration Guide (Table):**
    - `Widget Type`
    - `Label`
    - `V4 DataSource ID` (from `4.1-widget-datasource-map.md`)
    - `Key Configurations` (Extract and list critical settings from the V3 JSON. For tables, list the `columns` array. For maps, detail the `markers` array. For filters, specify `targetKey` and any other relevant keys)."

---
 chained-output/4.2-page-reconstruction-guide.md 
---

**🔄 Feedback & Refinement Loop 4**

**User Prompt:**
"Review the guide in `4.2-page-reconstruction-guide.md`. The `Key Configurations` for the table widget `[Example_Widget_ID]` on the `[Example_Page_Name]` page are too generic. Refine the plan to include a detailed list of all columns from the V3 widget's configuration, including their name, type, and format."
Subtask 5: Generate the Navigation Configuration Plan

Objective: Create the final configuration plan for the V4 application shell, including the navigation menu and default routing.

** chained-prompts/subtask-5-navigation-plan **

code
Code
download
content_copy
expand_less
**➡️ Prompt 5.1: Generate V4 Navigation Configuration Plan**

**System Instruction:** You are a lead developer configuring an application's main navigation shell and routing.

**User Prompt:**
"Based on the navigation summary from `5.1-navigation-summary.md`, create a simple, actionable plan for configuring the V4 navigation menu. Assume the V4 configuration requires a list of items, each with a label, icon, and route.

Provide a Markdown list of navigation items to create, using the titles and routes from the V3 page definitions. Explicitly state which page should be set as the default/home route."

---
 chained-output/5.1-v4-navigation-plan.md 
---

**🔄 Feedback & Refinement Loop 5**

**User Prompt:**
"Review the navigation plan in `5.1-v4-navigation-plan.md`. The icons are missing. Re-examine the V3 page and navigation configuration files, extract the icon name for each page (e.g., from a `meta.IconSetting.name` field), and add an 'Icon Name' column to the navigation plan table."
