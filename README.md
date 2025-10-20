# Altibuddy: Altibase AI buddy

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Additional Notes](#additional-notes)

## Introduction

Altibuddy connects AI assistants (like Claude Desktop) to your databases, enabling:
- Natural language SQL query generation
- Intelligent context(tables and artifacts) search using vector embeddings
- Secure database operations with fine-grained permissions

Altibuddy provides two main components:
- **MCP Server (`altibuddy`)**: Model Context Protocol server that connects AI assistants to your Altibase databases
- **Management Server (`altibuddy-manage`)**: Web-based dashboard for configuration and management
- **Architecture**
```
    ┌─────────────────┐
    │  MCP Client     |
    |(Claude Desktop) │
    │(Gemini CLI)     │
    └────────┬────────┘
             │ MCP Protocol
             │ 
    ┌────────▼────────┐      ┌───────────────────┐
    │   MCP Server    │      │ Management Web UI │
    │   (altibuddy)   │◄────►│ (altibuddy-manage)│
    └────────┬────────┘      └───────────────────┘
             │
    ┌────────▼─────────────────┐
    │   Core Services          │
    │  - Database Manager      │
    │  - Vector Store (SQLite) │
    └────────┬─────────────────┘
             │
    ┌────────▼─────────────────┐
    │  Target Databases        │
    │  - Altibase (JDBC)       │
    └──────────────────────────┘
```

## Prerequisites
- **Operating System**
   - Windows x64
- **Python**
   - Required for Python package running
- Microsoft Visual C++ Redistributable for Visual Studio 2015-2022
   - Required for making vector embeddings
   - Download: https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist
     - X64: https://aka.ms/vs/17/release/vc_redist.x64.exe
- **Java Runtime Environment (JRE)**
   - Required for JDBC database connections
   - Download: https://www.oracle.com/java/technologies/downloads/
- **AI Client** (choose one):
   - [Claude Desktop](https://claude.ai/download)
   - [Gemini CLI](https://github.com/google-gemini/gemini-cli)
   - Any MCP-compatible client
- **Database**
   - Altibase: Version 6.5 or above

## Getting Started

### Step 1: Start the Management Server
```bash
# Install uv if not already installed
pip install uv

# Download altibuddy-1.0.0-cp311-cp311-win_amd64.whl from this repository.

# Run management server
uvx --python 3.11.9 --from /path/to/the/package/altibuddy-1.0.0-cp311-cp311-win_amd64.whl altibuddy-manage
```

Open your browser to: **http://localhost:8000**

### Step 2: Configure Database Connection
- Navigate to the **Database Connections** tab
- **Edit** or **Add New Connection**

### Step 3: Configure SQL Permissions
- Navigate to the **Settings** tab
- All SQL types are **disabled** by default except SELECT SQL type
- Enable only necessary SQL types (INSERT, UPDATE, DELETE, CREATE, DROP, etc.) that you want to run using your AI assistant
- Use table whitelists for SQL operations on limited tables

### Step 4: Update Your Database Contexts
- Navigate to the **Actions** tab
- Click **Update DB Objects**
- **Upload artifacts** (user domain artifacts)
- Data is shared with your AI assistant. You need to exercise **caution** with sensitive data.

### Step 5: Configure MCP Client
- Locate your MCP client's config file and add altibuddy configuration:
```json
{
  "mcpServers": {
    "altibuddy": {
      "command": "uvx",
      "args": ["--from", "/path/to/your/package/altibuddy-1.0.0-cp311-cp311-win_amd64.whl", "altibuddy"],
      "env": {
        "JAVA_HOME": "/path/to/your/JRE"
      }
    }
  }
}
```

### Step 6: Start Your MCP Client
- That's it. Enjoy your AI assistant with altibuddy.

## Additional Notes

### Management Server (altibuddy-manage)
This HTTP server is only needed for management operations. You can keep it closed when you don't need management operations.

#### Configure Management Server HTTP Setting
- By setting .env file
  - altibuddy-manage searches for the .env file in the current working directory (CWD) where altibuddy-manage is executed.
  - It will also recursively search up the directory tree from the CWD until it finds .env file.
  ```bash
  ALTIBUDDY_HOST=127.0.0.1
  ALTIBUDDY_PORT=8000
  ```
- By setting environment variables (ALTIBUDDY_HOST, ALTIBUDDY_PORT)

#### Context Metadata Tab

###### Features
- Retrieve tables and artifacts context
- Search contexts by similarity
- Filter by context type

###### Context Types
- User Table: Tables and views from your database
- System Table: System catalog tables
- User Artifact: Documentation you upload
- System Artifact: System documentation

#### Settings Tab

###### Embedding Settings
- Provider (hugging-face, gemini, openai): Default is hugging-face
- Model name: Default is nlpai-lab/KURE-v1
- API key (if required)
- Dimensions must equal or lower than 3072
- Current model(nlpai-lab/KURE-v1) supports Korean and English vector embedding
- For support of other languages, replace the current model with a proper model, then click `Initialize Embeddings` and `Make Embeddings` in the Actions tab

###### LLM Conversation Settings
- fewshot_count: Number of few-shot records to return in table_context_details MCP tool
- list_contexts_count: Number of contexts to return in list_contexts MCP tool

#### Actions Tab

###### Features
- Upload documentation for AI context (Markdown or text)
- Upload multiple files
- Text extraction (pdf to markdown conversion)
- Chunking for optimal vector search
- Markdown ATX-style headers (#(1~6 level) Header) are used for optimal vector search

### MCP Server (altibuddy)

The MCP server provides AI assistants with tools to interact with your databases.

#### Context Search Tools
- list_altibase_database_connections
- list_altibase_database_artifact_contexts
- list_altibase_database_table_contexts
- altibase_database_table_context_details

#### SQL Execution Tools
- run_altibase_select_sql
- run_altibase_insert_sql
- run_altibase_other_sql

### Troubleshooting

#### SQL Execution Fails
- Symptom: "SQL type not allowed" error
- Solutions:
  - Check Settings tab - SQL Permissions
  - Enable required SQL type
  -  Check table whitelist if configured

#### Check Logs
- `altibuddy_http.log` - Management server logs
- `altibuddy_mcp.log` - MCP server logs
- MCP client's log

#### Embedded Database (SQLite)
- Windows: %LOCALAPPDATA%\altibuddy\altibuddy.db
- The sqlite_vec extension vec0 virtual table is used for vector search 
