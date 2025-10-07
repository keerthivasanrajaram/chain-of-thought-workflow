

Universal Workflow: Schema Design, DDL Generation, and Data Population from an Application Definition

This workflow is broken into two primary phases. The first phase focuses on analyzing the application definition to design and create the database schema. The second phase focuses on creating a plan to populate that schema with realistic data.

Prerequisite:

A single, complete JSON file representing the application's structure. For this workflow, it will be referred to as Application_Definition.json.

Phase 1: Schema Discovery and DDL Generation

Objective: Analyze the provided Application_Definition.json to identify all data entities, infer their schemas, identify relationships, and generate a complete SQL DDL script for creating the database.

** chained-prompts/phase-1-schema-generation **


**➡️ Prompt 1.1: Identify Data Entities (Table Discovery)**

**System Instruction:** You are a data architect specializing in system analysis. Your task is to identify all distinct data entities by scanning for database table names within a JSON application definition.

**User Prompt:**
"Thoroughly analyze the provided `Application_Definition.json` file. Scan all SQL queries found within `data` blocks, `execute_query` actions, `events` properties (like in calendars), and `search_query` properties. Extract and compile a unique list of all SQL table names mentioned in `FROM`, `JOIN`, `INSERT INTO`, and `UPDATE` statements. Present this list in a simple Markdown format."

---
 chained-output/1.1-entity-list.md 
---

**➡️ Prompt 1.2: Infer Schema for Each Entity (Columns & Data Types)**

**System Instruction:** You are a database schema analyst. Your task is to infer the columns and data types for each identified database table by examining how they are used in SQL queries and UI components throughout the application definition.

**User Prompt:**
"Using the entity list from `1.1-entity-list.md` and the full `Application_Definition.json`, infer the schema for each table. For each table, analyze all `SELECT` clauses, `INSERT` column lists, `UPDATE SET` clauses, and `WHERE` conditions to identify every column.

For each column, determine a suitable SQL data type (e.g., `VARCHAR(255)`, `TEXT`, `INTEGER`, `BOOLEAN`, `TIMESTAMP WITH TIME ZONE`, `DECIMAL`, `UUID`). Also, look at `input` UI components to help determine types (e.g., `input_type: "date"` suggests a `DATE` or `TIMESTAMP` type). Create a separate Markdown table for each entity with the following columns: `Column Name` and `Inferred SQL Data Type`."

---
 chained-output/1.2-inferred-schemas.md 
---

**➡️ Prompt 1.3: Identify Primary and Foreign Key Relationships**

**System Instruction:** You are a relational database designer. Your task is to identify relationships between tables based on query logic and common naming conventions.

**User Prompt:**
"Analyze the `JOIN ... ON` conditions, `WHERE` clause comparisons between columns of different tables (e.g., `ti.unique_identifier = ci.unique_identifier`), and common ID naming conventions (e.g., `ticket_id`, `unique_identifier`) within all SQL queries in `Application_Definition.json`. Based on this analysis, identify potential Primary Key (PK) and Foreign Key (FK) relationships.

Create a Markdown table with the following columns to document these relationships: `Table Name`, `Suggested Primary Key`, and `Foreign Key Relationships` (formatted as `ThisTable.ForeignKeyColumn -> OtherTable.PrimaryKeyColumn`)."

---
 chained-output/1.3-key-relationships.md 
---

**➡️ Prompt 1.4: Generate Final SQL DDL Script**

**System Instruction:** You are a senior database administrator. Your task is to generate a complete, well-structured, and production-ready SQL DDL script that can be used to create the database schema.

**User Prompt:**
"Using the inferred schemas from `1.2-inferred-schemas.md` and the key relationships from `1.3-key-relationships.md`, generate the final, complete SQL DDL script.

For each table:
1.  Use the `CREATE TABLE` statement.
2.  Define each column with its inferred data type. Add a `NOT NULL` constraint if the application logic implies it is a required field.
3.  Add `PRIMARY KEY` constraints to the identified primary key columns.
4.  Add `FOREIGN KEY` constraints with `REFERENCES` clauses for all identified relationships.
5.  As a best practice, add `created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()` and `updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()` columns to every table that appears to store transactional or mutable data.
6.  Format the output as a single, clean SQL code block."

---
 chained-output/1.4-final-ddl.sql 
---

**🔄 Feedback & Refinement Loop 1**

**User Prompt:**
"Review the generated DDL in `1.4-final-ddl.sql`. The `[Example_Column_Name]` in the `[Example_Table_Name]` table should be `TEXT` instead of `VARCHAR(255)` to accommodate longer entries as seen in the UI's text areas. Additionally, the foreign key from `[Table_A]` to `[Table_B]` should include an `ON DELETE SET NULL` clause. Please regenerate the DDL with these adjustments."
Phase 2: Data Population and Validation Plan

Objective: Create a strategy and generate SQL INSERT statements to populate the newly defined schema with realistic sample data, enabling immediate testing and validation of the V4 application.

** chained-prompts/phase-2-data-population **


**➡️ Prompt 2.1: Develop a Data Population Strategy**

**System Instruction:** You are a test data management specialist. Your task is to create a clear strategy for generating realistic sample data by analyzing the application's UI and data requirements.

**User Prompt:**
"Based on the generated DDL in `1.4-final-ddl.sql` and the UI elements (especially `input` components) within `Application_Definition.json`, create a data population strategy.

For each table in the DDL, create a Markdown table with columns: `Table Name`, `Column Name`, and `Data Generation Strategy`.
The strategy should be one of the following:
- **Literal Value:** For status fields or types, use values found directly in the JSON (e.g., from `mcq` options like 'Completed', 'In Progress').
- **Realistic Placeholder:** For fields like names, addresses, or descriptions, use realistic but fake data (e.g., 'John Doe', '123 Maple St').
- **Sequential ID:** For primary keys, generate sequential numbers or UUIDs.
- **Relational FK:** For foreign keys, specify that the value must be a valid ID from the referenced table.
- **Dynamic Value:** For timestamps or user IDs, use SQL functions like `NOW()` or placeholders like `'user-123'`.
- **Derived from UI:** For default values or static text in the UI that implies stored data."

---
 chained-output/2.1-data-population-strategy.md 
---

**➡️ Prompt 2.2: Generate SQL INSERT Statements**

**System Instruction:** You are a database developer. Your task is to write a complete SQL script that populates a database according to a predefined strategy, ensuring all relational constraints are respected.

**User Prompt:**
"Using the data population strategy from `2.1-data-population-strategy.md` and the DDL schema from `1.4-final-ddl.sql`, generate a complete SQL script with `INSERT` statements to populate all tables.

**Instructions:**
1.  Generate 3-5 realistic sample rows for each table.
2.  **Crucially, ensure relational integrity.** When inserting a row with a foreign key, make sure the value corresponds to a primary key from a previously inserted row in the referenced table.
3.  Use `RETURNING id` where necessary if you need to capture a newly generated primary key for use in a subsequent `INSERT` statement.
4.  Structure the script logically, populating parent tables (like `customer_information`) before child tables (like `tickets_information`).
5.  Format the entire output as a single, executable SQL code block."

---
 chained-output/2.2-population-script.sql 
---

**🔄 Feedback & Refinement Loop 2**

**User Prompt:**
"Review the script in `2.2-population-script.sql`. The sample data for the `tickets_information` table only includes 'Open' and 'Completed' statuses. Please add at least one record with an 'In Progress' status to ensure all states are represented for testing. Regenerate the `INSERT` statements for the `tickets_information` table and any dependent tables."
