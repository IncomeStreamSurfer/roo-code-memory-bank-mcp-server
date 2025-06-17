[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/incomestreamsurfer-roo-code-memory-bank-mcp-server-badge.png)](https://mseep.ai/app/incomestreamsurfer-roo-code-memory-bank-mcp-server)

# Roo Code Memory Bank MCP Server

This project implements the core functionality of the [Roo Code Memory Bank](https://github.com/GreatScottyMac/roo-code-memory-bank) system as a Model Context Protocol (MCP) server. It allows AI assistants to maintain project context across sessions by interacting with a file-based memory bank using structured MCP tools.

## Features

This MCP server provides the following tools:

*   **`initialize_memory_bank`**: Creates the `memory-bank/` directory and standard `.md` files (`productContext.md`, `activeContext.md`, `progress.md`, `decisionLog.md`, `systemPatterns.md`) with initial templates.
    *   *Input*: (Optional) `{ "project_brief_content": string }`
    *   *Output*: `{ "status": "success" | "error", "messages"?: string[], "message"?: string }`
*   **`check_memory_bank_status`**: Checks if the `memory-bank/` directory exists and lists the `.md` files within it.
    *   *Input*: `{}`
    *   *Output*: `{ "exists": boolean, "files": string[] }`
*   **`read_memory_bank_file`**: Reads the full content of a specified memory bank file.
    *   *Input*: `{ "file_name": string }`
    *   *Output*: `{ "content": string }` or error object.
*   **`append_memory_bank_entry`**: Appends a new, timestamped entry to a specified file, optionally under a specific markdown header. Creates the file if it doesn't exist.
    *   *Input*: `{ "file_name": string, "entry": string, "section_header"?: string }`
    *   *Output*: `{ "status": "success" | "error", "message": string }`

## Prerequisites

*   Node.js (v18 or later recommended)
*   npm (usually included with Node.js)
*   An MCP client environment (like the one used by Cline) capable of managing and launching MCP servers.

## Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/IncomeStreamSurfer/roo-code-memory-bank-mcp-server.git
    cd roo-code-memory-bank-mcp-server
    ```
2.  **Install dependencies:**
    ```bash
    npm install
    ```
3.  **Build the project:**
    ```bash
    npm run build
    ```
    This compiles the TypeScript code into JavaScript in the `dist/` directory.

## Configuration (for Cline MCP Client)

To make this server available to your AI assistant (like Cline), you need to add its configuration to your MCP settings file (e.g., `cline_mcp_settings.json`).

Find the `mcpServers` object in your settings file and add the following entry:

```json
{
  "mcpServers": {
    // ... other server configurations ...

    "roo-code-memory-bank-mcp": {
      "autoApprove": [
        "initialize_memory_bank",
        "check_memory_bank_status",
        "read_memory_bank_file",
        "append_memory_bank_entry"
      ],
      "disabled": false,
      "timeout": 60,
      "command": "node", // Or "cmd.exe" with "/c node ..." on Windows if needed
      "args": [
        // IMPORTANT: Replace this path with the actual absolute path
        // to the compiled index.js file on your system
        "/path/to/your/cloned/repo/roo-code-memory-bank-mcp-server/dist/index.js"
      ],
      "env": {},
      "transportType": "stdio"
    }

    // ... other server configurations ...
  }
}
```

**Important:** Replace `/path/to/your/cloned/repo/` with the correct absolute path to where you cloned the repository on your machine. Ensure the path separator is correct for your operating system (e.g., use backslashes `\` on Windows).

## Running the Server

You don't typically need to run the server manually. The MCP client (like Cline) will automatically start the server using the `command` and `args` specified in the configuration file when one of its tools is called for the first time.

If you want to test it manually, you can run `npm start` from the project directory.

## Usage

The AI assistant interacts with the server using the defined tools. The typical workflow involves:

1.  Checking the memory bank status (`check_memory_bank_status`).
2.  Initializing if needed (`initialize_memory_bank`).
3.  Reading relevant files (`read_memory_bank_file`) to gain context.
4.  Appending entries (`append_memory_bank_entry`) as decisions are made or progress occurs.

The `memory-bank/` directory will be created in the same directory where the server process is started (which should be the root of this project directory when launched via the MCP client configuration).


### Custom instructions

Set these instructions inside Roo

You must use MCPs where needed

I have a specific MCP flow:

Use context7 to find any relevant documentation pieces you will need for this process, ensure to feed any relevant knoweldege to any relevant subtasks - use context7 at all times to do research on important documentation if you're unsure of something
Use google maps mcp in order to search for <niche> + <county> - this will allow us to find the basic businesses we need to accomplish our task
Use brave search mcp to find URLs to scrape
Use fetch mcp with fetch_txt and fetch_markdown to find text and images on pages in order to convert into JSON files and create something in-depth
Use openrouter search to find general sentiment of topics, reviews, etc.

**Utilize the `roo-code-memory-bank-mcp` server to maintain project context:**
*   At the start of a task or significant subtask, use `check_memory_bank_status`.
*   If the memory bank exists (`exists: true`), use `read_memory_bank_file` for relevant files (e.g., `productContext.md`, `activeContext.md`) to load the current project context.
*   Incorporate this loaded context into your planning and execution.
*   When making significant decisions, progress updates, or architectural changes, use `append_memory_bank_entry` to record the information in the appropriate file (`decisionLog.md`, `progress.md`, etc.), ensuring context persistence.
*   If the memory bank doesn't exist, consider using `initialize_memory_bank` if appropriate for the project.