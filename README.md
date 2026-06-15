# mcp-hub

A self-hosted MCP (Model Context Protocol) server registry and proxy, built with NestJS, PostgreSQL, Redis, and BullMQ.

> Register any MCP server once. Proxy tool calls through a single endpoint. Stream real-time logs over WebSocket.

## Why

The [official MCP registry](https://github.com/modelcontextprotocol/servers) is built in Go and cloud-hosted. **mcp-hub** is:

- **Self-hosted** — runs in your own infra, zero cloud dependency
- **TypeScript-native** — NestJS + strict mode, familiar for JS/TS teams
- **Proxy layer** — one endpoint for all your MCP servers with auth forwarding
- **Observable** — every tool call logged to PostgreSQL, streamed live via WebSocket
- **Health monitoring** — async health checks via BullMQ, per-server status

## Stack

- **NestJS** — API framework
- **PostgreSQL** — server registry + tool call logs
- **Redis** — WebSocket pub/sub + BullMQ transport
- **BullMQ** — async health check jobs
- **Socket.io** — real-time tool call log streaming

## Quick Start

```bash
git clone https://github.com/DIYA73/mcp-hub
cd mcp-hub
cp .env.example .env
docker compose up
```

API available at `http://localhost:3000/api/v1`

## API Reference

### Servers

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/servers` | Register a new MCP server |
| `GET` | `/api/v1/servers` | List all registered servers |
| `GET` | `/api/v1/servers/:id` | Get server by ID |
| `PATCH` | `/api/v1/servers/:id` | Update server config |
| `DELETE` | `/api/v1/servers/:id` | Remove server |

### Proxy

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/proxy/:serverId/tools` | List tools on a server |
| `POST` | `/api/v1/proxy/:serverId/call` | Call a tool on a server |

### Logs

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/logs` | All tool call logs |
| `GET` | `/api/v1/logs/server/:serverId` | Logs for a specific server |

### Health

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/health/status` | Registry health summary |
| `POST` | `/api/v1/health/check-all` | Queue health checks for all servers |
| `POST` | `/api/v1/health/check/:serverId` | Health check one server |

## WebSocket

Connect to `ws://localhost:3000/logs` (Socket.io namespace `/logs`).

```js
import { io } from 'socket.io-client';
const socket = io('http://localhost:3000/logs');
socket.on('tool-call', (log) => console.log(log));
socket.emit('subscribe-server', { serverId: 'your-server-id' });
```

## Register a Server

```bash
curl -X POST http://localhost:3000/api/v1/servers \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "my-github-mcp",
    "url": "http://localhost:8080",
    "transport": "http",
    "description": "GitHub MCP server"
  }'
```

## Call a Tool

```bash
curl -X POST http://localhost:3000/api/v1/proxy/{serverId}/call \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "create_issue",
    "input": { "repo": "DIYA73/mcp-hub", "title": "Bug report" }
  }'
```

## License

MIT
