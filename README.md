# CodeAlpha Jenkins Remoting Project

## Overview
Set up a Jenkins controller and a separate Jenkins agent, connected over
Docker networking, to demonstrate distributed builds and node isolation.

## Setup Commands

Controller:
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-controller jenkins/jenkins:lts

Custom network:
docker network create jenkins-net
docker network connect jenkins-net jenkins-controller

Node created in Jenkins UI (Manage Jenkins > Nodes > New Node):
- Name: remote-agent-1
- Remote root directory: /home/jenkins/agent
- Labels: remote linux-agent
- Launch method: Launch agent by connecting it to the controller
- Usage: Only build jobs with label expressions matching this node

Agent container:
docker run -d --name jenkins-agent-1 \
  --network jenkins-net \
  jenkins/inbound-agent:latest \
  -url http://jenkins-controller:8080/ \
  -secret <secret-from-node-page> \
  -name remote-agent-1 \
  -webSocket

## Proof of Remoting
A freestyle job "remoting-test" was created, restricted to the label
"remote", running a shell step that prints the container hostname.
Build output confirmed execution on remote-agent-1 (see screenshots).

## Node Isolation
The agent's Usage setting was changed to "Only build jobs with label
expressions matching this node" to prevent unrelated jobs from being
scheduled on it.

## Screenshots
See /screenshots for: agent online in Nodes list, job config, build
console output confirming remote execution.
