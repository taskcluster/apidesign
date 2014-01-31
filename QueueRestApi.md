TaskCluster Queue REST API
==========================

REST API version 0.2.0 available under, `/0.2.0/`.

    POST  /task/<task-id>
    POST  /task/<task-id>/claim
    GET   /task/<task-id>/artifact-url
    POST  /task/<task-id>/completed
    GET   /claim-work/<provisioner-id>/<worker-type>
    GET   /pending-tasks/<provisioner-id>


REST API additions for version 0.2.1.

    GET   /task/<task-id>/status
    POST  /task/<task-id>/cancel
    POST  /task/<task-id>/failed


End-Point Details
-----------------

### POST `/task/<task-id>`

This request takes a task definition and ensures that the task will eventually
be resolved. If the task already is resolved, or a task with the same id but a
different definition have been submitted an error is returned.

Basically, this operation is idempotent as long as the task is pending or
running and you're submitting the same definition. The goal here is that a
scheduler can confirm task submission.

**Request**

    {
      // See TaskDefinition.md
    }

**Response**

    {
      status:         // Task status structure
    }

**Error codes**
  * `409` Conflict (task-id is already used by a task with different definition)
    if the task.json already exists on S3, it isn't resolved and the id isn't
    in the database, we add it to the database. If task-id is already on S3 and
    in database and it has the same definition, we just acknowledge that it was
    submitted.


### POST `/task/<task-id>/claim`
Claim (or reclaim) a task, this should update the `taken_until` timestamp in the
tasks table and task status structure.

**Request**

    {
      worker_id:      ...,
      run_id:         // Run-id, if reclaiming a task; otherwise null
    }

**Response**

    {
      status:         // Task status structure,
      run_id:         // Run-id assigned of this run
      logs_url:       // signed url for uploading (expires around taken_until)
      result_url:     // signed url for uploading (expires around taken_until)
    }

**Error codes**
  * `404`, Task not found (ie. never uploaded or resolved)
  * `410`, Task have been assigned to another worker.


### GET  `/task/<task-id>/artifact-url`
Get urls for uploading build artifacts to S3.

**Request**

    {
      worker_id:      <worker-id>,
      run_id:         <run-id>,
      artifacts:      List of artifact names, eg. ['binaries.tar.gz', 'dump.gz']
    }

**Response**

    {
      status:         // Task status structure
      run_id:         // Run-id
      expiration:     // Timestamp when signed urls expires
      artifact_urls: {
        'binaries.tar.gz':      // Signed url for upload
        'dump.tar.gz':          // Signed url for upload
      }
    }

**Error codes**
  * `404`, Task not found (ie. never uploaded or resolved)
  * `410`, Task have been assigned to another worker.


### POST `/task/<task-id>/completed`
Report a task as completed, do not assume that you're task is reported completed
until this method returns `200 OK` or `404`.

**Request**

    {
      worker_id:      <worker-id>,
      run_id:         <run-id>,
    }

**Response**
    {
      status:         // Task status structure
    }

**Error codes**
  * `404`, Task not found (ie. never uploaded or resolved)


### GET  `/claim-work/<worker-type>`
Poll a task that requires the given worker-type.

**Request**

    {
      worker_id:      <worker-id>
    }

**Response**

    {
      status:         // Task status structure,
      run_id:         // Run-id assigned to this run
      logs_url:       // signed url for uploading (expires around taken_until)
      result_url:     // signed url for uploading (expires around taken_until)
    }

**Error codes**
  * `204`, No tasks available, response: `{sleep: <number of seconds>}`
  * `404`, Task not found (ie. never uploaded or resolved)
  * `410`, Task have been assigned to another worker.


### **GET** `/pending-tasks/<provisioner-id>`