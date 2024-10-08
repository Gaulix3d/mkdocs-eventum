# Lifespan Events
Lifespan events are aplication level events which you can trigger `on_startup` and `on_shutdown`. These events are useful for performing tasks like starting and stopping background tasks, initializing databases, or sending notifications to external services.

## Starup event
The `on_startup` event is triggered when the application is starting. You can simply create it this way:

```python
app = Eventum()

@app.lifespan_event('on_startup')
async def startup():
    print('Application is starting...')
```

## Shutdown event
The `on_shutdown` event is triggered when the application is shutting down. Here's an example of how to use it:

```python
app = Eventum()

@app.lifespan_event('on_shutdown')
async def shutdown():
    print('Application is shutting down...')
```