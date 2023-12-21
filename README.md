

# The HumansBestFriend app 

:star: Star us on GitHub — it motivates us a lot! \
:man: Yassine Essamadi  & Hamza Reguig Hichem 


![aimeos-frontend](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ao4tbseGZYAKTYlny-QOWw.png)

## Table Of Content

- [Project Overview](#project-overview)
    - [Requirement](#requirement)
    - [Getting Started](#getting-started)
    - [Project Architecture](#project-architecture)
    - 
    - [Task 1](#task-1)
    - [Task 2](#task-2)
- [Project setup](#project-setup)
    - [Introduction](#introduction)
    - [Task1 setup](#task1-setup)
    - [Task2 setup](#task2-setup)

## Project Overview
the project is divided into two tasks, the first consists of deploying the Prestashop application in the same network range, the second part becomes challenging separate the frontend from the server side by network range is to ensure communication between the two.
### Requirement

- The project need to be done inside a virtual machine that run docker and docker compose. Follow the docker documentation as we saw during the course for installation instructions if you don't have it yet.

### Getting started

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.


### Project Architecture

![architeture diagram](https://github.com/stdynv/Docker-Prestashop/assets/78117993/5d48aaed-5194-45c8-a33a-9a5e7f925d59)

### Task 1 
Deploy this application inside a network. Make sure the two containers can communicate with each other using their names.

### Task 2
create two networks with the cidr specified in the schemas. Please make use of `--subnet` for adding a subnet to the network when creating it. The name of the networks are `ynov-frontend-network` for the frontend website container and `ynov-backend-network` for the database container.
*in order for the two containers to talk to each other* :
- make use of a third container that can be called `gateway` or `router` and connect this container to the two above networks.
- Inside the router gateway, configure the route table by using ip below command

**Note**: The goal for the second task is not to deploy the application but to make sure the containers can talk to each other using their ips.

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
