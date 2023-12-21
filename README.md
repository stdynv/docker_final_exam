

# The HumansBestFriend app 

:star: Star us on GitHub — it motivates us a lot! \
:man: Yassine Essamadi  & Hamza Reguig Hichem 


![aimeos-frontend](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ao4tbseGZYAKTYlny-QOWw.png)

## Table Of Content

- [Project Overview](#project-overview)
    - [Requirement](#requirement)
    - [Getting Started](#getting-started)
    - [Project Architecture](#project-architecture)
- [Project setup](#project-setup)
    - [Introduction](#introduction)
    - [Task1 setup](#task1-setup)
    - [Task2 setup](#task2-setup)

### Requirement

- The project need to be done inside a virtual machine that run docker and docker compose. Follow the docker documentation as we saw during the course for installation instructions if you don't have it yet.

## Getting started

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

### Project Architecture

![architeture diagram](https://github.com/stdynv/docker_final_exam/blob/main/humans-best-friend/architecture.png)

### Tasks

1. Create a file called docker-compose.build.yml (not running containers). This file will be responsable for building the application images from the Dockerfile contents provided.
2. The images that are build need to be published first in your publish docker registry and then in a private registry. Please make sure you have a frontend for the private registry as we saw in the course.
3. Create another file called `compose.yml` and it will be responsible of deploying the application and all the needed containers. Please make sure that the images are referencing the one in your private registry.

## Project Setup

### Introduction
The prestashop image available [Prestashop image avaible here](https://hub.docker.com/r/bitnami/prestashop) can be used to deploy an e-commerce application. This application make use of two components. a fronten website and a database for storing persistante data.


```docker
docker pull bitnami/prestashop:latest
```

### Task1 Setup

**Step 1: Create a docker network for the application**

```docker
docker network create ynov-network
```
**Step 2:  Create a volume for MariaDB and create a MariaDB container**
- Create MariaDB volume
```docker
docker volume create --name mariadb_data
```
- Create a MariaDB container
```docker
docker run -d --name mariadb --network ynov-network \
-e MYSQL_ROOT_PASSWORD=0000 -e MYSQL_DATABASE=prestashop_db \
-e MYSQL_USER=prestashop_user -e MYSQL_PASSWORD=0000 \
-v mariadb_data:/var/lib/mysql mariadb
```
**Step 3: Create the frontend container** 
- create frontend volume
```docker
docker volume create --name prestashop_data
```
- create frontend container
  ```docker
  docker run -d --name prestashop_front --network ynov-network -e DB_SERVER=mariadb_hichem -e DB_NAME=prestashop_db -e DB_USER=prestashop_user -e DB_PASSWD=0000 -p 8080:80 -v prestashop_data_hichem:/var/data/html prestashop/prestashop
  ```
Access your application at ```localhost:8080```

![prestashop](https://github.com/stdynv/Docker-Prestashop/assets/78117993/47827fcd-12c4-43e9-8f68-828912a178e1)

**Optional Step Ping the app** 
to ping the app on the backend/ frontend : 
- access as root
  
```docker
docker exec -it mariadb bash
```
- update packages :
  
```
apt-get update
```
- install ping linux package
  ```
  apt-get install iputils-ping
  ```
- ping the application
  
  ```
  ping mariadb
  ```

### Task2 setup
**step1: Create two network adresses** 
- front end nework
```bash
docker network create --subnet=10.0.0.0/24 ynov_front_network
```
- backend network
```bash
docker network create --subnet=10.0.0.0/24 ynov_back_network
```
**Step2 : Create new containers**
- Server side container
```docker
docker run -d --name mariadb_container \
--network ynov_back_network \
--ip 10.0.1.2 \
-e MYSQL_ROOT_PASSWORD=1234 \
-e MYSQL_DATABASE= prestashop_db \
-e MYSQL_USER=prestashop_user \
-e MYSQL_PASSWORD=1234 \
-v mariadb2:/var/lib/mysql \
mariadb:latest
```
- front end container
```docker
docker run -d --name prestashop_container \
--network front_network \
--ip 10.0.0.2 \
-e DB_SERVER=10.0.1.2 \
-e DB_NAME=prestashop_db \
-e DB_USER=prestashop_user \
-e DB_PASSWD=1234 \
-v prestashop2:/var/www/html \
prestashop/prestashop:latest
```
- Router : to communicate with the two contianers

```
docker run -d --name routeur \
--network ynov_front_network \
--network ynov_back_network \
--ip 10.0.0.2\
--ip 10.0.1.2\
-p 80:80 \
nginx:latest
```
**Step3 : Router Configuration**
- configurer les interface réseau des conteneurs
```
docker network disconnect ynov_front_network prestashop_container
docker network connect --ip 10.0.0.2 ynov_front_network prestashop_container
docker network disconnect ynov_back_network mariadb_container
docker network connect --ip 10.0.2.1 ynov_back_network mariadb_container
```

**Step4 : Activate IP routage**
- create file on ```/etc/sysctl.conf``` :
```
echo "net.ipv4.ip_forward=1" > /etc/sysctl.conf
```
- configure le routage sur le routeur
```
apt-get update
apt-get install -y procps
sysctl -p
sysctl -w net.ipv4.ip_forward=1
```

**Step5 : join to networks**
```
docker exec routeur ip route add 10.0.0.2 via 10.0.1.1
```
```
docker exec routeur ip route add 10.0.1.2 via 10.0.1.1
```
- runeach command separately :
  ```
  docker run --privileged -it routeur sh
  docker exec -it routeur apt-get update
  docker exec -it routeur apt-get install -y iproute2
  docker exec -it routeur which ip
  ```
**Step6 : Vérifiy passerel**
```
docker exec -it prestashop_container ip route
```
```
docker exec -it mariadb_container ip route
```
- Verify network config on router
```
docker exec routeur ip route
```

**FinalStep : check communication between the two containers**
- install traceroute package
  ```
  docker exec prestashop_container apt-get update
  docker exec prestashop_container apt-get install -y traceroute
  ```
- run traceroute command on each of network adresses :

```
docker exec -it prestashop_container traceroute 10.0.1.2
```
```
docker exec -it mariadb_container ping 10.0.0.2
```
