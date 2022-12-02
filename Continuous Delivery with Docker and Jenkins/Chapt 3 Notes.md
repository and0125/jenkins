# Configuring Jenkins

## Installing Jenkins on Docker

The Jenkins image is avilable from the Docker Hub registry so to install we should execute:

`docker run -p 8080:8080 jenkins/jenkins:lts` where the version number is optional. This command installs the latest stable edition, but you may want to use the version number directly in production to avoid any breaking changes when rebuilding.

Running this image will provide an admin user and a password to proceed with the jenkins installation. Make sure to store the password somewhere safe, but its available to see at the following location within the container: /var/jenkins_home/secrets/initialAdminPassword.

If the plugins do not install, it's likely that you are using too old of a version of jenkins, and should upgrade it.

The port will eventually open. Go to `localhost:8080` to view the jenkins main menu. Use the admin password to login. It will ask you to install some plugins for jenkins to work; the default option is just to install all the plugins.

Once installed, the jenkins front-end will ask you to set up a username, password, and other basic access information. If you skip this, you will have to use the initialAdminPassword as the password to access the server.

It will also ask you to establish the URL for accessing the Jenkins dashboard for API actions. You can change if needed, but it defaults to `localhost:8080`.

## Hello World

Steps were:

- click add a new item
- name the item hello world
- choose pipeline as the item to create
- There are a lot of options for the pipeline, but go to the advanced optiosn to get to the Pipeline script section.
- The book has a sample script on page 83 to insert
- click save
- click build now.

There's a build history that pops up that is now called `Stage View`. You can click on the build you want to view to see the details from the pipeline build. These are numbered, and the first build is \#1.

Once you click on the build, you can click in the left hand panel to see the console output.

## Jenkins Architecture

This demo executed in no time, but most pipelines are complex, and time is spent downloading objects from the internet and compiling source code, and running tests, which can take hours to build.

In common scenarios, there may also be concurrent pipelines run. Usually the whole team, and possibly the whole organization uses a single jenkins instance.

To ensure it runs quickly and smoothly, jenkins uses manager and worker nodes.

### Managers and Workers

If one team commits frequently, and builds take a decent amount of time, you can quickly overwhelm one instance of jenkins.

In order to avoid this, we want to configure jenkins to delegate responsibility of executing builds to worker instances.

The node you start on is the jenkins manager node, and you don't want to execute builds on that node.

The manager node is responsible for the following activities:

- receiving build triggers (for example, from a commit to GitHub)
- sending notifications (emails sent after build success or failure)
- Handling HTTP requests (for interactions with clients)
- managing the build environment (managing worker nodes)

The managers are usually a dedicated machine with RAM ranging from 200 MB for small projects to 70+ GB for huge projects.

The worker nodes generally have no requirements except that they have the RAM to execute a single build, so if the project requires 100 GB of RAM to execute, then all the worker nodes should meet that.

These agent or worker nodes should be as generic as possible, to be able to build projects written in any langauge. That way, these agents can be interchanged to help with optimizing resources. It's also possible to tag agents so that they only build specific parts of the pipeline on a given agent if its impossible to be completely generic for a base image.

## Test and production Instances

Because jenkins itself is a software, you'll want to update it from time to time. To do this you should always have a test and production instance of jenkins running. The test environment should always be as similar as possible to the production environment, so it requires a similar number of agents to be attached.

For instance, Netflix has test and production manager instances, which each of them owning a pool of worker nodes, and have the ability to add worker nodes at any moment. Netflix had their ad-hoc instances provided from their internal organization instead of from AWS as well, which is an interesting hybrid feature.

## Communication between Jenkins Nodes

Thre are different protocols that can be used to communicate between workers and managers, but the bi-directional communication has to be established:

- SSH: the manager connects to the worker using the SSH protocol. Jenkins comes with an SSH client built-in, so the only requirement is that the SSH daemnon is configured on teh worker nodes. This is the most convenient and stable way to connect because it uses standard Unix methods.
- Java Web Start: a java application si started on each worker machine and the TCP connection is established between the jenkins worker and the manager. This is often used if the workers are inside a fire-walled network and the manager cannot SSH in.

## Setting Agents

The agents always communicate with the manager using one of these two protocols, but at a higher level, they can be attached in various ways based on some considerations:

- static versus dynamic: The simplest option is to add workers permanently to the manager. The drawback of this is that you will always have to manually change something if you want more or less workers. A better option is to dynamically provision workers as needed.
- Specific versus general-purpose: agents can be specific or general purpose.

the combination of these considerations result in four common strategies:

- permanent agents: adding specific agent nodes to the manager node thorugh the jenkins web interface.
- permanent docker agents
- jenkins swarm agents
- dynamically provisioned docker agents

### permanent agents

This is done by going in the interface to: Manage Jenkins > Manage Nodes > new Node.

when you create the new node you'll give it the following properties:

- name
- description
- \# of executors: this is the number of concurrent builds that can be run on the worker.
- Remote root directory: this is the directory on the worker machine that the agent can use to run build jobs. The most important data is transferred back to the manager, so the directory is not critical.
- labels: this includes the tags to match specific builds.
- usage: this is the option to decide whether the agent should only be used for matched labels or for all builds.
- launch method: has the following options:
  - Launch via Java Web Start: the connection will be estblished by the worker node; it is possible to download the JAR file and the instructions on how to run it on the worker node.
  - Launch agent via execution of command on the master: this is the custom command run on the manager node to start the worker, in most cases it will send the Java Web Start application and start it on the worker.
  - Launch agent via SSH: here the manager will ocnnect to the worker directly with the SSH protocol.
- Availabiltiy: This is the option to decide whether the agent should be up all the time or the master should turn it offline at some times.

Note that the Java Web Start application uses port 50000 (50,000) for communications, so if you use the docker-based manager, you want to publish that port with `-p50000:50000`.

Once agents are setup correctly, you can reduce the # of executors on the manager to 0, so that it does not execute any builds.

The flaw to this solution is that you need to maintain multiple worker types for different projects and environments.

### Permanent Docker Agents

the idea behind this solution is to have general-purpose worker nodes. Each will be configured with the docker engine installed, and each build is defined along with the docker image the build should run within.

The configuration is static, so its done the same way we did this for the permanent agents, the only difference is that we need to install docker on each machine that will be a worker node. Then labels won't be necessary because any worker can run any build because the image to use will be defined in the pipeline.

Then, once the build is started, the Jenkins worker starts a container from the docker image and executes the steps of the pipeline inside that container. this way you know the execution environment and don't have to configure each worker separately.

## Jenkins Swarm Agents

So far, the definition of the environment for each build has had to be defined in the manager node at some point. Such a solution though, can be a burden if you need to frequently scale the number of worker machines. Jenkins Swarm allows you to dynamically add workers without the need to configure them in the Jenkins manager node.

Jenkins swarm has a self-organizing swarm plug-in module you can install from the Jenkins Web UI. Go to Manage Jenkins > Manage Plugins and install the swarm plugin.

From there, the Jenkins manager is prepared for workers to be added dynamically.

The second step is to run the Jenkins swarm application on every machine that would act as a worker. This can be done with the swarm-client.jar application, which can be downloaded form the Jenkins swarm plugin page (see page 94).

### Dynamically Provisioned Docker Agents

Another options is to set up Jenkins Dynamically to create a new build agent each time a build is started. This is the most flexible approach, since the number of workers adjusts to the number of builds.

you can do this by installing the docker plugin in the Jenkins manager (Manage Jenkins > manage Plugins).

Dynamically provisioned docker agents can be treated as a layer over the standard agent mechanism.

When a Jenkins job is started, the manager runs a new container from the worker image on the Docker host.

The worker container is actually an Ubuntu image with the sshd server installed.

The Jenkins manager automatically adds the created agent to the agent list.

The agent is accessed using the SSH communication protocol, to perform the build.

After the build, the manager stops and removes the agent.

As a note, you can run the jenkins manager from a docker container as well as making docker containers the jenkins workers too.It's reasonable to do both, and they will work separately.

The solution now containerizes the whole worker node, not just the build environment. This makes the agent's life cycle automated, and allows for easy scalability.

No matter what agent configuration you use, you can check whether everything works correctly.

### Creating your own Jenkins Images

You may want to build specific images to satisfy the build environment requirements.

For a worker node, you'll want to:

- create a dockerfile
- build the image
- push the image to a registry
- change the agent configuration on the master node

You may also want to build a manager node image which may be in order to be able to scale manager nodes efficiently as well.

Jenkins is configured with XML files and provides a DSL language that's like Groovy. You can add a Groovy script to a Dockerfile in order to manipulate the Jenkins configuration.

## Configuration management

Jenkins configuration can be managed through the Web interface for the most part.

three commonly updated configuration items are:

- Security: You can add users to your jenkins manager because it comes with a database created during the initial configuration process. These can be managed from the Manage users setting page. You can also implement LDAP for access, and this is done from the COnfigure Global Security page. By default, logged in users can do anything, so you want to think about roles, groups, and permissions for large scale organizations.
- Plugins: there are tons of plugins to choose from and you can write your own if needed.
- Backups: You can either install a plugin to do periodic backups or setup a cron job to archive the directories into a safe place.
