AMQP Exchanges
==============

All exchanges are topic exchanges and messages are in the JSON format

    Exchange:       Routing Key
    task-created    <worker-type>.<task.routing>
    task-claimed    <worker-type>.<worker-id>.<run-id>.<task.routing>
    task-resolved   <state>.<worker-type>.<worker-id>.<run-id>.<task.routing>
                    , where state is 'completed' or 'failed'

Observe that `<worker-id>` contains exactly one dot, hence, when binding to a
topic you must specify this an extra `*`.


Example Use Cases
-----------------
_Most of these use-cases will not be implemented initially._

### An Observant Provisioner
A smart provisioner may bind the `task-created` and `task-claimed` exchanges to
an exclusive queue, it would only bind messages matching `<worker-type>.#`,
where `<ẁorker-type>` is any worker the provisioner can spawn. This way the
provisioner can have an accurate estimate of the number of pending tasks.

### Long Pulling Worker
A worker that is idle may choose to listen to a queue bound to `task-created`
for topics matching `<worker-type>.#`. This way the worker doesn't have to
poll the queue for work as soon as it has established that the queue doesn't
have any pending work.

### Worker w. Cancellation Support
A worker may bind the `task-resolved` with for messages matching
`chanceled.*.<worker-id>.#` in order to get a message whenever a job it's
working on is canceled.


### Dependent Task Scheduler
Any entity scheduling dependent tasks might should listen for message
resolutions matching `*.*.*.*.*.<scheduler-key>.#`, where `<scheduler-key>` is a
unique routing key added to tasks by the scheduler.


### Task Statistics Monitor
Might listen for task resolutions, fetch result artifacts and do statistics
based on these.
