Task Definition
===============

``` Javascript
{
  "version":            "0.2.0",
  "routing":          // Routing keys specific to the task, try.test.crash...
                      // (max 64 chars, as many dots as you would like)
  "retries":          // Number of times to try this task. 0 if it's not idempotent!
                      // (at most 999 higher numbers are **not** supported)
  "priority":         // Double, realative priority of the task
  "created":          // Creation time (ISO 8601)
  "deadline":         // Datetime after which the task is resolved (ISO 8601)
  "provisioner_id":   // Provisioner designated to spawn the worker
  "worker_type":      // Machine type needed
  "payload":          // JSON Object interpreted by worker
  "metadata":         // Formalized tags
    "name":           // Human readable name of task, useful for debugging
                      // for exmaple: `Build of gecko on linux64 for try <rev>`
    "owner":          // E-mail of person who caused this task, this is the
                      // person who did `hg push try`
    "maintainer":     // E-mail of person who automated this task and wrote the
                      // task definition or template it was generated from.
  "tags":             // Informal tags, basically any small useful key, value
                      // pairs it could be nice to have, for debugging,
                      // statistics or task execution time prediction.
    "branch":         // Could be useful if it applies
    "revision":       ...
    "description":    ...
}
```

Note that **workers** are fairly free to define how they interpret the payload.
The payload could be a JSON object like: `{docker_image, cmd, stdin_for_cmd}`,
queue doesn't care. Just document it for people who end up using a worker-type
featuring your worker.
