AMQP Exchanges
==============

All exchanges are topic exchanges and we have the following.

  Exchange            | Message Occur When
  -------------------:|---------------------------------------------------------
  `task_pending_v1`   | Whenever a task becomes pending, by creation or timeout
  `task_running_v1`   | Whenever a task is scheduled on a worker
  `task_completed_v1` | Whenever a task is resolved a completed by a worker
  `task_failed_v1`    | Whenever a task has failed


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

### `task_pending_v1`
``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
}
```

### `task_running_v1`
``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
  "run_id":         // run-id that was just started
  "logs":           // URL to logs.json which will appear during run
  "worker-group":   // Worker group the worker belongs to
  "worker-id":      // Identifier of the worker that just started
}
```

### `task_completed_v1`
``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
  "result":         // URL to results.json from run that completed the task
  "logs":           // URL to logs.json from run that completed the task
}
```

### `task_failed_v1`
Message signifies that a task has failed, either because it was completed before
it's deadline, all retries failed and workers stopped responding or the task was
canceled. The specific _reason_ is evident from that task status structure.

``` Javascript
{
  "version":        "0.2.0",
  "status":         // Task status structure
}
```

Example Use Cases
-----------------
_Most of these use-cases will not be implemented initially._

### An Observant Provisioner
A provisioner may bind the `task_pending_v1` and `task_running_v1` exchanges to
an exclusive queue, it would only bind messages matching `<provisioner-id>.#`,
where `<provisioner-id>` identifies the provisioner. This way the provisioner
can have an accurate estimate of the number of pending tasks, without having
to constantly query the queue.

### Long Pulling Worker
A worker that is idle may choose to listen to a queue bound to `task_pending_v1`
for topics matching `<provisioner-id>.<worker-type>.#`. This way the worker
doesn't have to poll the queue for work as soon as it has established that the
queue doesn't have any pending work.

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
