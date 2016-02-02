Examples of a Polling Worker
============================

Chart tool: http://sverweij.github.io/mscgen_js/

## Simple Failure Free Interaction
_This is the common case._

```
# Example of polling worker that doesn't fail

Worker, RabbitMQ, Queue, Scheduler;

# Scheduler sends a task to the queue
Scheduler   =>  Queue     : "Post Task";
Queue       =>  RabbitMQ  : "Task Pending";

# Worker polls for work
Worker      =>  Queue     : "/claim-work/...";
Queue       >>  Worker    : "Task";

# Worker completes the task
Worker box Worker         : "Computing";
Worker      =>  Queue     : "/task/.../complete";

# Queue informs RabbitMQ
Queue       =>  RabbitMQ  : "Task Complete";

...;
```






