# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WhatsApp MCP (Model Context Protocol) server that enables Claude to interact with personal WhatsApp accounts. It consists of two main components:

1. **Go WhatsApp Bridge** (`whatsapp-bridge/`): Connects to WhatsApp's web API using the whatsmeow library, handles authentication via QR code, and stores message history in SQLite
2. **Python MCP Server** (`whatsapp-mcp-server/`): Implements the MCP protocol to provide standardized tools for Claude to interact with WhatsApp data

## Architecture

- **Data Storage**: All messages stored in SQLite (`whatsapp-bridge/store/messages.db` and `whatsapp-bridge/store/whatsapp.db`) - these files are excluded from git to keep personal data local
- **API Communication**: Python MCP server communicates with Go bridge via HTTP API on localhost:8080
- **Authentication**: Uses WhatsApp Web's multidevice API with QR code authentication
- **Media Handling**: Supports downloading and sending images, videos, audio, and documents

## Development Commands

### Go WhatsApp Bridge
```bash
# Run the WhatsApp bridge (from whatsapp-bridge directory)
cd whatsapp-bridge
go run main.go

# For Windows (enable CGO)
go env -w CGO_ENABLED=1
go run main.go
```

### Python MCP Server
```bash
# Run the MCP server (from whatsapp-mcp-server directory)
cd whatsapp-mcp-server
uv run main.py
```

## Key Files and Structure

### Go Bridge (`whatsapp-bridge/`)
- `main.go`: Main application handling WhatsApp connection, message storage, and HTTP API
- `go.mod`: Dependencies including whatsmeow library and SQLite driver
- `store/`: SQLite databases for messages and WhatsApp session data

### Python MCP Server (`whatsapp-mcp-server/`)
- `main.py`: FastMCP server exposing WhatsApp tools to Claude
- `whatsapp.py`: Core WhatsApp operations (search, message handling, contacts)
- `audio.py`: Audio file processing utilities for voice messages
- `pyproject.toml`: Python dependencies including MCP and HTTP libraries

## Available MCP Tools

The Python server exposes these tools to Claude:
- `search_contacts`: Search contacts by name/phone
- `list_messages`: Retrieve messages with filters and context
- `list_chats`: List available chats with metadata
- `get_chat`: Get specific chat information
- `send_message`: Send messages to contacts/groups
- `send_file`: Send media files (images, videos, documents)
- `send_audio_message`: Send voice messages (requires .ogg opus format)
- `download_media`: Download media from messages

## Database Schema

### chats table
- `jid`: Chat identifier (PRIMARY KEY)
- `name`: Chat/contact name
- `last_message_time`: Timestamp of last message

### messages table
- `id`: Message ID
- `chat_jid`: Foreign key to chats table
- `sender`: Sender identifier
- `content`: Message text content
- `timestamp`: Message timestamp
- `is_from_me`: Boolean indicating if message was sent by user
- `media_type`: Type of media attachment
- Media metadata fields for file handling

## Dependencies

### Go Dependencies
- `go.mau.fi/whatsmeow`: WhatsApp Web API client
- `github.com/mattn/go-sqlite3`: SQLite driver
- `github.com/mdp/qrterminal`: QR code display

### Python Dependencies
- `mcp[cli]`: Model Context Protocol implementation
- `httpx`: HTTP client for API requests
- `requests`: HTTP requests library

## Authentication Flow

1. First run requires QR code scan with WhatsApp mobile app
2. Session data stored in `whatsapp-bridge/store/whatsapp.db`
3. Re-authentication may be required after ~20 days
4. Multiple device limit enforced by WhatsApp

## Media Processing

- **Audio**: Automatic conversion to .ogg opus format for voice messages (requires FFmpeg)
- **Images/Videos/Documents**: Direct file transfer via `send_file` tool
- **Download**: Media downloaded on-demand using `download_media` tool