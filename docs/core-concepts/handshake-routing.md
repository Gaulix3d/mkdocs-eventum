# Handshake Routing

Handshake routing is yet another core concept of Eventum ASGI as well as a starting point for each connection. To create a router you can either use a decorator `handshake_route` or a function 
`add_handshake_route`. You can access both from the `Eventum` class e.g. `Eventum.handshake_route` or `Eventum.add_handshake_route`. 

## Decorator
The decorator `handshake_route` is used to create a router for the handshake request. It accepts a `path` as a parameter and `required_headers` as an optional parameter that specifies the headers that the client must send in the handshake request, it has to be a list of strings.

```python
from eventum_asgi import Eventum
from eventum_asgi.models import WSConnection
app = Eventum()

@app.handshake_route(path='/', required_headers=['X-Custom-Header', 'X-Another-Header'])
async def websocket_handler(connection: WSConnection):
    await connection.accept()
```
## Function
The function `add_handshake_route` is used to create a router for the handshake request. It accepts a `path` as a parameter and `required_headers` as an optional parameter that specifies the headers that the client must send in the handshake request, it has to be a list of strings. Additionally, it accepts a `handler` parameter that is a coroutine function that will be called when the handshake request is received.

```python
from eventum_asgi import Eventum
from eventum_asgi.models import WSConnection
app = Eventum()

def websocket_handler(connection: WSConnection):
    await connection.accept()

app.add_handshake_route(path='/', required_headers=['X-Custom-Header', 'X-Another-Header'], handler=websocket_handler)
```
!!! note
    Don't forget that the function you are passing to `add_handshake_route` or decorating with `handshake_route` 
    must be a coroutine function and must contain the `connection` parameter which is of type `WSConnection`.

