# WebSocket Connections
Eventum ASGI provides a simple way to handle WebSocket connections throght the `WSConnection` class. This class is responsible for managing the WebSocket connection and provides methods for sending and receiving data.

## Accepting a Connection
To accept the WebSocket connection, you need to call the `accept` method on the `WSConnection` instance. This method will initiate the WebSocket handshake and send the handshake response to the client.

```python
@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
    await connection.accept()
```

### Extra Headers
Additionally, you can pass an 'extra_headers' parameter to the `accept` method to add additional headers to the handshake response. You can either pass a dictionary or a 'Headers' class instance.

#### Headers as a dictionary
```python
@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
    await connection.accept(extra_headers={'X-Custom-Header': 'value'})
```

#### Headers as a Headers class instance
```python
from eventum_asgi.models import Headers
@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
    headers = Headers({'X-Custom-Header': 'value'})
    await connection.accept(extra_headers=headers)
```

### Subprotocols factory
You can also pass a 'subprotocols_factory' function as a parameter to the `accept` method to specify a function that will be handle the subprotocols choice. Function should return a single value as a string and accepts a subprotocols list as a parameter. 

```python
def subprotocols_factory(subprotocols: list) -> str:
    return subprotocols[0]

@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
    await connection.accept(subprotocols_factory=subprotocols_factory)
```

## Sending Data
Before diving futher into how to send and receive data it's important to learn the philosophy behind how Eventum ASGI handles WebSocket connections. The framework is maibly based on the concept of event-based routing. This means that most of the time you won't use methods like `receive_text`, `receive_bytes` or `receive_data` after you accepted a connection it gets into a loop and waits for an event to occur. Instead, you'll use the `send_text`, `send_bytes` methods to send data to the client directly from the event handlers.

### Sending Text
To send text data, you can use the `send_text` method. This method accepts a string as a parameter and sends it to the client.

```python
@app.event('user_registered')
async def message_handler(connection: WSConnection, event: dict):
    await connection.send_text('{"event": "user_registered", "status": "success"}')
```
### Sending Bytes
To send bytes data, you can use the `send_bytes` method similarly to the `send_text` method. This method accepts a bytes-like object as a parameter and sends it to the client.

```python
@app.event('user_registered')
async def message_handler(connection: WSConnection, event: dict):
    await connection.send_bytes(b'{"event": "user_registered", "status": "success"}')
```

!!! note
    When you receive an event you can access the json data using the `event` parameter.

### Sending HTTP Response
Sometimes you may need to send an HTTP response to the client. You can use the `send_http_response` method to do this. This method accepts a `HttpResponse` object as a parameter and sends it to the client.

```python
from eventum_asgi.http_eventum import HttpResponse

@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
    await connection.accept()
    response = HttpResponse(status_code=200, headers={'Content-Type': 'text/plain'}, body='Hello, World!')
    await connection.send_http_response(response)
```

## Closing Connections
Closing a connection is a crucial aspect of handling WebSocket connections. When client disconects, eventum-asgi will automatically close the connection. You can also close a connection manually using the `close` method. This method accepts an optional `code` parameter that specifies the close code, and an optional `reason` parameter that specifies the close reason.

```python
from eventum_asgi import WSConnection, Eventum

app = Eventum()

@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
    await connection.accept()
    await connection.close(code=3000, reason='Test disconnect')
```

## Flags
Flags is another core concept of `WSConnection` class. It's a dictionary for you to store any data you need to keep track of. You can use flags to store information about the connection, such as the user's session ID or any other data you need to access later. Can be useful for implementing features like authentication or authorization.

### Adding Flags
You can set flags using the `add_flag` or 'add_flags' methods. The `add_flag` method accepts `name` and `value` as a parameters, while the `add_flags` method accepts a dictionary as a parameter.

### Removing Flags
You can remove flags using the `remove_flag` or 'remove_flags' methods. The `remove_flag` method accepts `name` as a parameter, while the `remove_flags` method accepts a list of names as a parameter. There's also an option to clear all the flags using the `clear_flags` method.

### Getting Flags
You can get flags using the `get_flag` or 'flags' methods. The `get_flag` method accepts `name` as a parameter and returns a specific flag, while the `flags` method returns a dictionary of all the flags.

### Flags Usage Example
```python
@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
    await connection.accept()
    connection.add_flag('session_id', '123')
    connection.add_flags({'some_data1': 'just_data1', 'some_data2': 'just_data2', 'some_data3': 'just_data3'})

@app.event('profile_updated')
async def profile_handler(connection: WSConnection, event: dict):
    if get_flag('session_id') == '123':
        await connection.send_text('{"event": "profile_updated", "status": "success"}')
        connection.remove_flag('some_data1')
        print(connection.flags)
    elif get_flag('session_id') == '456':
        connection.clear_flags()
    else:
        connection.remove_flags(['some_data1', 'some_data2'])
```   