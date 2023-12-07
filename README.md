

# Deploy PrestaShop application on Docker

:star: Star us on GitHub — it motivates us a lot! \
:man: Yassine Essamadi  & Hamza Reguig Hichem 

[Prestashop](https://prestashop.fr/) is an open source Web application for creating an online store for e-commerce purposes. The application is published under the terms of the Open Software 3.0 license. PrestaShop is also the name of the company that publishes this solution.

![aimeos-frontend](https://res.cloudinary.com/practicaldev/image/fetch/s--81UoW7-c--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://raw.githubusercontent.com/moghwan/dev.to/master/blog-posts/1-prestashop-docker-compose/assets/header.png)

## Table Of Content

- [Project Overview](#project-overview)
    - [Project Architecture](#project-architecture)
    - [Task 1](#task-1)
    - [Task 2](#task-2)
- [Project setup](#project-setup)
    - [Introduction](#introduction)
    - [Task1 setup](#task1-setup)
    - [Task2 setup](#task2-setup)

## Project Overview
the project is divided into two tasks, the first consists of deploying the Prestashop application in the same network range, the second part becomes challenging separate the frontend from the server side by network range is to ensure communication between the two.

- Task 1 : Deploy application on the same network adress
- Task 2 : Communicate the front side and server side with different network adresses 

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
