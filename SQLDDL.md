

Universal Workflow for Generating a Detailed V3 to V4 Migration Plan

This workflow guides an LLM to first establish a baseline schema by analyzing the provided application JSON, and then generate a comprehensive migration plan for all system components.

Prerequisite:

A single, complete JSON file representing the legacy application's structure. For this workflow, we will refer to it as V3_System_Definition.json.

Phase 1: Schema and DDL Generation (Reverse Engineering)

Objective: Analyze the provided V3_System_Definition.json to identify all data entities, infer their schemas, and generate a complete SQL DDL. This DDL will represent the foundational data structure for the V4 system.

** chained-prompts/phase-1-schema-generation **


**➡️ Prompt 1.1: Identify Data Entities (Table Discovery)**

**System Instruction:** You are a data architect specializing in system analysis. Your task is to identify all distinct data entities by scanning for database table names within a JSON application definition.

**User Prompt:**
"Thoroughly analyze the provided `V3_System_Definition.json` file. Scan all `data` blocks (SQL queries) and `execute_query` actions within the `views` object. Extract and compile a unique list of all SQL table names mentioned in `FROM`, `JOIN`, `INSERT INTO`, and `UPDATE` statements. Present this list in a simple Markdown format."

---
 chained-output/1.1-entity-list.md 
---

**➡️ Prompt 1.2: Infer Schema for Each Entity**

**System Instruction:** You are a database schema analyst. Your task is to infer the columns and data types for each identified database table by examining how they are used in SQL queries.

**User Prompt:**
"Using the entity list from `1.1-entity-list.md` and the full `V3_System_Definition.json`, infer the schema for each table. For each table, analyze all `SELECT` clauses, `INSERT` column lists, `UPDATE SET` clauses, and `WHERE` conditions to identify every column.

For each column, determine a suitable SQL data type (e.g., `VARCHAR(255)`, `TEXT`, `INTEGER`, `BOOLEAN`, `TIMESTAMP WITH TIME ZONE`, `DECIMAL`, `UUID`). Create a separate Markdown table for each entity with the following columns: `Column Name` and `Inferred SQL Data Type`."

---
 chained-output/1.2-inferred-schemas.md 
---

**➡️ Prompt 1.3: Identify Primary and Foreign Key Relationships**

**System Instruction:** You are a relational database designer. Your task is to identify relationships between tables based on query logic and naming conventions.

**User Prompt:**
"Analyze the `JOIN ... ON` conditions, `WHERE` clause comparisons between columns of different tables, and common ID naming conventions (e.g., `ticket_id`, `unique_identifier`) within all SQL queries in `V3_System_Definition.json`. Based on this analysis, identify potential Primary Key (PK) and Foreign Key (FK) relationships.

Create a Markdown table with the following columns to document these relationships: `Table Name`, `Suggested Primary Key`, and `Foreign Key Relationships` (formatted as `ThisTable.ForeignKeyColumn -> OtherTable.PrimaryKeyColumn`)."

---
 chained-output/1.3-key-relationships.md 
---

**➡️ Prompt 1.4: Generate Final SQL DDL**

**System Instruction:** You are a senior database administrator. Your task is to generate a complete, well-structured, and production-ready SQL DDL script.

**User Prompt:**
"Using the inferred schemas from `1.2-inferred-schemas.md` and the key relationships from `1.3-key-relationships.md`, generate the final, complete SQL DDL script.

For each table:
1.  Use the `CREATE TABLE` statement.
2.  Define each column with its inferred data type.
3.  Add `PRIMARY KEY` constraints to the identified primary key columns.
4.  Add `FOREIGN KEY` constraints with `REFERENCES` clauses for all identified relationships.
5.  As a best practice, add `created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()` and `updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()` columns to every table.
6.  Format the output as a single, clean SQL code block."

---
 chained-output/1.4-final-ddl.sql 
---

**🔄 Feedback & Refinement Loop 1**

**User Prompt:**
"Review the generated DDL in `1.4-final-ddl.sql`. The relationship between `[Example_Table_A]` and `[Example_Table_B]` is missing an `ON DELETE SET NULL` clause. Please regenerate the DDL for `[Example_Table_A]` to include this clause on its foreign key constraint."
Phase 2: Generating the Component & Logic Migration Plan

Objective: Create detailed plans for migrating forms, workflows, and UI pages, using the generated DDL as the new data foundation.

** chained-prompts/phase-2-migration-plan **

code
Code
download
content_copy
expand_less
**➡️ Prompt 2.1: Generate V4 Form Migration Plan**

**System Instruction:** You are a senior software engineer planning a UI migration. You need to create clear, actionable instructions for a development team.

**User Prompt:**
"Using the task/question inventory from `2.1-task-question-inventory.md` and the complexity watchlist from `2.2-form-complexity-watchlist.md`, generate a detailed form migration plan. Assume V4 uses a modern, component-based form builder.

For each V3 task (form), provide:
- **V4 Form Name:** [V3 Task Name]
- **V4 Form Fields (in a table):**
  - `V3 Field Title`
  - `V3 Field Type`
  - `Recommended V4 Component` (e.g., TextInput, DatePicker, Select, LocationPicker, FieldGroup)
  - `Configuration Notes` (For dropdowns, specify the data source. For nested structures like SUBFORM, describe the required nested fields or recommend a reusable component strategy.)"

---
 chained-output/2.1-form-migration-plan.md 
---

**➡️ Prompt 2.2: Generate V4 Declarative Workflow Plan**

**System Instruction:** You are a workflow design expert. Your task is to translate a legacy process model into a modern, declarative trigger-and-action workflow definition.

**User Prompt:**
"Using the process diagrams (`3.1-process-flow-diagrams.md`) and the automation inventory (`3.2-automation-rules-inventory.md`), create a V4 migration plan for each process. Assume V4 workflows are defined by a trigger and a sequence of steps.

For each process, provide a plan in the following format:

**Process:** [Process Name]
- **Trigger:** [e.g., 'Manual Start by user', 'On Row Creation in `[table_name]`']
- **Steps:**
  - **1. [Action Type]:** [e.g., 'Show Form']
    - **Form:** [Form Name from `2.1-form-migration-plan.md`]
    - **Allocation:** [e.g., 'All assigned users']
  - **2. [Action Type]:** [e.g., 'On Form Submission (Automation)']
    - **Action:** [e.g., 'Create Row in Table `[table_name]`']
    - **Mappings:** [Reference mappings from the automation inventory.]
- **Conditional Logic:** [Describe any branching logic in plain English.]"

---
 chained-output/2.2-workflow-migration-plan.md 
---

**➡️ Prompt 2.3: Generate V4 Page Reconstruction Guide**

**System Instruction:** You are a senior front-end developer writing technical specifications for rebuilding a user interface.

**User Prompt:**
"Using the page inventory (`4.1-page-widget-inventory.md`) and the original `V3_System_Definition.json`, generate a reconstruction guide for each page.

For each page, provide:
1.  **Page Name:** [Page Title]
2.  **V4 Route:** [pageRoute from JSON]
3.  **V4 Layout:** Describe the grid layout based on the UI structure (e.g., 'Top section: Header card. Main section: two-column grid of summary cards. Bottom section: list of cards').
4.  **Widget Configuration Guide (Table):**
    - `Widget Type`
    - `Element ID`
    - `V4 Datasource Name` (from `data` block key)
    - `Key Configurations` (For lists, specify the `data` source. For interactive elements, describe the `onclick` action, including navigation targets and arguments.)"

---
 chained-output/2.3-page-reconstruction-guide.md 
---

**➡️ Prompt 2.4: Generate V4 Navigation Configuration Plan**

**System Instruction:** You are a lead developer configuring an application's main navigation shell and routing.

**User Prompt:**
"Based on the navigation summary from `5.1-navigation-summary.md`, create a simple, actionable plan for configuring the V4 navigation menu. Assume the V4 configuration requires a list of items, each with a label, icon, and route. Provide a Markdown list of navigation items to create and specify which page should be set as the default/home route."

---
 chained-output/2.4-v4-navigation-plan.md 
---

**🔄 Feedback & Refinement Loop 2**

**User Prompt:**
"Review the complete migration plan (`2.1` to `2.4`). The page reconstruction guide for `[Example_Page_Name]` does not fully explain the conditional visibility logic for the container with `element_id` `[Example_Element_ID]`. Refine the 'Key Configurations' for this widget to explicitly state the condition: `Visible when {{[datasource_name].data.[0].[column]}} == '[value]'`."
