> This is a course by Test Driven Development. Here are some of my notes on the course which I've put together

# Introduction

Celery is a background task queue while FastAPI is a server. Celery helps to offload computationally intensive tasks to the backend so that responses on the user's end remains fast , snappy and within a reasonable time frame.

Celery provides a few advantages over Fast API's default `Background Tasks` module. 

1. It runs on a separate thread and event loop. Therefore it doesn't take away from your allocated bandwidth for your server
2. It provides a host of different benefits such as automatic retries, job monitoring and chaining of events

We use the Celery client as a producer to add new tasks to the queue via a message broker. This can be either [[Redis]] or [[RabbitMQ]]. Celery workers then consume new tasks from the queue, again, via the message broker. Once processed, results are then stored in the result backend.

![[Pasted image 20230828202729.png | 500]]

## Utilising Celery

## Celery Worker and Producer

We need to initialise 2 things here

- A Celery Client - This is a producer which will be creating new tasks 
- A Celery Worker - This is a long running process which will be monitoring the message broker ( in this case Redis or RabbitMQ ) for new tasks and then executing them

We can initialise a celery worker with the command

```
celery -A main.celery worker --loglevel=info
```

Where `main.celery` indicates where the tasks are deployed within. 

We can also initialise a new celery client as

```
from celery import Celery
celery = Celery(

__name__, broker="redis://127.0.0.1:6379/0", backend="redis://127.0.0.1:6379/0"

)
```
## Flower

We can initiate a flower UI instance with the command

```
celery -A main.celery flower --port=5555
```

This will boot up a UI which we can use to view subsequent requests for


# Debugging

When debugging Celery worker tasks, we can utilise the `rdb` instance. We can utilise it by simply running

```
@shared_task
def divide(x, y):
    from celery.contrib import rdb
    rdb.set_trace()

    import time
    time.sleep(5)
    return x / y
```

We can then use telnet inside the docker container in order to access the rdb instance

```bash
docker-compose exec <worker docker container> bash
(container)$ telnet 127.0.0.1 6901
```

