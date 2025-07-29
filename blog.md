<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Complete Guide to Creating an MCP Server

The **Model Context Protocol (MCP)** is an open standard developed by Anthropic that enables AI assistants to securely connect with external data sources and tools. Think of MCP as "USB-C for AI" - it provides a standardized way for AI applications like Claude Desktop, Cursor, and other LLM-powered tools to interact with your data and services[^1_1][^1_2].

## What is an MCP Server?

An MCP server acts as a bridge between AI applications and external systems, exposing three core types of functionality[^1_3][^1_4]:

- **Tools**: Functions that AI models can call to perform actions (like API calls, calculations, or data modifications)
- **Resources**: Read-only data sources that provide context to AI models (similar to GET endpoints)
- **Prompts**: Reusable templates that guide AI interactions


## Architecture Overview

MCP follows a client-server architecture with three main components[^1_5][^1_6]:

- **MCP Hosts**: AI applications (like Claude Desktop) that need access to external capabilities
- **MCP Clients**: Embedded within hosts, manage connections to MCP servers (1:1 relationship)
- **MCP Servers**: Lightweight programs that expose tools, resources, and prompts to clients


## Development Approaches

### 1. Python Development with FastMCP

FastMCP is the most popular Python framework for building MCP servers[^1_7][^1_8][^1_9]:

#### Installation

```bash
# Using uv (recommended)
uv pip install fastmcp

# Or using pip
pip install fastmcp
```


#### Basic Server Structure

```python
# server.py
from fastmcp import FastMCP

# Create server instance
mcp = FastMCP("My Server")

# Define a tool
@mcp.tool
def add_numbers(a: int, b: int) -> int:
    """Add two numbers together."""
    return a + b

# Define a resource
@mcp.resource("config://settings")
def get_config() -> dict:
    """Get application configuration."""
    return {"version": "1.0", "debug": True}

# Define a dynamic resource template
@mcp.resource("users://{user_id}")
def get_user(user_id: str) -> dict:
    """Get user information by ID."""
    return {"id": user_id, "name": f"User {user_id}"}

if __name__ == "__main__":
    mcp.run()
```


#### Running the Server

```bash
# Run locally for testing
fastmcp run server.py

# Or using standard Python
python server.py
```


### 2. TypeScript Development

TypeScript provides excellent support for MCP servers using the official SDK[^1_10][^1_11]:

#### Installation

```bash
npm install @modelcontextprotocol/sdk
```


#### Basic Server Structure

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
});

// Add a tool
server.tool("get_weather", "Get weather for a city", {
  city: { type: "string", description: "City name" }
}, async ({ city }) => {
  return {
    content: [{
      type: "text",
      text: `Weather in ${city}: Sunny, 22°C`
    }]
  };
});

// Start the server
const transport = new StdioServerTransport();
await server.connect(transport);
```


## Transport Methods

MCP supports multiple transport mechanisms for communication[^1_12][^1_13][^1_14]:

### 1. STDIO Transport (Default)

- Uses standard input/output streams
- Ideal for local development and CLI tools
- Simple configuration, no network setup required


### 2. Streamable HTTP Transport (Recommended for Remote)

- Modern transport method (2025-03-26 specification)
- Single HTTP endpoint for bidirectional communication
- Supports authentication and session management
- Better for production deployments


### 3. SSE Transport (Deprecated)

- Server-Sent Events based transport
- Being replaced by Streamable HTTP
- Still supported for backward compatibility


## Testing and Debugging

### MCP Inspector

The MCP Inspector is the primary tool for testing and debugging MCP servers[^1_15][^1_16][^1_17]:

```bash
# Install and run inspector
npx @modelcontextprotocol/inspector server.py

# For TypeScript servers
npx @modelcontextprotocol/inspector node build/index.js

# With custom ports
CLIENT_PORT=8080 SERVER_PORT=9000 npx @modelcontextprotocol/inspector server.py
```

The Inspector provides:

- Web-based UI at `http://localhost:6274`
- Interactive tool testing
- Resource exploration
- Real-time debugging and logging


### Testing Tools

```python
# Using FastMCP's built-in testing
from fastmcp import FastMCP, Client

mcp = FastMCP("Test Server")

async def test_server():
    async with Client(mcp) as client:
        result = await client.call_tool("add_numbers", {"a": 5, "b": 3})
        print(f"Result: {result.text}")
```


## Integration with AI Clients

### Claude Desktop Integration

Claude Desktop is the most popular MCP client[^1_18][^1_19][^1_20]:

#### Configuration File Location:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`


#### Configuration Example:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["path/to/server.py"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```


#### Using FastMCP CLI:

```bash
# Automatic installation to Claude Desktop
fastmcp install claude-desktop server.py --server-name "My Server"
```


### Other Supported Clients

MCP works with various AI-powered applications:

- **Cursor**: Code editor with AI assistance
- **VS Code**: With Copilot extensions
- **Windsurf**: AI-powered development environment
- **Zed**: Modern code editor


## Deployment Options

### Local Deployment

```bash
# Run server locally
python server.py

# Or with FastMCP
fastmcp run server.py
```


### Remote Deployment

#### Google Cloud Run

```bash
# Deploy to Cloud Run
gcloud run deploy mcp-server \
  --source . \
  --platform managed \
  --region us-central1 \
  --no-allow-unauthenticated
```


#### Cloudflare Workers

```bash
# Install Wrangler CLI
npm install -g wrangler

# Deploy to Cloudflare
wrangler deploy
```


#### Docker Deployment

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 8000

CMD ["python", "server.py"]
```


## Security Best Practices

### Authentication and Authorization

MCP supports OAuth 2.1-based authentication for remote servers[^1_21][^1_22][^1_23]:

```python
# FastMCP with authentication
from fastmcp import FastMCP
from fastmcp.auth import APIKeyAuthProvider

mcp = FastMCP("Secure Server")

# Configure API key authentication
auth_provider = APIKeyAuthProvider(
    keys=["your-secure-api-key"],
    header_name="X-API-Key"
)

mcp.run(transport="sse", auth=auth_provider)
```


### Security Considerations

1. **Input Validation**: Always validate and sanitize inputs to prevent injection attacks[^1_24][^1_25]
2. **Access Control**: Implement proper authentication and authorization
3. **Network Security**: Use HTTPS/TLS for remote deployments
4. **Credential Management**: Never hardcode secrets; use environment variables or secure vaults
5. **Minimal Permissions**: Grant only necessary permissions to MCP servers

## Advanced Features

### Server Composition

```python
# Combine multiple servers
from fastmcp import FastMCP

main_server = FastMCP("Main Server")
weather_server = FastMCP("Weather Server")

# Mount another server
main_server.mount(weather_server, prefix="weather")
```


### Dynamic Tool Generation

```python
# Generate tools from OpenAPI specs
from fastmcp import FastMCP

mcp = FastMCP.from_openapi("https://api.example.com/openapi.json")
```


### Resource Templates

```python
@mcp.resource("files://{path}")
def read_file(path: str) -> str:
    """Read file contents."""
    with open(path, 'r') as f:
        return f.read()
```


## Common Use Cases

### Database Integration

```python
@mcp.tool
def query_database(sql: str) -> list:
    """Execute SQL query."""
    # Add proper validation and security measures
    return execute_query(sql)
```


### API Wrapper

```python
@mcp.tool
async def fetch_weather(city: str) -> dict:
    """Get weather data for a city."""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.weather.com/v1/weather/{city}")
        return response.json()
```


### File System Access

```python
@mcp.resource("filesystem://{path}")
def read_file_resource(path: str) -> str:
    """Read file as resource."""
    return Path(path).read_text()
```


## Troubleshooting

### Common Issues

1. **Server Not Connecting**: Check transport configuration and ensure server is running
2. **Tool Not Found**: Verify tool registration and naming
3. **Permission Errors**: Check file permissions and API credentials
4. **Port Conflicts**: Use different ports or check for running processes

### Debugging Tips

1. **Use MCP Inspector**: Essential for testing and debugging
2. **Check Logs**: Enable verbose logging for detailed error information
3. **Validate JSON-RPC**: Ensure proper message formatting
4. **Test Incrementally**: Start simple and add complexity gradually

## Best Practices

### Development

- **Start Simple**: Begin with basic tools and resources, then add complexity
- **Use Type Hints**: Leverage Python type hints for automatic schema generation
- **Document Everything**: Provide clear docstrings for all tools and resources
- **Handle Errors Gracefully**: Implement proper error handling and user feedback


### Production

- **Monitor Performance**: Track server performance and response times
- **Implement Rate Limiting**: Prevent abuse with appropriate rate limiting
- **Log Extensively**: Maintain detailed logs for debugging and auditing
- **Regular Updates**: Keep dependencies and frameworks updated


### Security

- **Validate All Inputs**: Never trust user input without validation
- **Use Secure Transports**: Always use HTTPS/TLS for remote deployments
- **Implement Authentication**: Secure your servers with proper auth mechanisms
- **Regular Security Audits**: Periodically review and test security measures


## Resources and Community

### Official Documentation

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [FastMCP Documentation](https://gofastmcp.com/)
- [MCP GitHub Repository](https://github.com/modelcontextprotocol)


### Example Servers

- [Awesome MCP Servers](https://github.com/wong2/awesome-mcp-servers) - Curated list of community servers
- [Official Example Servers](https://github.com/modelcontextprotocol/servers) - Reference implementations


### Community

- Join the MCP community discussions on GitHub
- Share your servers and contribute to the ecosystem
- Follow best practices and security guidelines

Creating an MCP server opens up powerful possibilities for AI integration. Whether you're building simple tools for personal use or complex enterprise integrations, MCP provides the standardized foundation needed to connect AI models with your data and services securely and efficiently.

<div style="text-align: center">⁂</div>

[^1_1]: https://blog.marcnuri.com/connecting-to-mcp-server-with-langchainjs

[^1_2]: https://github.com/BigUncle/MCP-Server-Unified-Deployment

[^1_3]: https://hackteam.io/blog/build-your-first-mcp-server-with-typescript-in-under-10-minutes/

[^1_4]: https://github.com/modelcontextprotocol/servers

[^1_5]: https://composio.dev/blog/mcp-server-step-by-step-guide-to-building-from-scrtch

[^1_6]: https://developers.cloudflare.com/agents/guides/remote-mcp-server/

[^1_7]: https://gofastmcp.com/tutorials/create-mcp-server

[^1_8]: https://www.pondhouse-data.com/blog/create-mcp-server-with-fastmcp

[^1_9]: https://developers.cloudflare.com/agents/model-context-protocol/transport/

[^1_10]: https://simplescraper.io/blog/how-to-mcp

[^1_11]: https://learn.microsoft.com/nl-nl/azure/developer/ai/build-mcp-server-ts

[^1_12]: https://mcp-framework.com/docs/Transports/stdio-transport/

[^1_13]: https://www.youtube.com/watch?v=rnljvmHorQw

[^1_14]: https://docs.anthropic.com/en/docs/claude-code/mcp

[^1_15]: https://support.anthropic.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop

[^1_16]: https://gofastmcp.com/integrations/claude-desktop

[^1_17]: https://apidog.com/blog/mcp-server-connect-claude-desktop/

[^1_18]: https://mcp-framework.com/docs/Authentication/overview/

[^1_19]: https://stytch.com/blog/mcp-authentication-and-authorization-servers/

[^1_20]: https://www.infracloud.io/blogs/securing-mcp-servers/

[^1_21]: https://www.philschmid.de/mcp-introduction

[^1_22]: https://github.com/modelcontextprotocol/inspector

[^1_23]: https://www.youtube.com/watch?v=98l_k0XYXKs

[^1_24]: https://mcpcat.io/guides/setting-up-mcp-inspector-server-testing/

[^1_25]: https://snyk.io/articles/how-to-debug-mcp-server-with-anthropic-inspector/

[^1_26]: https://devblogs.microsoft.com/dotnet/build-a-model-context-protocol-mcp-server-in-csharp/

[^1_27]: https://www.youtube.com/watch?v=jLM6n4mdRuA

[^1_28]: https://www.youtube.com/watch?v=MC2BwMGFRx4

[^1_29]: https://www.reddit.com/r/ClaudeAI/comments/1hoafi1/introducing_mcpframework_build_a_mcp_server_in_5/

[^1_30]: https://dev.to/debs_obrien/building-your-first-mcp-server-a-beginners-tutorial-5fag

[^1_31]: https://www.datacamp.com/tutorial/mcp-model-context-protocol

[^1_32]: https://towardsdatascience.com/model-context-protocol-mcp-tutorial-build-your-first-mcp-server-in-6-steps/

[^1_33]: https://www.digitalocean.com/community/tutorials/model-context-protocol

[^1_34]: https://platform.openai.com/docs/mcp

[^1_35]: https://modelcontextprotocol.io/quickstart/user

[^1_36]: https://www.youtube.com/watch?v=rXdAUwsJmnc

[^1_37]: https://modelcontextprotocol.io/quickstart/server

[^1_38]: https://www.builder.io/blog/mcp-server

[^1_39]: https://treblle.com/blog/mcp-servers-guide

[^1_40]: https://www.youtube.com/watch?v=nTMSyldeVSw

[^1_41]: https://www.youtube.com/watch?v=egVm_z1nnnQ

[^1_42]: https://www.youtube.com/watch?v=CiArUs_2jm4

[^1_43]: https://www.youtube.com/watch?v=vzGkSn59rDU

[^1_44]: https://www.youtube.com/watch?v=Zw3sfAIpeH8

[^1_45]: https://www.youtube.com/watch?v=DfWHX7kszQI

[^1_46]: https://github.com/ruslanmv/Simple-MCP-Server-with-Python

[^1_47]: https://github.com/modelcontextprotocol/typescript-sdk

[^1_48]: https://www.anthropic.com/news/model-context-protocol

[^1_49]: https://www.digitalocean.com/community/tutorials/mcp-server-python

[^1_50]: https://www.youtube.com/watch?v=-WogqfxWBbM

[^1_51]: https://www.freecodecamp.org/news/how-to-build-a-custom-mcp-server-with-typescript-a-handbook-for-developers/

[^1_52]: https://github.com/modelcontextprotocol/python-sdk

[^1_53]: https://github.com/punkpeye/fastmcp

[^1_54]: https://scrapfly.io/blog/posts/how-to-build-an-mcp-server-in-python-a-complete-guide

[^1_55]: https://dev.to/shadid12/how-to-build-mcp-servers-with-typescript-sdk-1c28

[^1_56]: https://developer.hashicorp.com/terraform/docs/tools/mcp-server/deploy

[^1_57]: https://www.merge.dev/blog/mcp-best-practices

[^1_58]: https://milvus.io/ai-quick-reference/whats-the-best-way-to-deploy-an-model-context-protocol-mcp-server-to-production

[^1_59]: https://www.claudemcp.com/specification

[^1_60]: https://www.thoughtworks.com/insights/blog/generative-ai/model-context-protocol-beneath-hype

[^1_61]: https://treblle.com/blog/model-context-protocol-guide

[^1_62]: https://www.redhat.com/en/blog/model-context-protocol-mcp-understanding-security-risks-and-controls

[^1_63]: https://www.linkedin.com/pulse/securing-model-context-protocol-mcp-challenges-best-muayad-sayed-ali-sot4e

[^1_64]: https://cloud.google.com/blog/topics/developers-practitioners/build-and-deploy-a-remote-mcp-server-to-google-cloud-run-in-under-10-minutes

[^1_65]: https://google.github.io/adk-docs/tools/mcp-tools/

[^1_66]: https://developers.cloudflare.com/agents/model-context-protocol/

[^1_67]: https://www.speakeasy.com/mcp/using-mcp/remote-servers

[^1_68]: https://docs.anthropic.com/en/docs/mcp

[^1_69]: https://docs.cursor.com/context/model-context-protocol

[^1_70]: https://www.youtube.com/watch?v=6ZVRJKw5q9g

[^1_71]: https://www.anthropic.com/engineering/desktop-extensions

[^1_72]: https://mcp-framework.com/docs/debugging/

[^1_73]: https://www.trendmicro.com/vinfo/nl/security/news/cybercrime-and-digital-threats/mcp-security-network-exposed-servers-are-backdoors-to-your-private-data

[^1_74]: https://mcp.so/client/online-mcp-inspector/web-mcp

[^1_75]: https://www.reddit.com/r/ClaudeAI/comments/1h9lau7/is_there_a_step_by_step_guide_to_set_up_mcp_with/

[^1_76]: https://www.descope.com/blog/post/mcp-auth-spec

[^1_77]: https://www.youtube.com/watch?v=vDT1_b5eEkM

[^1_78]: https://modelcontextprotocol.io/specification/2025-06-18/basic/security_best_practices

[^1_79]: https://www.youtube.com/watch?v=Y0tZ35dFFx4

[^1_80]: https://github.com/sparfenyuk/mcp-proxy

[^1_81]: https://github.com/wong2/awesome-mcp-servers

[^1_82]: https://github.com/jlowin/fastmcp

[^1_83]: https://mcp-framework.com/docs/Transports/transports-overview/

[^1_84]: https://modelcontextprotocol.io/examples

[^1_85]: https://www.datacamp.com/tutorial/building-mcp-server-client-fastmcp

[^1_86]: https://brightdata.com/blog/ai/sse-vs-streamable-http

[^1_87]: https://modelcontextprotocol.io/clients

[^1_88]: https://www.reddit.com/r/modelcontextprotocol/comments/1hrq8x9/how_to_build_mcp_servers_with_fastmcp_stepbystep/
