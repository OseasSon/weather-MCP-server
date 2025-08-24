# Weather MCP Server for Intercom

Custom Model Context Protocol (MCP) server that integrates weather tools with Intercom for enhanced customer support.

## 🎯 For Solutions Engineers

This project demonstrates custom MCP server integration with Intercom, perfect for:
- **Product demos** showing MCP capabilities
- **Customer proof-of-concepts** for weather-related businesses
- **Technical demonstrations** of Intercom's extensibility

## 🚀 Quick Start

1. **Read the setup guide**: [`SETUP_GUIDE.md`](./SETUP_GUIDE.md)
2. **Start the server**: `uv run weather.py`

## 🛠️ What It Provides

### Weather Tools for Intercom Fin
- **`greet`** - Personalized customer greetings
- **`get_alerts`** - Real-time weather alerts for US states
- **`get_forecast`** - Detailed weather forecasts by coordinates

### Technical Features
- **Real-time data** from National Weather Service API
- **HTTP transport** via ngrok tunneling
- **Async operations** for optimal performance
- **Error handling** with graceful fallbacks

## 📋 Files Overview

- **`weather.py`** - Main MCP server implementation
- **`SETUP_GUIDE.md`** - Complete setup instructions for team
- **`pyproject.toml`** - Python project configuration

## ⚠️ Important Note

Due to [MCP SDK bug #423](https://github.com/modelcontextprotocol/python-sdk/issues/423), you may need to apply a simple patch to enable tool discovery. This bug may be fixed in future SDK releases. Full instructions in the setup guide.

## 🔧 Tech Stack

- **FastMCP** - MCP server framework
- **Python 3.11+** - Runtime environment
- **httpx** - HTTP client for weather API
- **uvicorn** - ASGI server
- **ngrok** - HTTP tunneling

---

**Created for Intercom Solutions Engineering Team**
**Status**: ✅ Working with Intercom integration
