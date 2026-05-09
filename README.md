# MCP Research Server

MCP server for searching arXiv papers and exposing the results through SSE.

## Local run

Install dependencies:

```powershell
uv sync
```

Run the server with the default local port `8001`:

```powershell
uv run python research_server.py
```

Expected local log:

```text
INFO:     Started server process [23596]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8001 (Press CTRL+C to quit)
INFO:     127.0.0.1:61624 - "GET /sse HTTP/1.1" 200 OK
```

By default the server listens on:

```text
http://127.0.0.1:8001
{
"status": "ok",
"transport": "sse"
}

http://localhost:8001/sse
event: endpoint
data: /messages/?session_id=ccaf7ca039914debacab75a7d8922c96
: ping - 2026-05-09 13:31:53.656777+00:00
: ping - 2026-05-09 13:32:08.673246+00:00
: ping - 2026-05-09 13:32:23.702054+00:00
: ping - 2026-05-09 13:32:38.712254+00:00
```

Useful endpoints:

```text
GET /        - health check
GET /healthz - health check
GET /sse    - MCP SSE endpoint
```

To use a different local port, set `PORT` before running:

```powershell
$env:PORT = "8765"
uv run python research_server.py
```

With this override, the server listens on `http://0.0.0.0:8765`.

## Render.com deployment

Create a new Render Web Service from this repository.

Recommended settings:

```text
Environment: Python
Build Command: pip install -r requirements.txt
Start Command: python research_server.py
```

Render provides the `PORT` environment variable automatically. The server reads
that value and binds to `0.0.0.0`, which is required for Render port detection.

Expected deploy log:

```text
Uvicorn running on http://0.0.0.0:<PORT>
```

`0.0.0.0:<PORT>` is the internal address used inside the Render container.
Render exposes it through the public service URL.

Public Render endpoints:

```text
https://mcp-project-7xzc.onrender.com/
{
"status": "ok",
"transport": "sse"
}

https://mcp-project-7xzc.onrender.com/healthz
{
"status": "ok"
}

https://mcp-project-7xzc.onrender.com/sse
event: endpoint
data: /messages/?session_id=7f29ec0b5ee14d89be2040e49ad097a2
: ping - 2026-05-09 11:29:53.402607+00:00
: ping - 2026-05-09 11:30:08.403472+00:00
: ping - 2026-05-09 11:30:23.404598+00:00
```

Endpoint details:

```text
GET /        - root health endpoint. Returns 200 OK with basic service status.
GET /healthz - explicit health check endpoint. Returns 200 OK when the service is running.
GET /sse    - MCP SSE endpoint used by MCP clients to connect to the server.
```

The `/` and `/healthz` endpoints are regular HTTP endpoints intended for
browser checks, Render health checks, and quick availability tests. The `/sse`
endpoint is long-lived and is intended for MCP clients, not as a normal browser
page.
