# Middleware
Eventum ASGI provides a simple and flexible way to add middleware to your application. In the context of our framemwork Middleware is only used when handhake route is called. There some middlewares that are already included in the framework.

## Built-in Middleware

### ExceptionMiddleware

`ExceptionMiddleware` is a middleware that catches exceptions raised during the execution of the application and sends a response back to the client with the appropriate status code and message. Used when you purposely raise an `HttpException`.

### ServerErrorMiddleware

`ServerErrorMiddleware` signals that something went wrong from the server side. It also prints a traceback to the console.

## Custom Middleware
Middleware can be added to the application by using the `add_middleware` method. The middleware itself should inherit from the `MiddlewareClass` class which you can import from `eventum_asgi.middleware`.
Here is an example of a custom middleware and the minimum pattern you should follow:

```python
from eventum_asgi.middleware import MiddlewareClass

class CustomMiddleware(MiddlewareClass):
    def __init__(self, call_next: Union[CallNext[WSConnection], MiddlewareClass[WSConnection]],
                 *args: Any, **kwargs: Any) -> None:
        self.call_next = call_next

    async def __call__(self, connection: WSConnection) -> None:
        await self.call_next(connection)
```