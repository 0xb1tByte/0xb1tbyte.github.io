---
layout: post
title:  "Docker Get Started"
category: docker
---

## Environment

Windows 10 with Docker Toolbox

## Installation

```bash
# 版本
docker --version

# 資訊
docker info
docker version

# 測試安裝
docker run hello-world

# 列出已下載的 image
docker image ls
```

## Orientation

- "A container is launched by running an image."
- "A container runs natively on Linux and shares the kernel of the host machine with other containers."

    > 多個 container 在同一個 kernel

```bash
# 列出所有正在運行的 container
docker ps
```

## Container

- Define a container with `Dockerfile`

```bash
# Build
docker build -t [image_name] .

# Run
docker run -p [machine_port]:[container_port] [image_name]

# 列出 container 資訊
docker container ls

# Stop
docker container stop [container_id]
```

## Service

- "In a distributed application, different pieces of the app are called “services.”"
- "Services are really just “containers in production.”"
- To define, run, and scale services

    > 在 `docker-compose.yml` file 裡可設定多個 service

- "A single container running in a service is called a task."
- "Docker performs an in-place update, no need to tear the stack down first or kill any containers."

    > 改完 `docker-compose.yml` 後重新執行一次 Deploy service 的指令

```bash
# 使主機成為一個 swarm
docker swarm init

# Deploy stack onto a swarm
docker stack deploy -c docker-compose.yml [app_name]

# 列出目前正在執行的 service
docker service ls

# 列出 service 上的 task
docker service ps [service_name]

# 列出主機 constainer 上所有的 task
docker container ls -q

# Take the app down
docker stack rm [app_name]

# Take down the swarm
docker swarm leave --force
```

## Swarm

- "Multi-container, multi-machine applications are made possible by joining multiple machines into a “Dockerized” cluster called a swarm."
- "A swarm is a group of machines that are running Docker and joined into a cluster."

    > 多台主機 (physical or virtual) 組成 swarm cluster

- "Swarm managers are the only machines in a swarm that can execute your commands, or authorize other machines to join the swarm as workers."

```bash
# 使主機成為一個 swarm manager
docker swarm init

# 使用 virtualbox 建立 VM
docker-machine create --driver virtualbox

# 列出所有 docker vm
docker-machine ls

# 透過 ssh 使指定 vm 成為一個 swarm manager
docker-machine ssh [vm_name] "docker swarm init --advertise-addr [vm_ip]"

# 使主機加入 swarm 成為一個 swarm worker
docker swarm join --token [token] [vm_ip]:[vm_port]

# 列出 swarm node 列表
docker-machine ssh [vm_name] "docker node ls"

# get the command to configure your shell to talk to [vm_name]
docker-machine env [vm_name]

# 列出 stack 上所有 service
docker stack ps [app_name]

# 移除 stack
docker stack rm [app_name]
```

## Stack

- "A stack is a group of interrelated services that share dependencies, and can be orchestrated and scaled together."
- Add services to stack in `docker-compose.yml`

## Conclusion

### Container

```text
+-----------+
| container |
+-----------+
  ^
  | Run
+-------+
| image |
+-------+
  ^
  | Build
+------------+
| Dockerfile |
+------------+
```

```text
+-----------+-----------+-----------+
| container |    ...    | container |
+-----------+-----------+-----------+
|              Kernal               |
+-----------------------------------+
|                OS                 |
+-----------------------------------+
```

### Service &  Stack

```text
+-----------------------------------------+
| Stack                                   |
| +-------------+---------+-------------+ |
| | Service     |   ...   | Service     | |
| | +---------+ |         | +---------+ | |
| | |Container| |   ...   | |Container| | |
| | +---------+ |         | +---------+ | |
| +-------------+---------+-------------+ |
+-----------------------------------------+
  ^
  | Deploy
+-------------------------------------+
| docker-compose.yml                  |
| +---------------------------------+ |
| | Stack                           | |
| | +---------+---------+---------+ | |
| | | Service |   ...   | Service | | |
| | +---------+---------+---------+ | |
| +---------------------------------+ |
+-------------------------------------+
```

### Swarm

```text
+---------------------------------+
|            Services             |
+---------------------------------+
|              Stack              |
+---------+--------+-----+--------+
| Manager | Worker | ... | Worker |
+---------+--------+-----+--------+
| machine | machine| ... | machine| <- physical or virtual
+---------+--------+-----+--------+
|              Swarm              |
+---------------------------------+
```

## Reference

- [Docker Docs - Get started with Docker]( https://docs.docker.com/get-started/)