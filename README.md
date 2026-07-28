# CodeAlpha Jenkins Remoting Project

## Overview
Set up a Jenkins controller and a separate Jenkins agent, connected over Docker networking, to demonstrate distributed builds and node isolation.

## Setup Commands

**Controller:**
```bash
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-controller jenkins/jenkins:lts
```

**Custom network:**
```bash
docker network create jenkins-net
docker network connect jenkins-net jenkins-controller
```

**Node created in Jenkins UI** (Manage Jenkins > Nodes > New Node):
- Name: `remote-agent-1`
- Remote root directory: `/home/jenkins/agent`
- Labels: `remote linux-agent`
- Launch method: Launch agent by connecting it to the controller
- Usage: Only build jobs with label expressions matching this node

**Agent container:**
```bash
docker run -d --name jenkins-agent-1 \
  --network jenkins-net \
  jenkins/inbound-agent:latest \
  -url http://jenkins-controller:8080/ \
  -secret 8bb9f3047542f5b922343b21d8225f02f4ca70310a1ac90c89495010ae592b8e \
  -name remote-agent-1 \
  -webSocket
```

## Proof of Remoting
A freestyle job `remoting-test` was created, restricted to the label `remote`, running a shell step that prints the container hostname. Build output confirmed execution on `remote-agent-1`:

```
Started by user Confidence Nwaokike
Running as SYSTEM
Building remotely on remote-agent-1 (linux-agent remote) in workspace
/home/jenkins/agent/workspace/remoting-test
+ hostname
8126ae0b2606
+ echo Running on: 8126ae0b2606
Running on: 8126ae0b2606
+ echo This is remoting in action
This is remoting in action
Finished: SUCCESS
```

## Node Isolation
The agent's Usage setting was changed to "Only build jobs with label expressions matching this node" to prevent unrelated jobs from being scheduled on it.

## Screenshots
See `/screenshots` for: agent online in Nodes list, job config, build console output confirming remote execution.

## Demo
LinkedIn write-up (video walkthrough attached to the post):

> Done with Task 2 of my DevOps internship at CodeAlpha — Jenkins Remoting.
>
> For this one I set up a Jenkins controller and a separate agent (both in Docker containers),
> connected them over a custom network, and got the agent registered as a remote node. Ran a
> test job restricted to that node and confirmed in the console output that it actually
> executed on the agent and not the controller.
>
> Also locked the node down so it only picks up jobs meant for it, instead of accepting
> anything — small thing but it's the kind of setup you'd actually want in a real environment.
>
> Video walkthrough is attached, and the full setup + screenshots are on GitHub: https://lnkd.in/e6ye2Wsw
>
> Thanks @CodeAlpha for the task.
>
> #DevOps #Jenkins #Docker #CodeAlpha
