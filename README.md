# MCP Crash Course for Python Developers

The Model Context Protocol (MCP) is a framework that helps Python developers build AI applications by connecting large language models (LLMs) to external tools and data in a standardized way. This crash course covers the essentials of MCP, from understanding its basics to creating servers and clients using tools, resources, and prompts.

## Table of Contents

1. [Introduction and Context](./1-introduction-and-context/README.md)
2. [Understanding MCP](./2-understanding-mcp/README.md)
3. [Simple Server Setup with Python SDK](./3-building-a-server/README.md)
4. [OpenAI Integration](./4-openai-integration/README.md)
5. [MCP vs. Function Calling](./5-function-calling-vs-mcp/README.md)

## Sett-vs-mcping Up Your Environment

To get started, set up your development environment with the MCP Python SDK.

### Install Dependencies
Install required packages from `requirements.txt` in your project directory (`crash-course`):
```bash
# Preferred: Use uv for dependency management
uv pip install -r requirements.txt

# Or use pip
pip install -r requirements.txt
```
Ensure `mcp` is included in `requirements.txt`. If not, install it:
```bash
pip install mcp
```

### MCP CLI Commands
The MCP CLI offers tools for testing and running servers:
```bash
# Test a server with the MCP Inspector (web-based tool)
mcp dev server.py

# Add a server to Claude Desktop
mcp install server.py

# Run a server directly
mcp run server.py
```

### Example: Running in `3-simple-server-setup`
In your `3-simple-server-setup` directory (`client-sse.py`, `client-stdio.py`, `README.md`, `server.py`):
- Verify `server.py` matches the example from Part 3:
  ```python
  from mcp.server.fastmcp import FastMCP

  mcp = FastMCP("DemoServer")

  @mcp.tool()
  def say_hello(name: str) -> str:
      """Greets a person by name."""
      return f"Hello, {name}! Nice to meet you."

  if __name__ == "__main__":
      mcp.run()
  ```
- Run the server:
  ```bash
  python server.py
  ```
  Or test with the MCP Inspector:
  ```bash
  mcp dev server.py
  ```

## Troubleshooting Previous Errors
- **No `run` Script**: You previously tried running a non-existent `run` script (`/home/abdiel/ai-cookbook/mcp/crash-course/3-simple-server-setup/run`). Use `python server.py` or `mcp dev server.py` instead.
- **ECONNREFUSED on Port 3001**: The `mcp dev server.py` error (`connect ECONNREFUSED ::1:3001, 127.0.0.1:3001`) occurred because `server.py` used stdio transport, but the MCP Inspector expected SSE on port 3001. To fix, update `server.py` for SSE:
  ```python
  mcp = FastMCP("DemoServer", host="127.0.0.1", port=3001)
  mcp.run(transport="sse")
  ```
  Then run:
  ```bash
  python server.py
  mcp dev server.py
  ```
- **SSE Deprecation**: The warning about SSE being replaced by "streamable-http" suggests a newer MCP version. Check for updates:
  ```bash
  pip install --upgrade mcp
  ```
  If "streamable-http" is required, consult [MCP documentation](https://modelcontextprotocol.io) for configuration details.

## Resources for Learning More
- [MCP Documentation](https://modelcontextprotocol.io)
- [MCP Specification](https://spec.modelcontextprotocol.io)
- [Python SDK GitHub](https://github.com/modelcontextprotocol/python-sdk)
- [Supported MCP Servers](https://github.com/modelcontextprotocol/servers)
- [MCP Architecture](https://modelcontextprotocol.io/docs/concepts/architecture)

## Next Steps
- Start with `3-simple-server-setup` by running `python client-stdio.py` to test the `say_hello` tool.
- Explore the OpenAI integration in `4-openai-integration` using `client.py` and `data/kb.json` (from previous sections).
- Check `README.md` files in each directory for specific instructions.