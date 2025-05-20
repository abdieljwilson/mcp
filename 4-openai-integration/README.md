# OpenAI Integration with MCP

This guide explains how to connect OpenAI's API to an MCP server, enabling OpenAI to use tools provided by the server to answer user queries.

## What You’ll Do

This example covers:
1. Building an MCP server with a tool to access a knowledge base.
2. Linking OpenAI to the MCP server.
3. Enabling OpenAI to use the server’s tools dynamically for user queries.

## How It Connects

This example uses **stdio transport**, meaning:
- The client and server run in the same process.
- The client starts the server automatically as a subprocess.
- No separate server process is needed.

For separate client and server setups (e.g., on different machines), use **SSE (Server-Sent Events)** transport instead. See the [Simple Server Setup](../3-simple-server-setup) section for SSE details.

### How Data Flows

1. **User Query**: User asks a question (e.g., "What’s the company’s vacation policy?").
2. **OpenAI API**: Receives the query and sees available MCP tools.
3. **Tool Selection**: OpenAI picks the right tool for the query.
4. **MCP Client**: Sends OpenAI’s tool request to the MCP server.
5. **MCP Server**: Runs the tool (e.g., fetches knowledge base data).
6. **Response**: Tool results go back to OpenAI via the client.
7. **Final Answer**: OpenAI uses the tool results to respond.

## How OpenAI Uses MCP Tools

OpenAI’s function-calling works with MCP tools by:
1. **Registering Tools**: The MCP client converts MCP tools to OpenAI’s format.
2. **Choosing Tools**: OpenAI selects tools based on the query.
3. **Running Tools**: The MCP client executes the tools and returns results.
4. **Using Results**: OpenAI includes tool results in its response.

## Why Use MCP?

MCP acts as a bridge between OpenAI and your backend:
- **Standardized**: Provides a consistent way for AI to use tools.
- **Simple**: Hides complex backend details from OpenAI.
- **Secure**: Controls which tools and data OpenAI can access.
- **Flexible**: Lets you update the backend without changing the AI setup.

## Implementation

### Server (`server.py`)

The MCP server provides a `get_knowledge_base` tool that retrieves Q&A pairs from a JSON file.

Example:
```python
from mcp.server.fastmcp import FastMCP
import json

mcp = FastMCP("KnowledgeBaseServer")

@mcp.tool()
def get_knowledge_base(query: str) -> str:
    """Retrieve answers from the knowledge base."""
    with open("data/kb.json", "r") as f:
        kb = json.load(f)
    return kb.get(query, "No answer found.")

if __name__ == "__main__":
    mcp.run()
```

### Client (`client.py`)

The client:
1. Connects to the MCP server via stdio.
2. Converts MCP tools to OpenAI’s function format.
3. Manages communication between OpenAI and the MCP server.
4. Processes tool results for OpenAI’s final response.

Example:
```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
import openai

async def main():
    server_params = StdioServerParameters(command="python", args=["server.py"])
    async with stdio_client(server_params) as (read_stream, write_stream):
        async with ClientSession(read_stream, write_stream) as session:
            await session.initialize()
            tools = await session.list_tools()
            # Convert MCP tools to OpenAI format
            openai_tools = [{"type": "function", "function": {"name": t.name, "description": t.description}} for t in tools.tools]
            # Call OpenAI API
            client = openai.AsyncOpenAI(api_key="your-api-key")
            response = await client.chat.completions.create(
                model="gpt-4",
                messages=[{"role": "user", "content": "What’s the vacation policy?"}],
                tools=openai_tools
            )
            # Handle tool call
            if response.choices[0].message.tool_calls:
                tool_call = response.choices[0].message.tool_calls[0]
                result = await session.call_tool(tool_call.function.name, arguments=tool_call.function.arguments)
                print(f"Tool Result: {result.content[0].text}")

if __name__ == "__main__":
    asyncio.run(main())
```

### Knowledge Base (`data/kb.json`)

A JSON file with Q&A pairs, e.g.:
```json
{
  "What’s the vacation policy?": "Employees get 15 days of paid vacation per year.",
  "What’s the remote work policy?": "Remote work is allowed two days per week."
}
```

## Running the Example

1. **Install Dependencies**:
   Ensure `mcp` and `openai` are installed:
   ```bash
   pip install mcp openai
   ```
   Check your `requirements.txt` in the parent directory (`crash-course`) and install:
   ```bash
   pip install -r ../requirements.txt
   ```

2. **Set Up OpenAI API Key**:
   Create a `.env` file in `3-simple-server-setup`:
   ```bash
   echo "OPENAI_API_KEY=your-api-key-here" > .env
   ```
   Load the `.env` file in `client.py` by adding:
   ```python
   from dotenv import load_dotenv
   import os
   load_dotenv()
   openai.api_key = os.getenv("OPENAI_API_KEY")
   ```
   Replace `"your-api-key"` in `client.py` with `os.getenv("OPENAI_API_KEY")`.

3. **Create the Knowledge Base**:
   Create a `data` directory and `kb.json`:
   ```bash
   mkdir data
   echo '{"What’s the vacation policy?": "Employees get 15 days of paid vacation per year."}' > data/kb.json
   ```

4. **Run the Client**:
   Since this uses stdio, the client starts the server automatically:
   ```bash
   python client.py
   ```
   Expected output:
   ```
   Tool Result: Employees get 15 days of paid vacation per year.
   ```

## Troubleshooting Previous Errors

Your earlier errors (`ECONNREFUSED` and `No such file or directory: run`) suggest issues with the MCP setup:
- **No `run` Script**: Your directory (`client-sse.py`, `client-stdio.py`, `README.md`, `server.py`) doesn’t include a `run` script. Use `python server.py` or `mcp dev server.py` instead.
- **ECONNREFUSED on Port 3001**: The MCP Inspector tried to connect to `http://localhost:3001/sse`, but `server.py` wasn’t running with SSE. For OpenAI integration with stdio, you don’t need SSE or the Inspector, so run `python client.py` as shown above.

If you want to test with the Inspector:
- Update `server.py` for SSE:
  ```python
  mcp = FastMCP("KnowledgeBaseServer", host="127.0.0.1", port=3001)
  mcp.run(transport="sse")
  ```
- Start the server:
  ```bash
  python server.py
  ```
- Run the Inspector:
  ```bash
  mcp dev server.py
  ```

## Notes
- The stdio transport simplifies development by running the server and client together.
- For production, consider SSE transport to separate the client and server. Update `client.py` to use `sse_client` (see [Simple Server Setup](../3-simple-server-setup)).
- Check the MCP GitHub for updates on the deprecated SSE transport: [MCP Servers](https://github.com/modelcontextprotocol/servers).