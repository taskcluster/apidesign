AMQP Exchanges
==============

We have the following topic exchanges:

  Exchange                  | Message Occur When
  -------------------------:|---------------------------------------------------------
  `v1/queue:task-pending`   | A task becomes pending, by creation or timeout
  `v1/queue:task-running`   | A task is scheduled on a worker
  `v1/queue:task-completed` | A task is resolved a completed by a worker
  `v1/queue:task-failed`    | A task has failed (or is canceled)


All messages have the same **routing key format**, which is a dot (`.`)
separated list of identifiers, defined as follows:
  1. `task-id`
  2. `run-id` *
  3. `worker-group` *
  4. `worker-id` *
  5. `provisioner-id`
  6. `worker-type`
  7. `task.routing` (The task routing key may contain additional dots)

(* The special key `_` will be used for keys not available, for example
`run_id` for a `task_pending_v1` message).


Message Exchange Details
------------------------

### `queue:task-pending`
``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
}
```

### `queue:task-running`
``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
  "run_id":         // run-id that was just started
  "logs":           // URL to logs.json which will appear during run
  "worker_group":   // Worker group the worker belongs to
  "worker_id":      // Identifier of the worker that just started
}
```

### `queue:task-completed`
``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
  "result":         // URL to results.json from run that completed the task
  "logs":           // URL to logs.json from run that completed the task
  "run_id":         // Run id that completed the task
  "worker_id":      // worker id that completed the task
  "worker_group":   // worker group that completed the task
}
```

### `queue:task-failed`
Message signifies that a task has failed, either because it was completed before
it's deadline, all retries failed and workers stopped responding or the task was
canceled. The specific _reason_ is evident from that task status structure.

``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
  "run_id":         // Last run id of the task (highest run-id)
  "worker_id":      // Last worker-id that worked on the task
  "worker_group":   // Last worker-group that worked on the task
}
```

**Considerations, with. multiple simultanous runs**:
Technically, we can have more than one run at the same time, if a task was
reclaimed too late by a worker and the queue allowed the worker to reclaim it.
In practice, however, this won't happen very often. So routing this message to
the highest run-id makes sense. If we want cancellation support along with the
ability to race workers against each other on a taskcluster-queue level we
should consider adding a `queue:run-canceled` exchange. But it is probably
better to implement a race feature on a higher-level, i.e. submit two identical
tasks with different provisioner_id or worker_id, then cancel the slowest when
the first finishes.

Example Use Cases
-----------------

### An Observant Provisioner
A provisioner may bind the `task_pending_v1` and `task_running_v1` exchanges to
an exclusive queue, it would only bind messages matching `*.*.*.*.<provisioner-id>.#`,
where `<provisioner-id>` identifies the provisioner. This way the provisioner
can have an accurate estimate of the number of pending tasks, without having
to constantly query the queue.

### Long Pulling Worker
A worker that is idle may choose to listen to a queue bound to `task_pending_v1`
for topics matching `*.*.*.*.<provisioner-id>.<worker-type>.#`. This way the worker
doesn't have to poll the queue for work as soon as it has established that the
queue doesn't have any pending work. Naturally, the worker would have to poll
the queue initially.

### Worker w. Cancellation Support
A worker may bind the `task-failed` with for messages matching
`*.*.<worker-group>.<worker-id>.#` in order to get a message whenever a job it's
working on is canceled.

Notice, that if the worker sits on a multi-core machine where a number of
workers are running in parallel, then a super-worker process could manage a
group of workers identified as sub-processes on the machine. Hence, only one
binding to `*.*.<worker-group>.#` would be needed.

### Dependent Task Scheduler
Any entity scheduling dependent tasks might should listen for message
resolutions matching `*.*.*.*.*.*.<scheduler-key>.#`, where `<scheduler-key>` is
a unique routing key added to tasks by the scheduler.

### Task Statistics Monitor
Might listen for task resolutions, fetch `result.json` and do statistics from
about tasks based on the contents here.
