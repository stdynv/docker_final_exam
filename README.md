

# The HumansBestFriend app 

:star: Star us on GitHub — it motivates us a lot! \
:man: Yassine Essamadi  & Hamza Reguig Hichem 


![aimeos-frontend](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ao4tbseGZYAKTYlny-QOWw.png)

## Table Of Content

- [Project Overview](#project-overview)
    - [Requirement](#requirement)
    - [Getting Started](#getting-started)
    - [Project Architecture](#project-architecture)
- [How to use](#how-to-use)
    - [Installation](#installation)
    - [Results](#results)

## Project overview
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

## How to use 

### Prerequisites
- The project need to be done inside a virtual machine that run docker and docker compose. Follow the docker documentation as we saw during the course for installation instructions if you don't have it yet.
- You should have installed docker & docker-compose

### Installation 
clone the repository using : 
  ```git clone https://github.com/stdynv/docker_final_exam ```

navigate to to the folder where the composers exists
``` 
cd /humans-best-friend
```
run docker build image compose file 
```docker
docker-compose -f docker-compose.build.yml build
```
it will build multiple images found on the dockerfile of the folders, to view the different images created you can run the command bellow 
```docker
docker iamges
```
2 - create priavte registry 
if you don't have registry installed, please pull the image using this command
```docker
docker pull registry
```
then run it on port 5000
```docker
docker run -d -p 5000:5000 --restart always --name registry registry:latest
```
Create tags for the images created 
```docker
docker tag humans-best-friend_worker localhost:5000/humans-best-friend_worker
```
```docker
docker tag humans-best-friend_vote localhost:5000/humans-best-friend_vote
```
```docker
docker tag humans-best-friend_result localhost:5000/humans-best-friend_result
```
```docker
docker tag humans-best-friend_seed-data localhost:5000/humans-best-friend_seed-data
```
upload the dockers images to a remote registry (private)

```docker
docker push localhost:5000/humans-best-friend_vote
```
```docker
docker push localhost:5000/humans-best-friend_result
```
```docker
docker push localhost:5000/humans-best-friend_seed-data
```
```docker
docker push localhost:5000/humans-best-friend_worker
```
to check if it's uploaded successefully : 
```curl localhost:5000/v2/_catalog```

3- using the ```compose.yml``` file we can create our container using this command :
```docker
 docker compose up -d
```
it will create and start the containers 

### Results: 
two interfaces are created, one for making the vote and the other the see the results of the votes

to access to the vote ```ip_adress:5002```
to access to the results ```ip_adress:5001```

What is my iP ? 
```
ip a
```

