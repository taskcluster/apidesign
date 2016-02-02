TaskCluster Queue REST API
==========================

REST API version 0.2.0 available under, `/0.2.0/`.

    POST  /task/new
    GET   /task/<task-id>/status
    POST  /task/<task-id>/claim
    GET   /task/<task-id>/artifact-urls
    POST  /task/<task-id>/completed
    GET   /claim-work/<provisioner-id>/<worker-type>
    GET   /pending-tasks/<provisioner-id>


REST API additions for version 0.2.1.

    POST  /task/<task-id>/cancel
    POST  /task/<task-id>/failed


End-Point Details
-----------------

### Common Error Codes

 * `400` Client syntax errors, such as invalid JSON posted by client or JSON
   posted by client
 * `500` Internal server failures, such as failure to upload to S3, failure to
   access database or failure to post a message over RabbitMQ.

### POST `/task/new`

This request takes a task definition and ensures that the task will eventually
be resolved. This generates a uuid server-side, this can be extracted from the
returned task status structure.

**Request**

    {
      // See TaskDefinition.md
    }

**Response**

    {
      status:         // Task status structure
    }

### POST `/task/<task-id>/claim`
Claim (or reclaim) a task, this should update the `taken_until` timestamp in the
tasks table and task status structure.

**Request**

    {
      worker_group:   ...
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


### GET  `/task/<task-id>/artifact-urls`
Get urls for uploading build artifacts to S3.

**Request**

    {
      worker_group:   <worker-group>,
      worker_id:      <worker-id>,
      run_id:         <run-id>,
      artifacts: {
        // Mapping from artifact name to mime-type
        'binaries.tar.gz':  'application/x-gtar',
        'details.json':     'application/json'
      }
    }

**Response**

    {
      status:         // Task status structure
      run_id:         // Run-id
      expires:        // Timestamp when signed urls expires
      artifact_urls: {
        'binaries.tar.gz':      // Signed url for upload
        'dump.tar.gz':          // Signed url for upload
      }
    }

**Error codes**
  * `404`, Task not found (ie. never uploaded or resolved)
  * `410`, Task have been assigned to another worker.


### POST `/task/<task-id>/completed`
Report a task as completed.

**Request**

    {
      worker_group:   <worker-group>,
      worker_id:      <worker-id>,
      run_id:         <run-id>,
    }

**Response**
    {
      status:         // Task status structure
    }

**Error codes**
  * `404`, Task not found (ie. never uploaded or resolved)


### GET  `/claim-work/<provisioner-id>/<worker-type>`
Poll a task that requires the given worker-type.

**Request**

    {
      worker_group:   <worker-group>,
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

Generally, workers shouldn't poll the queue for tasks using this method, instead
they should subscribe to message exchanges where new pending tasks are announced.
However, as workers are brought up this is useful to ensure that pending tasks
whose announcements was missed can be handled.

Also, it's a lot easier to poll for task when prototyping a new worker type.

### **GET** `/pending-tasks/<provisioner-id>`
Get list of pending tasks for a given provisioner.

**Response**

    [
      // list task status structures
    ]

Generally, provisioners shouldn't poll this method. There are message exchanges
which provisioners can subscribe to, in order to be updated about task states.
But if a provisioner crashes, this method is useful to restore state, it's
also useful when prototyping something quick initially.


### **POST** `/task/<task-id>/cancel`
Method to cancel a pending or running task, to be defined later.

### **POST** `/task/<task-id>/failed`
Method for workers to report that a task has been found to fail consistently,
hence, no more retries are necessary. This could happen if a worker finds that
a referenced hg revision doesn't exists on referenced repository, and this
isn't due a network failure.

Basically, workers should only report failure here if they predict that no
execution of the task will ever be able to complete.
