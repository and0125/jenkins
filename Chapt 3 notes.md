# jenkins

A repo to learn about Jenkins

## Defining Jenkins Architecture

In a distributed microservices architecture, you may have multiple servies to build, test and deploy regularly. This can make having multiple build machines make sense. Jenkins can be configured to run distributed builds across a fleet of nodes by setting up a boss and worker cluster.

- boss/master node: responsible for scheduling build jobs and distributing builds to workers for execution. It also monitors the workers states and collects and aggregates the build results.
- worker notes: this is a java executabe that runs on a remote machine, listens to request from the boss, and executes build jobs. You can have as many workers as you want and they can be added or removed on teh fly.

Jenkins can be run in a standalone mode, but its more effective when run as a cluster.

In this cluster structure, the boss handles scheduling and dispatching jobs, monitoring progress, and recording and presenting the results. These boss nodes can executre build jobs directly as well.

worker nodes can be added and configured on the Jenkins dashboard or through Jenkins Rest API.

### Managing Workers

You can manage Workers from:

- SSH
- CLI
- Jenkins

## Architecting Jenkins for Scale on AWS

It's possible to deploy a standalone or single node setup on AWS by deploying a Jenkins server on an EC2 instance form the AWS marketplace.

### Single Node Setup

There's an AMI image for Jenkins, and youcan select Jenkins LongTerm Support release and the machine instance type.

It's also possible to install Jenkins on an EC2 instance with the base image by using a package manager.

Once Jenkins is installed, you have to configure the security group attached to the instance to allow traffic on port 8080 for requests to hit this server. it can also be helpful to add an inbound security policy to allow for SSH traffic. by default the egress will allow all traffic out.

### Cluster Setup

It can be better to deploy a Jenkins cluster as it will scale for larger and complex projects. This will share the workload across multiple workers. Then instead of scheduling jobs on the boss instance, you'll assign the execution to the worker nodes. This will result in additional EC2 instances being deployed.

A typical configuration would be one manager node and 3 working nodes, but it can be helpful to not fix the number of workers in advance, by using an Auto Scaling Group, where you can define the minimum and maximum number of workers that would be added at any given time.

The ASG scaling functions can be set to scale in or out from a Cloudwatch Alarm, which can be set to watch the CPU usage of each instance. A common policy could be to scale out when the average CPU usage is above 80% for all workers, and scale in if the average usage is less than 20%.

Part of setting up this Auto Scaling Group functionality is to add a shell script to each of the nodes which will run at boot time which will use the Jenkins API to register the new working node's IP address with the manager node.

Another alternative can be to use the number of jobs wating in the build queue to trigger scaling. To do this, you'd need a tool like Prometheus that you could configure within a lambda function that looks at the Jenkins cluster metrics exported through Prometheus.

#### Securing a Cluster

To secure the cluster, you'll want the cluster to run within a private subnet, separate from the default VPC, and this can be updated to be more secure. One thing to use here would be specific Classless Inter-Domain Routing to block range and subnet sizes for the IP addresses of the cluster nodes.

You may also need to deploy an internet gateway (IGW) and attach it to the VPC of the Jenkins cluster, which is helpful for ensuring the cluster has internet connectivity for downloading external packages. A network Address translation (NAT) instance or gateway will be setup inside the private VPC of the Jenkins cluster, and the IGW will forward traffic to the NAT instance or gateway.

Because the Jenkins instances will be in this private subnet, you won't be able to SSH into them directly. One common workaround for this is to deploy a special instance that acts as a proxy and using that instance to SSH into your Jenkins instances. This is usually called a bastion host or a jump box. This jump box instance will be in the public subnet, and will rout only SSH traffic from your network over the Jenkins instances be setting up a secure SSH tunnel.

Another useful feature for a Jenkins cluster can be to set up a elastic load balancer in front of your manager instance so that you can view the Jenkins dashboard from the web, but also not expose your Jenkins instance directly to the internet.

Another useful feature is to set up multiple manager nodes in case one fails, and those can be reached from that elastic load balancer.

## Summary

- Deploying jenksins in a cluster allows for decoupling orchestration and build executions, and this can provide better performance.
- you need a highly available setup for jenkins to ensure your DevOps has no downtime.
- Setting up a cluster to run in distributed mode means you have a manager node that only does the delegation and scheduling of work, and worker nodes that perform the actual execution of build jobs.
