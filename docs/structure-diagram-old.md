# N8N Workflow Structure OLD

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


