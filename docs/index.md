<p align="center">
  <img width="320" height="320" src="https://raw.githubusercontent.com/tomchristie/uvicorn/master/docs/uvicorn.png" alt='uvicorn'>
</p>

<p align="center">
<em>The lightning-fast ASGI server.</em>
</p>

<p align="center">
<a href="https://github.com/encode/uvicorn/actions">
    <img src="https://github.com/encode/uvicorn/workflows/Test%20Suite/badge.svg" alt="Test Suite">
</a>
<a href="https://pypi.org/project/uvicorn/">
    <img src="https://badge.fury.io/py/uvicorn.svg" alt="Package version">
</a>
</p>

---

# Introduction

Uvicorn is a lightning-fast ASGI server implementation, using [uvloop][uvloop] and [httptools][httptools].

Until recently Python has lacked a minimal low-level server/application interface for
asyncio frameworks. The [ASGI specification][asgi] fills this gap, and means we're now able to
start building a common set of tooling usable across all asyncio frameworks.

ASGI should help enable an ecosystem of Python web frameworks that are highly competitive against Node
and Go in terms of achieving high throughput in IO-bound contexts. It also provides support for HTTP/2 and
WebSockets, which cannot be handled by WSGI.

Uvicorn currently supports HTTP/1.1 and WebSockets. Support for HTTP/2 is planned.

## Quickstart

Install using `pip`:

```shell
$ pip install uvicorn
```

This will install uvicorn with minimal (pure Python) dependencies.

```shell
$ pip install uvicorn[standard]
```

This will install uvicorn with "Cython-based" dependencies (where possible) and other "optional extras".

In this context, "Cython-based" means the following:

- the event loop `uvloop` will be installed and used if possible.
- the http protocol will be handled by `httptools` if possible.

Moreover, "optional extras" means that:

- the websocket protocol will be handled by `websockets` (should you want to use `wsproto` you'd need to install it manually) if possible.
- the `--reloader` flag in development mode will use `watchgod`.
- windows users will have `colorama` installed for the colored logs.
- `python-dotenv` will be installed should you want to use the `--env-file` option.
- `PyYAML` will be installed to allow you to provide a `.yaml` file to `--log-config`, if desired.

Create an application, in `example.py`:

```python
async def app(scope, receive, send):
    assert scope['type'] == 'http'

    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ],
    })
    await send({
        'type': 'http.response.body',
        'body': b'Hello, world!',
    })
```

Run the server:

```shell
$ uvicorn example:app
```

---

## Usage

The uvicorn command line tool is the easiest way to run your application...

### Command line options

```
Usage: uvicorn [OPTIONS] APP

Options:
  --host TEXT                     Bind socket to this host.  [default:
                                  127.0.0.1]
  --port INTEGER                  Bind socket to this port.  [default: 8000]
  --uds TEXT                      Bind to a UNIX domain socket.
  --fd INTEGER                    Bind to socket from this file descriptor.
  --reload                        Enable auto-reload.
  --reload-dir TEXT               Set reload directories explicitly, instead
                                  of using the current working directory.
  --workers INTEGER               Number of worker processes. Defaults to the
                                  $WEB_CONCURRENCY environment variable if
                                  available. Not valid with --reload.
  --loop [auto|asyncio|uvloop]
                                  Event loop implementation.  [default: auto]
  --http [auto|h11|httptools]     HTTP protocol implementation.  [default:
                                  auto]
  --ws [auto|none|websockets|wsproto]
                                  WebSocket protocol implementation.
                                  [default: auto]
  --lifespan [auto|on|off]        Lifespan implementation.  [default: auto]
  --interface [auto|asgi3|asgi2|wsgi]
                                  Select ASGI3, ASGI2, or WSGI as the
                                  application interface.  [default: auto]
  --env-file PATH                 Environment configuration file.
  --log-config PATH               Logging configuration file.
                                  Supported formats (.ini, .json, .yaml)
  --log-level [critical|error|warning|info|debug|trace]
                                  Log level. [default: info]
  --access-log / --no-access-log  Enable/Disable access log.
  --use-colors / --no-use-colors  Enable/Disable colorized logging.
  --proxy-headers / --no-proxy-headers
                                  Enable/Disable X-Forwarded-Proto,
                                  X-Forwarded-For, X-Forwarded-Port to
                                  populate remote address info.
  --forwarded-allow-ips TEXT      Comma separated list of IPs to trust with
                                  proxy headers. Defaults to the
                                  $FORWARDED_ALLOW_IPS environment variable if
                                  available, or '127.0.0.1'.
  --root-path TEXT                Set the ASGI 'root_path' for applications
                                  submounted below a given URL path.
  --limit-concurrency INTEGER     Maximum number of concurrent connections or
                                  tasks to allow, before issuing HTTP 503
                                  responses.
  --backlog INTEGER               Maximum number of connections to hold in
                                  backlog
  --limit-max-requests INTEGER    Maximum number of requests to service before
                                  terminating the process.
  --timeout-keep-alive INTEGER    Close Keep-Alive connections if no new data
                                  is received within this timeout.  [default:
                                  5]
  --ssl-keyfile TEXT              SSL key file
  --ssl-certfile TEXT             SSL certificate file
  --ssl-version INTEGER           SSL version to use (see stdlib ssl module's)
                                  [default: 2]
  --ssl-cert-reqs INTEGER         Whether client certificate is required (see
                                  stdlib ssl module's)  [default: 0]
  --ssl-ca-certs TEXT             CA certificates file
  --ssl-ciphers TEXT              Ciphers to use (see stdlib ssl module's)
                                  [default: TLSv1]
  --header TEXT                   Specify custom default HTTP response headers
                                  as a Name:Value pair
  --help                          Show this message and exit.
```

For more information, see the [settings documentation](settings.md).

### Running programmatically

To run uvicorn directly from your application...

**example.py**:

```python
import uvicorn

async def app(scope, receive, send):
    ...

if __name__ == "__main__":
    uvicorn.run("example:app", host="127.0.0.1", port=5000, log_level="info")
```

### Running with Gunicorn

[Gunicorn][gunicorn] is a mature, fully featured server and process manager.

Uvicorn includes a Gunicorn worker class allowing you to run ASGI applications,
with all of Uvicorn's performance benefits, while also giving you Gunicorn's
fully-featured process management.

This allows you to increase or decrease the number of worker processes on the
fly, restart worker processes gracefully, or perform server upgrades without downtime.

For production deployments we recommend using gunicorn with the uvicorn worker class.

```
gunicorn example:app -w 4 -k uvicorn.workers.UvicornWorker
```

For a [PyPy][pypy] compatible configuration use `uvicorn.workers.UvicornH11Worker`.

For more information, see the [deployment documentation](deployment.md).

## The ASGI interface

Uvicorn uses the [ASGI specification][asgi] for interacting with an application.

The application should expose an async callable which takes three arguments:

* `scope` - A dictionary containing information about the incoming connection.
* `receive` - A channel on which to receive incoming messages from the server.
* `send` - A channel on which to send outgoing messages to the server.

Two common patterns you might use are either function-based applications:

```python
async def app(scope, receive, send):
    assert scope['type'] == 'http'
    ...
```

Or instance-based applications:

```python
class App:
    async def __call__(self, scope, receive, send):
        assert scope['type'] == 'http'
        ...

app = App()
```

It's good practice for applications to raise an exception on scope types
that they do not handle.

The content of the `scope` argument, and the messages expected by `receive` and `send` depend on the protocol being used.

The format for HTTP messages is described in the [ASGI HTTP Message format][asgi-http].

### HTTP Scope

An incoming HTTP request might have a connection `scope` like this:

```python
{
    'type': 'http.request',
    'scheme': 'http',
    'root_path': '',
    'server': ('127.0.0.1', 8000),
    'http_version': '1.1',
    'method': 'GET',
    'path': '/',
    'headers': [
        [b'host', b'127.0.0.1:8000'],
        [b'user-agent', b'curl/7.51.0'],
        [b'accept', b'*/*']
    ]
}
```

### HTTP Messages

The instance coroutine communicates back to the server by sending messages to the `send` coroutine.

```python
await send({
    'type': 'http.response.start',
    'status': 200,
    'headers': [
        [b'content-type', b'text/plain'],
    ]
})
await send({
    'type': 'http.response.body',
    'body': b'Hello, world!',
})
```

### Requests & responses

Here's an example that displays the method and path used in the incoming request:

```python
async def app(scope, receive, send):
    """
    Echo the method and path back in an HTTP response.
    """
    assert scope['type'] == 'http'

    body = f'Received {scope["method"]} request to {scope["path"]}'
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ]
    })
    await send({
        'type': 'http.response.body',
        'body': body.encode('utf-8'),
    })
```

### Reading the request body

You can stream the request body without blocking the asyncio task pool,
by fetching messages from the `receive` coroutine.

```python
async def read_body(receive):
    """
    Read and return the entire body from an incoming ASGI message.
    """
    body = b''
    more_body = True

    while more_body:
        message = await receive()
        body += message.get('body', b'')
        more_body = message.get('more_body', False)

    return body


async def app(scope, receive, send):
    """
    Echo the request body back in an HTTP response.
    """
    body = await read_body(receive)
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ]
    })
    await send({
        'type': 'http.response.body',
        'body': body,
    })
```

### Streaming responses

You can stream responses by sending multiple `http.response.body` messages to
the `send` coroutine.

```python
import asyncio


async def app(scope, receive, send):
    """
    Send a slowly streaming HTTP response back to the client.
    """
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ]
    })
    for chunk in [b'Hello', b', ', b'world!']:
        await send({
            'type': 'http.response.body',
            'body': chunk,
            'more_body': True
        })
        await asyncio.sleep(1)
    await send({
        'type': 'http.response.body',
        'body': b'',
    })
```

---

## Alternative ASGI servers

### Daphne

The first ASGI server implementation, originally developed to power Django Channels, is [the Daphne webserver][daphne].

It is run widely in production, and supports HTTP/1.1, HTTP/2, and WebSockets.

Any of the example applications given here can equally well be run using `daphne` instead.

```
$ pip install daphne
$ daphne app:App
```

### Hypercorn

[Hypercorn][hypercorn] was initially part of the Quart web framework, before
being separated out into a standalone ASGI server.

Hypercorn supports HTTP/1.1, HTTP/2, and WebSockets.

```
$ pip install hypercorn
$ hypercorn app:App
```

---

## ASGI frameworks

You can use Uvicorn, Daphne, or Hypercorn to run any ASGI framework.

For small services you can also write ASGI applications directly.

### Starlette

[Starlette](https://github.com/encode/starlette) is a lightweight ASGI framework/toolkit.

It is ideal for building high performance asyncio services, and supports both HTTP and WebSockets.

### Django Channels

The ASGI specification was originally designed for use with [Django Channels](https://channels.readthedocs.io/en/latest/).

Channels is a little different to other ASGI frameworks in that it provides
an asynchronous frontend onto a threaded-framework backend. It allows Django
to support WebSockets, background tasks, and long-running connections,
with application code still running in a standard threaded context.

### Quart

[Quart](https://pgjones.gitlab.io/quart/) is a Flask-like ASGI web framework.

### FastAPI

[**FastAPI**](https://github.com/tiangolo/fastapi) is an API framework based on **Starlette** and **Pydantic**, heavily inspired by previous server versions of **APIStar**.

You write your API function parameters with Python 3.6+ type declarations and get automatic data conversion, data validation, OpenAPI schemas (with JSON Schemas) and interactive API documentation UIs.


[uvloop]: https://github.com/MagicStack/uvloop
[httptools]: https://github.com/MagicStack/httptools
[gunicorn]: http://gunicorn.org/
[pypy]: https://pypy.org/
[asgi]: https://asgi.readthedocs.io/en/latest/
[asgi-http]: https://asgi.readthedocs.io/en/latest/specs/www.html
[daphne]: https://github.com/django/daphne
[hypercorn]: https://gitlab.com/pgjones/hypercorn
