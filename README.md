# jenkins

A repo to learn about Jenkins

## Defining Jenkins Architecture

In a distributed microservices architecture, you may have multiple servies to build, test and deploy regularly. This can make having multiple build machines make sense. Jenkins can be configured to run distributed builds across a fleet of nodes by setting up a boss and worker cluster.

- boss/master node: responsible for scheduling build jobs and distributing builds to workers for execution. It also monitors the workers states and collects and aggregates the build results.
- worker notes: this is a java executabe that runs on a remote machine, listens to request from the boss, and executes build jobs. You can have as many workers as you want and they can be added or removed on teh fly.

Jenkins can be run in a standalone mode, but its more effective when run as a cluster.

In this cluster structure, the boss handles scheduling and dispatching jobs, monitoring progress, and recording and presenting the results. These boss nodes can executre build jobs directly as well.

worker nodes can be added and configured on the Jenkins dashboard or through Jenkins Rest API.

## Managing Workers

You can manage Workers from:

- SSH
- CLI
- Jenkins 