S3 Storage Layout
=================

We need a bucket full of truth, for now let's used `jonasfj-taskcluster-tasks`,
layout will be as follows:

    s3://.../<task-id>/task.json
    s3://.../<task-id>/resolution.json
    s3://.../<task-id>/runs/<run-id>/logs.json
    s3://.../<task-id>/runs/<run-id>/results.json
    s3://.../<task-id>/runs/<run-id>/artifacts/...

File Details
------------

### `/<task-id>/task.json`
Task definition uploaded when the task is published, task is not accepted before
this file has be uploaded to S3 by the queue.

### `/<task-id>/resolution.json`
When a task is resolved a status JSON blob is written to this file. This file is
never overwritten, as a task is only resolved once. Basically this points to
which `<run-id>` caused task resolution and dump of database entries for this
task before it was deleted from the database.

### `/<task-id>/runs/<run-id>/logs.json`
Created as soon as a worker knows where it's logs will be stored. If logs are
uploaded after the run, you can upload this after logs have been uploaded. If
logs are streamed to an online resource, this file should be uploaded as soon
as stream have started. Don't upload after task is resolved.

### `/<task-id>/runs/<run-id>/results.json`
Uploaded just before a task is resolved, once uploaded the worker may declare
the task resolved. This document should contain all interesting results or links
to interesting artifacts uploaded during the task.


### `/<task-id>/runs/<run-id>/artifacts/...`
All other artifacts a task would like to upload during its run is uploaded under
this prefix.
