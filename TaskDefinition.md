Task Definition
===============

    routing:          Routing keys specific to the task, try.test.crash...
                      (max 64 chars, as many dots as you would like)
    retries:          Number of times to try this task. 0 if it's not idempotent!
    priority:         Double, realative priority of the task
    deadline:         Datetime after which the task is resolved
    worker_type:      <worker-type> machine type needed
    payload:          JSON Object interpreted by worker
    metadata:         Formalized tags
      name:           'Build gecko on linux64'
      owner:          Email of person who caused this task
      scheduler:      'task-grapher-scheduler'
    tags:             Arbitrary tags
      branch:         try
      build:          'gecko'

Note that **workers** are fairly free to defined how they interpret payload.
Payload could be a JSON object like: `{docker_image, cmd, stdin_for_cmd}`,
queue and TaskCluster doesn't care. Just document it for people who end up using
a worker-type featuring your worker.
