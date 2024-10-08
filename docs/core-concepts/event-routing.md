# Event Routing

Event routing is a core concept of Eventum ASGI that allows you to route events to different handlers. It's based on JSON format and is used to get messages from a client. Each JSON client event should contain a 'event' field as one of the keys. You can use the JSON bellow as an example:

```json
{
    "event": "registered",
    "user_id": 1234
    "username": "John Doe"
    "email": "john.doe@example.com"
}
```

The sturcture of the JSON object is completely up to you, the only important thing is that the 'event' field must be present in the JSON object. But you can structure it in any way you want for example:

```json
{
    "event": "registered",
    "user": {
        "id": 1234,
        "username": "John Doe",
        "email": "john.doe@example.com"
    }
}
```

## Decorator

Similar to the handshake router, the decorator `event` is used to create a router for the events. It accepts a `event` and `validator` as parameters. The `event` parameter is a string that specifies the event that the client must send in JSON object, while the `validator` parameter should be an instance of `Event` class which itself inherits from Pydantic's `BaseModel` class. You can simply import the `Event` class like that `from eventum_asgi import Event`. `validator` parameter is optional.

```python
from eventum_asgi import Eventum, Event, WSConnection

app = Eventum()

class RegistrationValidator(Event):
    user_id: int
    username: str
    email: str

@app.handshake_route(path='/', required_headers=['X-Custom-Header', 'X-Another-Header'])
async def websocket_handler(connection: WSConnection):
    await connection.accept()

@app.event(event='registered', validator=Event)
async def registered_handler(connection: WSConnection, event: dict):
    await connection.send_text({'message': 'Welcome to the server!'})
```

## Function

`add_event` function works the same way as the decorator `event` but it accepts a `handler` parameter that is a coroutine function that will be called when the event is received.

```python
from eventum_asgi import Eventum, Event, WSConnection

app = Eventum()

class RegistrationValidator(Event):
    user_id: int
    username: str
    email: str

def registered_handler(connection: WSConnection, event: dict):
    await connection.send_text({'message': 'Welcome to the server!'})

app.add_event(event='registered', validator=Event, handler=registered_handler)
```