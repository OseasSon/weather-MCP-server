# Weather MCP Server Setup Guide
*Custom MCP Server for Intercom Integration*

## 🎯 Overview

This guide shows you how to create a custom MCP server that integrates weather tools with Intercom.

**What you'll build:**
- **greet** - Personalized greetings
- **get_alerts** - Weather alerts for US states
- **get_forecast** - Weather forecasts by coordinates

## 🔧 Prerequisites

- Python 3.11+
- ngrok account (free tier works)
- Intercom workspace with MCP connectors enabled

---

## 🚀 Step-by-Step Setup

### Step 1: Install UV Package Manager
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Step 2: Create Project
```bash
# Create a new directory for our project
uv init weather
cd weather

# Create virtual environment and activate it
uv venv
source .venv/bin/activate

# Install dependencies
uv add fastmcp httpx

# Create our server file
touch weather.py
```

### Step 3: Create the Weather MCP Server

Create `weather.py` with the following content:

```python
from typing import Any
import httpx
from fastmcp import FastMCP

# Initialize FastMCP server
mcp = FastMCP("weather")

# Constants
NWS_API_BASE = "https://api.weather.gov"
USER_AGENT = "weather-app/1.0"

async def make_nws_request(url: str) -> dict[str, Any] | None:
    """Make a request to the NWS API with proper error handling."""
    headers = {
        "User-Agent": USER_AGENT,
        "Accept": "application/geo+json"
    }
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, headers=headers, timeout=30.0)
            response.raise_for_status()
            return response.json()
        except Exception:
            return None

def format_alert(feature: dict) -> str:
    """Format an alert feature into a readable string."""
    props = feature["properties"]
    return f"""
Event: {props.get('event', 'Unknown')}
Area: {props.get('areaDesc', 'Unknown')}
Severity: {props.get('severity', 'Unknown')}
Description: {props.get('description', 'No description available')}
Instructions: {props.get('instruction', 'No specific instructions provided')}
"""

@mcp.tool()
async def greet(name: str) -> str:
    """Greet a person by name.

    Args:
        name: The name of the person to greet
    """
    await asyncio.sleep(0.01)
    return f"Hello, {name}!"

@mcp.tool()
async def get_alerts(state: str) -> str:
    """Get weather alerts for a US state.

    Args:
        state: Two-letter US state code (e.g. CA, NY)
    """
    url = f"{NWS_API_BASE}/alerts/active/area/{state}"
    data = await make_nws_request(url)

    if not data or "features" not in data:
        return "Unable to fetch alerts or no alerts found."

    if not data["features"]:
        return "No active alerts for this state."

    alerts = [format_alert(feature) for feature in data["features"]]
    return "\n---\n".join(alerts)

@mcp.tool()
async def get_forecast(latitude: float, longitude: float) -> str:
    """Get weather forecast for a location.

    Args:
        latitude: Latitude of the location
        longitude: Longitude of the location
    """
    # First get the forecast grid endpoint
    points_url = f"{NWS_API_BASE}/points/{latitude},{longitude}"
    points_data = await make_nws_request(points_url)

    if not points_data:
        return "Unable to fetch forecast data for this location."

    # Get the forecast URL from the points response
    forecast_url = points_data["properties"]["forecast"]
    forecast_data = await make_nws_request(forecast_url)

    if not forecast_data:
        return "Unable to fetch detailed forecast."

    # Format the periods into a readable forecast
    periods = forecast_data["properties"]["periods"]
    forecasts = []
    for period in periods[:5]:  # Only show next 5 periods
        forecast = f"""
{period['name']}:
Temperature: {period['temperature']}°{period['temperatureUnit']}
Wind: {period['windSpeed']} {period['windDirection']}
Forecast: {period['detailForecast']}
"""
        forecasts.append(forecast)

    return "\n---\n".join(forecasts)

@mcp.resource("resource://status")
def get_server_status() -> str:
    """Provides the current status of the Weather Server."""
    return "Weather server is online and ready."

if __name__ == "__main__":
    # Initialize and run the server
    mcp.run(transport='http')
```

---

## ⚠️ Critical Fix: MCP SDK Bug

**Current Issue**: There's a known bug in the MCP SDK that prevents tools from being discovered by Intercom. This bug may be fixed in future SDK releases, but currently requires a patch.

**If your tools don't show up in Intercom**, apply this temporary fix:

1. **Find the SDK file**:
```bash
find .venv -name "session.py" -path "*/mcp/server/*"
```

2. **Edit the file** (usually `.venv/lib/python3.11/site-packages/mcp/server/session.py`)

3. **Find lines ~165-166** and comment them out:
```python
# PATCH: Commented out to fix SDK bug #423
# if self._initialization_state != InitializationState.Initialized:
#     raise RuntimeError("Received request before initialization was complete")
pass  # Skip initialization check to fix SDK bug #423
```

**Note**: This patch may not be needed in future MCP SDK versions. Check [MCP SDK Issue #423](https://github.com/modelcontextprotocol/python-sdk/issues/423) for updates.

---

## 🌐 ngrok Setup

### Step 1: Start the MCP Server
```bash
uv run weather.py
```

You should see:
```
🖥️  Server name:     weather
📦 Transport:       Streamable-HTTP
🔗 Server URL:      http://127.0.0.1:8000/mcp
```

### Step 2: Create ngrok Tunnel
```bash
ngrok http 8000
```

Note the **public URL** (e.g., `https://abc123.ngrok.io`)

**Important**: Your MCP server runs on `/mcp` endpoint, so the full URL for Intercom will be:
```
https://your-ngrok-url.ngrok.io/mcp
```

---

## 🔗 Intercom Integration

### Step 1: Add MCP Server to Intercom

1. Go to **Settings > Data > Data connectors**
2. Click **"Add MCP server"**
3. Enter your ngrok URL with `/mcp` endpoint:
   ```
   https://your-ngrok-url.ngrok.io/mcp
   ```
4. Click **"Connect"**

### Step 2: Add Tools to Intercom

1. Find your weather server in **Data connectors**
2. Click **"+ New"** under your MCP server
3. Select the tools you want:
   - **greet** - Personalized greetings
   - **get_alerts** - Weather alerts for US states
   - **get_forecast** - Detailed weather forecasts

### Step 3: Test the Integration

Test each tool with these examples:
- **greet**: `{"name": "John"}`
- **get_alerts**: `{"state": "CA"}`
- **get_forecast**: `{"latitude": 37.7749, "longitude": -122.4194}`

---

## 🎪 Demo Usage

**Your weather MCP server enables Fin to:**
- Personalize greetings with customer names
- Provide real-time weather alerts for any US state
- Give detailed forecasts for specific locations

**Demo scenarios:**
- Customer asks about local weather → use `get_forecast`
- Customer mentions travel → check `get_alerts` for destination
- Personalized service → `greet` customers by name

---

## 🐛 Troubleshooting

**Tools not showing in Intercom?**
- Apply the SDK patch described above
- Check for "WARNING: Received request before initialization was complete" in logs

**Connection failed?**
- Verify ngrok is running and URL includes `/mcp`
- Ensure server is running on port 8000

**ngrok session expired?**
- Restart ngrok and update the URL in Intercom

**Success indicators:**
- Clean `200 OK` responses in server logs
- No initialization warnings
- Tools visible in Intercom's "Data connectors"

---

## 🎯 Next Steps

- Customize tools for specific demo scenarios
- Add more weather data sources (international APIs)
- Create industry-specific tools (retail, travel, events)
- Build complex workflows with multiple tool combinations

---

## 📚 References

- **FastMCP Installation Guide**: [https://gofastmcp.com/getting-started/installation](https://gofastmcp.com/getting-started/installation)
- **Official MCP Server Development**: [https://modelcontextprotocol.io/quickstart/server#importing-packages-and-setting-up-the-instance](https://modelcontextprotocol.io/quickstart/server#importing-packages-and-setting-up-the-instance)
- **ngrok Installation**: [https://ngrok.com/downloads/mac-os?tab=install](https://ngrok.com/downloads/mac-os?tab=install)
- **Intercom MCP Connectors**: [https://www.intercom.com/help/en/articles/11461635-powering-fin-with-your-external-tools-using-mcp-connectors#h_e42fe964f7](https://www.intercom.com/help/en/articles/11461635-powering-fin-with-your-external-tools-using-mcp-connectors#h_e42fe964f7)

---

**Created by**: Oséas Filho
**Created on**: August 2025
**MCP Version**: 1.13.1
**FastMCP Version**: 2.11.3
