Workflow 20: Duplicate Data Detection System

1. Workflow Overview
Workflow Name
Duplicate Data Detection System
Objective
Identify duplicate records within a dataset by comparing selected fields, flag duplicate records, separate unique records, and optionally prepare duplicates for merging or cleanup.

2. Business Use Cases
	• CRM Lead Deduplication 
	• Customer Database Cleanup 
	• Employee Record Validation 
	• Product Catalog Cleanup 
	• Inventory Duplicate Detection 
	• Data Migration Validation 
	• Excel/CSV Import Validation 
	• E-commerce Customer Import Cleanup 

3. Workflow Architecture

Manual Trigger
      │
      ▼
Set
(Create Sample Dataset)
      │
      ▼
Code
(Convert Array → Items)
      │
      ▼
Set
(Create Duplicate Key)
      │
      ▼
Code
(Group Records)
      │
      ▼
IF
(count > 1 ?)
     │
 ┌───┴──────────┐
 │              │
 ▼              ▼
Code          Code
Duplicates    Unique
 │              │
 └──────┬───────┘
        ▼
      Merge
 (Append Mode)
        │
        ▼
       IF
(is_duplicate?)
    │        │
    ▼        ▼
Duplicate  Unique

4. Complete Workflow Steps

STEP 1 — Manual Trigger
Node
Manual Trigger
Purpose
Starts the workflow manually.
Configuration
No configuration required.
Output
Workflow execution begins.

STEP 2 — Create Sample Dataset
Node
Set
Purpose
Creates a sample dataset.
Configuration
Keep Only Set

No
Add Field

records
Type

Array
Paste

[
  { "id": 1, "name": "John", "email": "john@example.com" },
  { "id": 2, "name": "Alice", "email": "alice@example.com" },
  { "id": 3, "name": "John", "email": "john@example.com" },
  { "id": 4, "name": "Bob", "email": "bob@example.com" },
  { "id": 5, "name": "Alice", "email": "alice@example.com" }
]
Output
One item containing

records[]

STEP 3 — Convert Array into Individual Items
Node
Code
Mode
Run Once for All Items
Code

const records = $input.first().json.records;

return records.map(record => ({
  json: record
}));
Purpose
Converts the array into separate items.
Output
Five individual records.

STEP 4 — Create Duplicate Key
Node
Set
Purpose
Creates a comparison key.
Important
Include Other Input Fields

ON
Add Field

duplicate_key
Expression

={{ $json.name.toLowerCase().trim() + "|" + $json.email.toLowerCase().trim() }}
Output Example

{
"id":1,
"name":"John",
"email":"john@example.com",
"duplicate_key":"john|john@example.com"
}

STEP 5 — Group Records
Node
Code
Mode
Run Once for All Items
Code

const items = $input.all();

const groups = {};

for (const item of items) {

  const key = item.json.duplicate_key;

  if (!groups[key]) {
    groups[key] = [];
  }

  groups[key].push(item.json);

}

return Object.entries(groups).map(([duplicate_key, records]) => ({
  json: {
    duplicate_key,
    count: records.length,
    records
  }
}));
Purpose
Groups all records having the same duplicate key.
Output

John  -> count 2
Alice -> count 2
Bob   -> count 1

STEP 6 — Detect Duplicate Groups
Node
IF
Condition
Value 1

={{$json.count}}
Operation

Greater Than
Value 2

1
True
Duplicate groups
False
Unique groups

STEP 7 — Expand Duplicate Records
Node
Code
(True Output)
Mode
Run Once for All Items
Code

const output = [];

for (const item of $input.all()) {

  for (const record of item.json.records) {

    output.push({
      json:{
        ...record,
        is_duplicate:true
      }
    });

  }

}

return output;
Output
Four duplicate records

John
John
Alice
Alice

STEP 8 — Expand Unique Records
Node
Code
(False Output)
Mode
Run Once for All Items
Code

const output = [];

for (const item of $input.all()) {

  for (const record of item.json.records) {

    output.push({
      json:{
        ...record,
        is_duplicate:false
      }
    });

  }

}

return output;
Output
One record

Bob

STEP 9 — Merge Both Branches
Node
Merge
Mode

Append
Input 1
Duplicate branch
Input 2
Unique branch
Output
Five records
ID	Name	Duplicate
1	John	True
2	Alice	True
3	John	True
4	Bob	False
5	Alice	True

STEP 10 — Separate Final Output
Node
IF
Condition
Value 1

={{$json.is_duplicate}}
Operation

Equals
Value 2

true
Output
True
Duplicate records
False
Unique records

5. Final Output
Duplicate Branch

John

John

Alice

Alice
Unique Branch

Bob

6. Real-World Examples
Example 1
CRM Lead Import
Duplicate email IDs are flagged before importing leads.

Example 2
Employee Database
Detect employees with the same official email.

Example 3
Customer Excel Upload
Prevent duplicate customer creation during bulk imports.

7. Common Mistakes
Problem
Only duplicate_key appears.
Solution
Enable

Include Other Input Fields

Problem
Everything goes to False branch.
Solution
IF condition should be

count > 1

Problem
Merge node returns only one branch.
Solution
Use

Append Mode

Problem
records is undefined.
Solution
Verify the Group Records Code node outputs:
	• duplicate_key 
	• count 
	• records array 

Problem
Final IF doesn't separate records.
Solution
Ensure is_duplicate is a Boolean (true/false), not a string.

8. Key Concepts Learned
	• Manual workflow triggering. 
	• Creating arrays with the Set node. 
	• Converting arrays into individual items. 
	• Building a composite comparison key. 
	• Grouping data using JavaScript in the Code node. 
	• Counting grouped records. 
	• Detecting duplicate groups with an IF node. 
	• Expanding grouped arrays back into individual items. 
	• Adding Boolean flags to records. 
	• Merging multiple branches using the Merge node. 
	• Separating final outputs into duplicate and unique datasets. 

9. Future Enhancements
You can extend this workflow by:
	1. Reading data from Google Sheets, Excel, CSV, or a Database instead of a Set node. 
	2. Comparing on multiple fields (e.g., name + email + phone). 
	3. Automatically removing duplicate records while keeping the latest or oldest entry. 
	4. Updating duplicate status back to Google Sheets, MySQL, or a CRM. 
	5. Sending duplicate records to Slack, Email, or Microsoft Teams for review. 
	6. Creating a reusable sub-workflow that can be called from other n8n workflows for duplicate detection. 
