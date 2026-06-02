# WSGI and ASGI Deep Dive

Standard interfaces that let Python web frameworks and servers be swapped freely — neither side needs to know what the other is built with.

## What problem they solve

- Before WSGI (PEP 333, 2003): each framework had bespoke server integrations — no way to swap Django onto a different server without rewriting glue code
- WSGI defines a single callable contract: any WSGI server + any WSGI app, fully swappable
- ASGI (2019) extends that contract to async + long-lived connections

## WSGI interface

```python
def app(environ, start_response):
    # environ: dict of request metadata (method, path, headers, body as file-like)
    # start_response: callback — call it to send status + response headers
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return [b'Hello World']   # iterable of bytes
```

- Synchronous — server blocks until app returns
- Request starts, response ends: that's the full lifecycle
- **Limitation**: no WebSockets (connection never "returns"), no SSE, no HTTP/2 streaming

## ASGI interface

```python
async def app(scope, receive, send):
    # scope:   connection metadata (type, path, headers)
    # receive: async callable — pull incoming events (body chunks, WS messages)
    # send:    async callable — push outgoing events (response headers, body, WS frames)

    await send({'type': 'http.response.start', 'status': 200, 'headers': [...]})
    await send({'type': 'http.response.body', 'body': b'Hello'})
```

- Async — models the full lifecycle of a WebSocket: one coroutine, events flowing in both directions, connection open as long as needed
- Supports: HTTP, WebSocket, SSE, HTTP/2, long-polling

## Gunicorn and Uvicorn are Python

- Both are plain Python packages — `pip install gunicorn` / `pip install uvicorn`
- CPython interpreter has no HTTP built in: executes bytecode, manages memory, knows nothing about sockets
- Gunicorn/Uvicorn use Python's `socket` stdlib to bind a port, accept TCP, parse HTTP, then call your app via the spec
- Node.js is different: its `http` module is baked into the runtime (C++ inside Node) — no equivalent layer needed

## Dev servers already do this

Frameworks ship a simple dev server built on Python's `socket` stdlib:

- Django: `manage.py runserver`
- Flask: `flask run`
- FastAPI: `uvicorn main:app` direct

These are single-process and intentionally not production-grade.

## What Gunicorn adds for production

- **Pre-fork workers**: spawns N OS processes; copy-on-write memory efficient
- **Crash recovery**: automatically restarts a crashed worker
- **Graceful restart**: drains in-flight requests before replacing workers on deploy
- **Timeout enforcement**: kills hung workers after threshold
- **Signal handling**: SIGTERM / SIGHUP / SIGWINCH handled correctly
- **PID files + logging**: process supervision tooling

Writing this reliably is hard — it's why every framework doesn't bundle its own.

## Common deployment patterns

| Scenario | Stack |
|---|---|
| Development | `manage.py runserver` or `uvicorn main:app` |
| Simple async prod | `uvicorn main:app --workers 4` |
| Django/Flask prod | Nginx → Gunicorn → Django |
| FastAPI prod | Nginx → Gunicorn (process mgr) → Uvicorn workers → FastAPI |

The Gunicorn + Uvicorn workers combo is common: Gunicorn owns process management, Uvicorn owns the async event loop inside each worker.
