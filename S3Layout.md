S3 Storage Layout
=================

We need a bucket full of truth, for now let's used `jonasfj-taskcluster-tasks`,
layout will be as follows:

    s3://.../<task-id>/task.json
    s3://.../<task-id>/resolution.json
    s3://.../<task-id>/runs/<run-id>/logs.json
    s3://.../<task-id>/runs/<run-id>/result.json
    s3://.../<task-id>/runs/<run-id>/artifacts/...

File Details
------------

### `/<task-id>/task.json`
Task definition uploaded when the task is published, task is not accepted before
this file has be uploaded to S3 by the queue.

### `/<task-id>/resolution.json`
When a task is resolved the following structure is written to this file:
``` Javascript
{
  "version":            "0.2.0",
  "status":             // Task status structure
  "result":             // URL to results.json from run that completed the task
  "logs":               // URL to logs.json from run that completed the task
  // If task failed then both `results` and `logs` are undefined.
}
```

### `/<task-id>/runs/<run-id>/logs.json`
This is a mapping from logical log name to URL for fetching said log. It must be
created as soon as a worker knows where it's logs will be stored. If logs are
uploaded after the run, you can upload this after logs have been uploaded. If
logs are streamed to an online resource, this file should be uploaded as soon
as stream have started. This must be uploaded before the task is resolved.

Example:
``` Javascript
{
  version:                "0.2.0",
  logs: {
    "stderr":             // URL to stderr
    "stdout":             // URL to stdout
    "warnings":           // URL to a task specific log...
    "worker-commands":    // URL to log of commands worker executed
    // Worker may put any number of logs in here, the key is a name of the log,
    // the value is a URL. Workers can put worker-specific logs, and even allow
    // for task specific logs...
  }
}
```

### `/<task-id>/runs/<run-id>/result.json`
Uploaded just before a task is resolved, once uploaded the worker may declare
the task resolved. This document should contain all interesting results or links
to interesting artifacts uploaded during the task.

Example:
``` Javascript
{
  "version":            "0.2.0",
  "artifacts": {
    "name":             // URL of artifact typically uploaded to ../artifacts/
    ...
  },
  "statistics": {
    "started":          // Timestamp of executing start
    "finished":         // Timestamp of execution end
  },
  "worker": {
    "worker_group":     // Worker group identifier
    "worker_id":        // Worker identifier
  }
  "result":             // Task-specific JSON blob
}
```


### `/<task-id>/runs/<run-id>/artifacts/...`
All other artifacts a task would like to upload during its run is uploaded under
this prefix.
