# Weather Agent - Google ADK with MCP Server

AI agent built with **Google Agent Development Kit (ADK)** that uses tools from a local **MCP server** via Streamable HTTP transport.

Bài nộp được kiểm tra local; không thực hiện bonus deploy Cloud Run. Minh chứng nằm trong [screenshots](../../screenshots/README.md).

## Architecture

```
┌─────────────────┐      ┌──────────────────┐      ┌─────────────────────┐
│   User Browser  │ ───> │  ADK Web UI      │ ───> │  Weather Agent      │
│   localhost:8000│      │  (Google ADK)    │      │  (Agent with MCP)   │
└─────────────────┘      └──────────────────┘      └─────────────────────┘
                                                             │
                                                             │ Streamable HTTP
                                                             ▼
                                                   ┌─────────────────────┐
                                                   │  MCP Server         │
                                                   │  localhost:8085/mcp │
                                                   │  FastMCP + Tools    │
                                                   └─────────────────────┘
                                                             │
                                                             ▼
                                                   ┌─────────────────────┐
                                                   │  WeatherAPI.com     │
                                                   └─────────────────────┘
```

## Features

- **HTTP MCP Tools**: Connects to the local MCP server via Streamable HTTP
- **3 Weather Tools**:
  - `get_current_weather(city)` - Real-time weather conditions
  - `get_forecast(city, days)` - Weather forecast up to 3 days
  - `health_check()` - Server health verification
- **Web Interface**: UI via ADK web
- **Streaming Responses**: Real-time AI responses

## Quick Start

### 1. Start the MCP Server

```bash
cd ../mcp-server
export WEATHERAPI_KEY="your_weatherapi_key"
export PORT=8085
uv run python weather.py
```

### 2. Setup Environment

```bash
cd ../mcp-client

# Create .env file with your Google API key
# Get free key from: https://aistudio.google.com/apikey
# Create .env with an editor and save as UTF-8:
# GOOGLE_API_KEY=your_google_api_key_here
# MCP_SERVER_URL=http://localhost:8085/mcp
```

### 3. Install Dependencies

```bash
uv sync
uv run python verify_setup.py
```

### 4. Run the Agent

```bash
uv run adk web
```

### 5. Use the Agent

1. Open browser: http://localhost:8000
2. Select `weather_agent` from dropdown
3. Ask questions like:
   - "What's the weather in Brisbane?"
   - "Give me a 3-day forecast for Tokyo"
   - "How's the weather in New York?"

Trên Windows PowerShell, đặt khóa server bằng `$env:WEATHERAPI_KEY="..."` và `$env:PORT="8085"`. Dừng server Auth trước khi chạy ADK ở cổng 8000; hoặc dùng `uv run adk web --port 8001` và mở cổng tương ứng.

## Project Structure

```
mcp-client/
├── weather_agent/
│   ├── agent.py           # Main agent with MCP connection
│   └── __init__.py
├── pyproject.toml
├── .env                   # Environment variables (create this)
└── README.md
```

## Configuration

### Agent Configuration

In `weather_agent/agent.py`:

```python
MCP_SERVER_URL = os.getenv("MCP_SERVER_URL", "http://localhost:8085/mcp")

connection_params = StreamableHTTPConnectionParams(
    url=MCP_SERVER_URL,
    timeout=30.0,
)

root_agent = Agent(
    name="weather_agent",
    model="gemini-2.5-flash",
    tools=[weather_tools],
)
```

## Troubleshooting

### Agent won't connect to MCP server

1. **Connection refused / 404 errors**: server unavailable or wrong endpoint
   - Ensure the MCP server is running on port 8085
   - Check `MCP_SERVER_URL` (env var / `.env`, default `http://localhost:8085/mcp`)

2. **400/405/406 on a plain GET**: can occur because the endpoint expects MCP headers/session
   - Verify with an MCP client and an actual tool call
   - A status below 500 alone does not prove a working MCP session

3. **Timeout errors**: Server not started
   - Start the MCP server first, then the ADK client

### Fallback Mode

The fallback agent is created only if initialization inside the `try` block fails. `McpToolset` can connect lazily, so errors during tool discovery or calls may occur later and will not necessarily trigger fallback. Confirm that chat events show a successful tool call and weather data. Fix the server connection and restart ADK web if needed.

## Environment Variables

Create `.env` file:
```bash
GOOGLE_API_KEY=your_gemini_api_key
```

## Resources

- [Google ADK Documentation](https://google.github.io/adk-docs/)
- [MCP Specification](https://modelcontextprotocol.io/)
- [FastMCP GitHub](https://github.com/jlowin/fastmcp)
- [WeatherAPI](https://www.weatherapi.com/)
