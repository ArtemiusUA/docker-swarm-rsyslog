# Rsyslog docker image 

Rsyslog image to help to collect logs over the docker swarm.
Configured in a way to split log files by service names.

## Overview

In case of big Docker Swarm deploys something like ELK will be the best choice.
But for smaller cases it may be an overkill, so better to use Rsyslog to collect
plain old text logs somewhere on manager node or on separate node on cheap Digital Ocean droplet.

## How-to

Add logs service to your rig as in below example.
Personally I like to make separate labels to limit services to specific nodes in swarm, as in example.  
But You can just limit it to management node if you have only one (e.g. node.role==manager). 
Then add Logspout service in global mode (it will start on each node).
Logspout will will be collecting logs from containers and pass them to logs service, 
where they will be splited by services and stored into separate files. 


```yaml
version: "3.5"

services:

  logs:
    image: artemiusua/docker-swarm-rsyslog:latest
    volumes:
      - type: bind
        source: /var/log/docker
        target: /var/log/docker
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.logs_role == master
      restart_policy:
        condition: any

  logspout:
    image: gliderlabs/logspout
    command: "syslog+tcp://logs:514"
    environment:
      LOGSPOUT: ignore
    volumes:
    - type: bind
      source: /run/docker.sock
      target: /var/run/docker.sock
    - type: bind
      source: /etc/hostname
      target: /etc/hostname
      read_only: true
    depends_on:
      - logs
    deploy:
      mode: global
      restart_policy:
        condition: any   
```

