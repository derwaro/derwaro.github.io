---
title: "User Based Or Dynamic Celery Queues"
date: 2024-02-02T11:08:17-06:00
lastmod: 
draft: false
tags: ["howtow", "python", "celery", "rabbitmq", "fastapi"]
categories: ["tech"]
---

# Introduction
Did you know that RabbitMQ processes tasks based on First In - First Out? I guess so, but what if that does not fit your needs? 

Consider this example: User A sends 30.000 tasks to the universal queue "UberQueue" and then User B sends 500 tasks to the very same queue. RabbitMQ will receive all tasks alright and start sending them out to workers. However, tasks from User B will only be processed after all 30.000 tasks from User A have been processed.

This is the standard behavior for RabbitMQ and I guess a fair one for most situations. However I needed another solution, since I needed a fairer distribution of task-procession between users[^1]. 

It is possible to solve this problem staying in Python, namely in this case a FastAPI backend and RabbitMQ together with Celery as Task Queue.

## Basic Idea
RabbitMQ works as said on the FIFO principle. However between queues it applies the round-robin fashion to send tasks to workers. Now the idea to distribute task-processing more evenly is to create a queue for each user. This way each user can post how many tasks/messages as wanted to it's personal queue. RabbitMQ then *loops* over it's queues (round-robin) and starts sending out messages one after another (FIFO) to workers.

The result sums up to the following example:
We start out with 3 workers (Celery instances running as *worker*): W1, W2, W3. All workers are connected to the same RabbitMQ broker and are able to pick up tasks from all queues.
Queue-wise we have: Q_User1, Q_User2, Q_User3. None have any specific relation to any worker. They are each a "personal" queue for one of our users.
We (that is our users) will publish the following messages to RabbitMQ:
- M1, M2, M3 to Q_User1  
- M4, M5, M6 to Q_User2  
- M7, M8, M9 to Q_User3  

Now RabbitMQ should loop over it's queues (including those mentioned and all others) and send out messages to workers:

**First Loop:**  
W1 receives M1 from Q_User1  
W2 receives M4 from Q_User2  
W3 receives M7 from Q_User3  

**Second Loop:**  
W1 receives M2 from Q_User1  
W2 receives M5 from Q_User2  
W3 receives M8 from Q_User3  

**Third Loop:**  
W1 receives M3 from Q_User1  
W2 receives M6 from Q_User2  
W3 receives M9 from Q_User3  

This should equalize task procession pretty much. I guess it comes with it's drawbacks since each queue means more load for RabbitMQ, however that's a risk I have to take, since it guarantees timely procession for each user and non blocking of user by a heavy-usage-user or general high load.

There are several adaptions to an existing Celery setup to make this happen.

# General Project Structure

```
project
├── src
    ├── api/
    │   ├── __init__.py
    │   ├── my-api.py
│   ├── celery-config/
│   │   ├── celery-config.py
│   │   └── celery-utils.py
│   ├── celery-tasks/
│   │   └── tasks.py
│   ├── __init__.py
│   └── main.py
├── config.py
├── config.env
└── requirements.txt
```

This is just the general and very generic project structure. I won't go into much detail in respect to package version, since, but I will share with you the one's I see necessary and currently have in my `requirements.txt`:
```
celery==5.3.4
fastapi==0.103.1
flower==2.0.1
```

# The API
The API shown here is very reduced and just exemplary represented in `my-api.py`. This simply serves to show how to invoke Celery from an endpoint. It really is a pointless thing, I just make sure to have the user be logged in so that the user id/uuid can be used to create a unique queue/consumer. 

**./src/api/my-api.py**
```python {hl_lines=[23, 29]}
from fastapi import (
    APIRouter,
    HTTPException,
    Depends,
)
from src.users import current_active_user
from src.celery_tasks.tasks import my_task
from src.celery_config.celery_utils import add_consumer_helper


router = APIRouter(
    prefix="/user_sequence",
    tags=["user_sequence"],
    dependencies="",
    responses={404: {"description": "Not found"}},
)


@router.post("/user_sequence")
async def user_sequence(
    current_user: Depends(current_active_user),
):
    add_consumer_helper(f"usersequence_{current_user.id}")

    task = my_task.apply_async(
        args=[
            current_user,
        ],
        queue=f"usersequence_{current_user.id}",
    )
    return {
        "message": "initiated sequence",
        "task_id": task.id,
        "current_user": current_user.id
    }
```

The crucial points for this article lie in the marked lines. On line 23 I use a little helper function defined in `celery-utils.py` (more later on) to add a consumer with a matching unique name to Celery. Then on line 29, in the task creation I pass this very same unique string as `queue` to Celery. The unique string can be anything, but is should not be completely random, since having many queues weighs on memory and CPU of RabbitMQ. With those two lines we direct Celery to both create consumers and matching queues.

# Celery Configuration and Utils
In this section I'll show the specific Celery specific configuration and utilities I used to make sure that all tasks are handled from all queues.

Again, all files are somewhat reduced to represent the parts actually needed to reproduce this article's goals. Some things may be missing,... Contact me, if you need more details on something specific!

**`./src/celery-config/celeryconfig.py`**
``` python {hl_lines=["22-25",33, 42]} 
import os
from kombu import Queue
import requests


queue_list_from_rabbitmq = []


def route_task(name, args, kwargs, options, task=None, **kw):
    if ":" in name:
        queue, _ = name.split(":")
        return {"queue": queue}
    return {"queue": "celery"}


class BaseConfig:
    CELERY_BROKER_URL: str = os.environ.get(
        "CELERY_BROKER_URL", "amqp://guest:guest@localhost:5672/"
    )
    # request to rabbitmq api to obtain queue names
    try:
        response = requests.get("http://guest:guest@localhost:15672/api/queues")
        for q in response.json():
            if q["durable"] == True and q["name"][:7] == "usersequence_":
                queue_list_from_rabbitmq.append(q["name"])
    except:
        queue_list_from_rabbitmq = ["docker_was_not_running"]


    CELERY_RESULT_BACKEND: str = os.environ.get("CELERY_RESULT_BACKEND", "rpc://")

    # create all missing queues. i.e. dynamic queues.
    CELERY_CREATE_MISSING_QUEUES: bool = True

    CELERY_TASK_QUEUES: list = (
        # default queue
        Queue("celery"),
        # custom queue
        Queue("queue_one"),
        Queue("another_queue"),
        Queue("yet_another_queue"),
        *[Queue(q) for q in queue_list_from_rabbitmq],
    )

    CELERY_TASK_ROUTES = (route_task,)
```

This file formerly *merely* defined the used broker url/credentials and some hard coded queues, which where created in RabbitMQ and used by Celery workers. Through some adaptions it is possible to make this a bit more dynamic.
The query on lines 22 to 25 gets a list of all already existing queues from RabbitMQ and adds the necessary information to a list `queue_list_from_rabbitmq`. The list specifically filters out those queues that match our dynamic name pattern (here: starting with "usersequence_" and are defined as `durable`). Line 33 makes sure that missing queues are automatically created and added, when a task with the `queue=` argument is received and that queue does not yet exist. Line 42 picks up the list from earlier on and adds all those queues that already exists in RabbitMQ to Celery (that is to the Celery workers).

**./src/celery-config/celery-utils.py**
```python {hl_lines=["36-38"]}
from celery import current_app as current_celery_app
from celery.result import AsyncResult
from celery.app.control import Control

from .celery_config import settings


def create_celery():
    celery_app = current_celery_app
    celery_app.config_from_object(settings, namespace="CELERY")
    celery_app.conf.update(task_track_started=True)
    celery_app.conf.update(task_serializer="pickle")
    celery_app.conf.update(result_serializer="pickle")
    celery_app.conf.update(accept_content=["pickle", "json"])
    celery_app.conf.update(result_expires=200)
    celery_app.conf.update(result_persistent=True)
    celery_app.conf.update(worker_send_task_events=False)
    celery_app.conf.update(worker_prefetch_multiplier=1)

    return celery_app


def get_task_info(task_id):
    """
    return task info for the given task_id
    """
    task_result = AsyncResult(task_id)
    result = {
        "task_id": task_id,
        "task_status": task_result.status,
        "task_result": task_result.result,
    }
    return result


def add_consumer_helper(queue_name: str):
    control = Control(current_celery_app)
    control.add_consumer(queue=queue_name)
```

The first function in this file is some standard Celery creation function that is called, when the main app starts. The second is for observing the status of tasks and can be called from an endpoint.
The really interesting thing in this file is the third function `add_consumer_helper`. This is the function that is called invoked from the endpoint that enqueues the actual task and adds the the specified `queue_name` as consumer to Celery. By calling this helper function before actually creating the task, the queue is created and a consumer added to listen for messages on that queue.

# Tasks
Again this task file is very generic, since this article is not focused on *what* a task is accomplishing, but more on how and in what order tasks are processed and distributed from RabbitMQ to workers.

```python
from celery import shared_task
import time

@shared_task(
    bind=True,
    autoretry=(),
    retry_backoff=True,
    retry_kwargs={"max_retries": 1},
    name="user_mytask",
)
def my_task(
    self,
    current_user,
    some_input
):
    # this can be anything. Here it simulates a task that runs for 5 seconds.
    time.sleep(5)
    result = some_input * 2
    
    return {
        "result": result
    }
```

# Main file
Once again a rather generic main.py file. All that is done in here is the definition of the FastAPI routers, a little helper/status endpoint to check out tasks by their task id and the the instanciation of Celery.

```python {hl_lines=[30]}
from fastapi import FastAPI, APIRouter
from src.api import my-api
from src.celery_config.celery_utils import create_celery, get_task_info

base_router = APIRouter()

@base_router.get("/task/{task_id}")
async def get_task_status(task_id: str) -> dict:
    """
    Return the status of the submitted Task
    """
    return get_task_info(task_id)


def create_app() -> FastAPI:
    _app = FastAPI(
        title="my_fastapi_api",
        version="0.0.1",
        debug=True,
    )
    # set up celery
    _app.celery_app = create_celery()
    # include APIRouters
    _app.include_router(my_api.router, prefix="/api")
    _app.include_router(base_router, prefix="/api")
    return _app


app = create_app()
celery = app.celery_app
```

# Conclusion And Closing Thoughts
This article shows how to implement a more dynamic handling of messages using RabbitMQ as broker together with Celery workers without changing how RabbitMQ processes and advances between queues. This is approached through dynamic definition of queues and having Celery together with RabbitMQ create missing queues and automatically adding consumers that listen on those queues. The big advantage over having one general queue for all users is, as mentioned above that, RabbitMQ iterates over all queues and send tasks from each queue to workers. The big point against this approach is the higher memory and CPU consumption of having many queues. I still can't give any exact numbers on that metric. The fine detail would be to find the right unit of uniqueness for the queues, but this largely depends on the actual use case. I'm thinking about having a queue for each user (which would result in a very high number of queues), grouping users in some way as all users starting with letter a to c and so on, or have the users create their own groups with invitees and then those groups get a personal queue. A further improvement would be to implement a way to clear unused queues from RabbitMQ and therefore lighten the burden.


[^1]: There exists also another possible solution: Prioritizing Queues. But I don't see this as dynamic as I want it to be, i.e. all queues for users should have the same priority, simply their tasks have to be processed more or less at the same time.