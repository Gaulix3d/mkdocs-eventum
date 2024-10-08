# HTTP Exceptions
Sometimes you may need to send an HTTP exception to the client. You can use the `HttpException` class to do this. You can import this class from `eventum_asgi.exceptions`.
Here's an example of how to use it:

```python
from eventum_asgi.exceptions import HttpException
from eventum_asgi.models import Headers
@app.handshake_route('/')
async def websocket_handler(connection: WSConnection):
   raise HttpException(status_code=400, headers=Headers({'X-Custom-Header': 'value'}), body=b'Bad Request')
```
## Built-in HTTP Exceptions
Some of the these exceptions are built-in and app raises them automatically.

### Not Found
`HttpNotFoundException` exception is raised when the requested resource is not found it sends `404 Not Found`. If client makes a request to a non-existent resource, the server should respond with this exception. The connection will be closed after sending the response. 

### Headers Missing
`RequiredHeadersMissingException` exception is raised when the client sends a request without the required headers when you specified them in the `required_headers` parameter of the `handshake_route` decorator or `add_handshake_route` method. The connection will be closed after sending the response. It sends `400 Bad Request` HTTP status code.

