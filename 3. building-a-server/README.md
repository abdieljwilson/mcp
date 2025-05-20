# MCP Crash Course for Python Developers: Part 3

## Setting Up a Simple MCP Server

This section shows how to create and run a basic MCP server using the Python SDK, and how to connect to it with a client.

### Creating Your First MCP Server

Here’s a simple MCP server with a tool that greets users:

```python
# server.py
from mcp.server.fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP("DemoServer")

# Define a tool
@mcp.tool()
def say_hello(name: str) -> str:
    """Greets a person by name."""
    return f"Hello, {name}! Nice to meet you."

# Run the server
if __name__ == "__main__":
    mcp.run()
```

This code:
- Sets up a server named "DemoServer".
- Defines a `say_hello` tool that takes a name and returns a greeting.
- Starts the server using the default stdio transport.

### Running the Server

You can run your MCP server in a few ways:

#### 1. Test with MCP Inspector
Use the MCP Inspector for easy testing:
```bash
mcp dev server.py
```
This runs the server locally and opens a web tool to interact with it.

#### 2. Integrate with Claude Desktop
If you use Claude Desktop, install the server:
```bash
mcp install server.py
```
This makes the server available to Claude Desktop.

#### 3. Run Directly
Run the server as a Python script:
```bash
python server.py
```
Or, for better dependency management:
```bash
uv run server.py
```

### How the Server Works
When you run the server:
1. It loads your defined tools (like `say_hello`).
2. It listens for connections via **stdio** (default, using standard input/output) or **SSE** (HTTP-based, if configured).

To use SSE for remote access, modify the server:
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("DemoServer", host="127.0.0.1", port=8050)

@mcp.tool()
def say_hello(name: str) -> str:
    """Greets a person by name."""
    return f"Hello, {name}! Nice to meet you."

if __name__ == "__main__":
    mcp.run(transport="sse")
```
Run it with:
```bash
python server.py
```
This starts the server at `http://127.0.0.1:8050`.

### Connecting with a Client (Stdio)

Here’s a client that connects to the server using stdio:
```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def main():
    # Define server parameters
    server_params = StdioServerParameters(command="python", args=["server.py"])

    # Connect to the server
    async with stdio_client(server_params) as (read_stream, write_stream):
        async with ClientSession(read_stream, write_stream) as session:
            await session.initialize()  # Start the connection
            # List tools
            tools = await session.list_tools()
            print("Available tools:")
            for tool in tools.tools:
                print(f"  - {tool.name}: {tool.description}")
            # Call the tool
            result = await session.call_tool("say_hello", arguments={"name": "Alice"})
            print(f"Result: {result.content[0].text}")

if __name__ == "__main__":
    asyncio.run(main())
```

This client:
- Connects to the server via stdio.
- Lists available tools.
- Calls the `say_hello` tool with the name "Alice".

### Connecting with a Client (SSE)

For an SSE-based connection:
```python
import asyncio
from mcp import ClientSession
from mcp.client.sse import sse_client

async def main():
    # Connect to the server
    async with sse_client("http://localhost:8050/sse") as (read_stream, write_stream):
        async with ClientSession(read_stream, write_stream) as session:
            await session.initialize()  # Start the connection
            # List tools
            tools = await session.list_tools()
            print("Available tools:")
            for tool in tools.tools:
                print(f"  - {tool.name}: {tool.description}")
            # Call the tool
            result = await session.call_tool("say_hello", arguments={"name": "Bob"})
            print(f"Result: {result.content[0].text}")

if __name__ == "__main__":
    asyncio.run(main())
```

This client connects to the server over HTTP/SSE and performs the same actions.

### Stdio vs. SSE: Which to Use?
- **Stdio**: Best for development or when the client and server run on the same machine. It’s simpler but less flexible.
- **SSE**: Ideal for production, where the server and client may be on different machines or containers. It’s more scalable.

For most production apps, use **SSE** for better flexibility and separation. Use **stdio** for quick testing or tightly coupled setups.