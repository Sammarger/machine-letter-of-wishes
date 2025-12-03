# MVP: Markdown Command Execution via n8n

## Overview
This MVP enables commands written in a Markdown (`.md`) file to be parsed, executed, and documented automatically using an n8n workflow.  

The workflow not only **executes commands** (e.g., move files, send emails) but also **creates detailed logs** of every action taken for auditability.

---

## Input
- **File type**: Markdown (`.md`)  
- **Format**: Each file contains a command block with parameters, e.g.:

```markdown
Command:
- Move file

Target File Path:
- File

Destination Path:
- Destination
```

---

## n8n Workflow Structure

A simplified version of the machine readable wishes is a function that takes an XForms submission as an input, and returns a log of the execution.

[Input: XForms File] -> [Wishes Executed] -> [Output: Log of Execution]

### Switching from MD to XForms

XForms as a platform for creating a machine readable letter of wishes is far more functional than a simple markdown file.
- XForms submissions come in the form of JSON lists, removing the need for a custom MD to JSON file. 
- From a users perspective, using XForms in this manner provides the user with a set of straightforward options to choose from, massively reducing the chance of input errors occurring.


### 1. Input Node (Read MD File)
- **Node:** `Read/Write Files from Disk`  
- **Purpose:** Reads the `.md` file from disk (or could be replaced by `HTTP Request` if loading from URL).  
- **Result:** The raw Markdown file text is passed into the workflow.

### 2. Extract from File
- **Node:** `Extract from Text File`  
- **Purpose:** Isolates command content from the Markdown file.

### 3. Convert MD → JSON
- **Node:** `Function`  
- **Purpose:** Parses Markdown into structured JSON.  
- **Sample Conversion:**
```json
{
  "command": "Move file",
  "target_file_path": "File",
  "destination_path": "Destination"
}
```

**Sample Code:**
```javascript
const text = $json["data"].toString();

// Simple key-value parser
const lines = text.split("\n").map(l => l.trim()).filter(Boolean);
let result = {};
let currentKey = null;

for (let line of lines) {
  if (line.endsWith(":")) {
    currentKey = line.replace(":", "").toLowerCase().replace(/ /g, "_");
  } else if (line.startsWith("-") && currentKey) {
    result[currentKey] = line.replace("-", "").trim();
  }
}

return [{ json: result }];
```

### 4. Log Command Interpretation
- **Node:** `Function`  
- **Purpose:** Creates a structured log entry describing how the Markdown input was interpreted.  
- **Output Example:**
```json
{
  "timestamp": "2025-09-16T10:00:00Z",
  "interpreted_command": "Move file",
  "raw_input": "... original markdown ..."
}
```

### 5. Command Identification
- **Node:** `Switch` (mode: Rules)  
- **Property:** `command`  
- **Cases:**
  - `"Move file"` → Route to *Move File Command*
  - `"Email"` → Route to *Send a Message*
  - *(Extendable: `"Copy file"`, `"Upload to Drive"`, etc.)*

### 6. Command Execution

#### A. Move File Command (branch)
- **Node:** `Execute Command` (shell) or `Move Binary File`.  
- **Purpose:** Moves a file from `target_file_path` to `destination_path`.

#### B. Send a Message (branch)
- **Node:** `Gmail` (or other email service).  
- **Purpose:** Sends an email with parameters from the JSON.

### 7. Execution Logging
- **Nodes:** `Log Move File`, `Log Send Email`  
- **Purpose:** Records the outcome of each executed branch.  
- **Output Example:**
```json
{
  "timestamp": "2025-09-16T10:05:00Z",
  "command": "Move file",
  "parameters": {
    "target_file_path": "/home/user/source.txt",
    "destination_path": "/home/user/archive/"
  },
  "result": "Success"
}
```

### 8. Compile Logs
- **Node:** `Function`  
- **Purpose:** Aggregates all logs generated during the workflow run into one unified object.

### 9. Convert Logs → File
- **Node:** `Convert to Text File`  
- **Purpose:** Transforms JSON logs into a plain text/Markdown file.

### 10. Save Documentation
- **Node:** `Write File to Disk`  
- **Purpose:** Saves the execution log locally for permanent recordkeeping.

---

## Example Workflow Run

**Input File (`command.md`):**
```markdown
Command:
- Move file

Target File Path:
- /home/user/source.txt

Destination Path:
- /home/user/archive/
```

**Generated JSON:**
```json
{
  "command": "Move file",
  "target_file_path": "/home/user/source.txt",
  "destination_path": "/home/user/archive/"
}
```

**Execution Result Log:**
```json
{
  "timestamp": "2025-09-16T10:05:00Z",
  "command": "Move file",
  "parameters": {
    "target_file_path": "/home/user/source.txt",
    "destination_path": "/home/user/archive/"
  },
  "result": "Success"
}
```

---

## Outputs
1. **Executed Actions**  
