# System Design - Job Schedular

Asynchronous Task Framework (ATF) - in Dropbox Blog

READ 2022/03/03

UPDATED 2022/08/01

## Requirements

A job can be scheduled for one time or multiple executions (cron job) by other services/microservices.

Be able to handle with different priority of execution of tasks - Tasks with higher priority should get executed first as possible

Clients can query the status of a scheduled/running task - Results of job executions are stored and can be queried.

## Non-Functional Requirements

One hundred million tasks execution per day. Be able to handle 1,000 async tasks per second

Highly available

Latency - the time between the scheduled time and task starting time should be minimal.

The same task shoule not be running multiple times at the same scheduled time.

## System Guarantees

At-least once task execution

No concurrent execution for the same task

Isolation between tasks

Delivery latency - 95% of tasks begin execution within five seconds from their scheduled execution time

## Infra Graph

![img](https://dropbox.tech/cms/content/dam/dropbox/tech-blog/en-us/2020/11/atf/diagrams/Techblog-ATF-720x844px-1.png)

## Components Detail Function (My own design solution)

### Task Database

ATF tasks' info are stored in and triggered from the task store. The task store could be only the metadata of a task like (id, start_time, status, context_id, submitter, submit_time, ... etc).

There could be another database storing the code entity and task detailed files.

### Store Consumer/Job Coordinator

The Store Consumer is a service that periodically queries the task store to find tasks that are ready for execution and pushes them into one of the multiple queues. These tasks are newly ready for execution or retrying ones. At the same time it updates the task's status. (like SCHEDULED -> READY)

### Queue

ATF uses SQS to queue tasks internally. These queues act as a buffer between the Store Consumer(coordinator) and Controllers(executors).

### Controller & Executor

Each worker host has one process responsible for polling tasks from SQS queues in a background thread, and then pushing them onto process local buffered queues. The Executor is a process with multiple threads, responsible for the actual task execution.

Each host has a single Controller process and multiple executor processes.

### Dead Letter Queue

Create dead letter queues filled with such misbehaving tasks which are got stuck in infinite retry loops due to occasional bugs in logic, or retried failed for certain times.

## Lifecycle of a task

* Client performs a Schedule RPC call to Frontend with task information, including execution time.

* Frontend creates EdgeStore entity and assoc for the task.

* When it is time to process the task, Store Consumer pulls the task from EdgeStore and pushes it to a related SQS queue.

* Executor makes NextWork RPC call to Controller, which pulls tasks from the SQS queue, makes a ClaimTask RPC to the HSC and then returns the task to the Executor.

* Executor invokes the callback for the task. While processing, Executor performs Heartbeat RPC calls to Heartbeat and Status Controller (HSC). Once processing is done, Executor performs TaskStatus RPC call to HSC.

* Upon getting Heartbeat and TaskStatus RPC calls, HSC updates the EdgeStore entity and assoc.

## Data Model

### Jobs

job_id, description, name, cron_definition,(or) one_time_scheduling, url, status, created_user, created_time, ...

### Job Execution

execution_id, job_id, scheduled_time, start_time, status, ...

#### Cron Strategy

0 0 12 * * ? - Fire at 12pm (noon) every day

0 15 10 ? * * - Fire at 10:15am every day

## State Machine Of Task State Transition

![img](https://dropbox.tech/cms/content/dam/dropbox/tech-blog/en-us/2020/11/atf/diagrams/Techblog-ATF-720x225px-2.png)

## Reference

[1] <https://dropbox.tech/infrastructure/asynchronous-task-scheduling-at-dropbox>

[2] <https://xunhuanfengliuxiang.gitbooks.io/system-desing/content/design-job-scheduler.html>

[3] <https://leetcode.com/discuss/general-discussion/1082786/System-Design%3A-Designing-a-distributed-Job-Scheduler-or-Many-interesting-concepts-to-learn>
