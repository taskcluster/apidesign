Examples of a Non-Polling Worker
================================

Chart tool: http://sverweij.github.io/mscgen_js/

## Example without Worker Failure
```
# Example of a non-polling worker that doesn't fail
# But also doesn't reply on a persistent queue

Worker, RabbitMQ, Queue, Scheduler;

# Worker declares queue
Worker      =>  RabbitMQ  : "Declare, Bind Queue";
Worker      =>  Queue   : "/claim-work/...";
Queue       >>  Worker    : "No tasks pending";

|||;

# Scheduler sends a task to the queue
Scheduler   =>  Queue     : "Post Task";
Queue       =>  RabbitMQ  : "Task Pending";
RabbitMQ    =>  Worker    : "Pending Task";

# Worker claims task
Worker      =>  Queue     : "/task/.../claim";
Queue       >>  Worker    : "Task claimed";
Worker      >>  RabbitMQ  : "Ack Message";

|||;

# Worker completes the task
Worker box Worker         : "Computing";
Worker      =>  Queue     : "/task/.../complete";

# Queue informs RabbitMQ
Queue       =>  RabbitMQ  : "Task Complete";

...;
```

## Example w. Worker Crashing or Spot Instance Disappears

```
# Example of a non-polling worker that crashes
# But also doesn't reply on a persistent queue

Worker, RabbitMQ, Queue, Scheduler;

# Worker declares queue
Worker      =>  RabbitMQ  : "Declare, Bind Queue";
Worker      =>  Queue     : "/claim-work/...";
Queue       >>  Worker    : "No tasks pending";

|||;

# Scheduler sends a task to the queue
Scheduler   =>  Queue     : "Post Task";
Queue       =>  RabbitMQ  : "Task Pending";
RabbitMQ    =>  Worker    : "Pending Task";

# Worker claims task
Worker      =>  Queue     : "/task/.../claim";
Queue       >>  Worker    : "Task claimed";
Worker      >>  RabbitMQ  : "Ack Message";

|||;

# Worker completes the task
Worker box Worker         : "Crash!!!";
Queue box Queue           : "taken_until expired\ndecrement retries";
Queue       =>  RabbitMQ  : "Pending Task";

---: Life goes on as usual;

...;
```



# Non-polling Worker w. Crash Detection (Crash)
```
# Example of a non-polling worker crash detection

Worker : "Worker: w1", RabbitMQ, Provisioner, Queue;

Provisioner =>  RabbitMQ    : "Declare Y, Subscribe Y";

# Worker declares queue
Worker      =>  RabbitMQ    : "Declare, Subcribe X";
Worker      box RabbitMQ    : "Queue X deadletters to Y";

# Now before we Ack M1
Worker      =>  RabbitMQ    : "Post 'w1 crashed' to X";
RabbitMQ    =>  Worker      : "M1: 'w1 Crashed";


---: Pending task is posted to RabbitMQ;

# As usual worker is listening for pending tasks
RabbitMQ    =>  Worker      : "M2: Pending Task";
Worker      =>  Queue       : "/task/.../claim";
Queue       >>  Worker      : "Task claimed";
Worker      >>  RabbitMQ    : "Ack M2";

---: Now consider a Crash;


Worker box Worker : Crash;

RabbitMQ box RabbitMQ       : "M1 deadletters to Y";
RabbitMQ    =>  Provisioner : "M1: w1 crashed";
Provisioner =>  Queue       : "w1 crashed";
Queue box Queue             : "Clear taken_until for runs with w1";
Queue       >>  Provisioner : "200, OK";
Provisioner >>  RabbitMQ    : "Ack M1";

...;
```