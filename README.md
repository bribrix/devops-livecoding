# **Report - DevOps course by Takima**
*Report by Briac Marchandise*

## $ First Part - Docker

***1-1 Document your database container essentials: commands and Dockerfile.***

- `Dockerfile`
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

Commands
- `docker build -t my-database .`: Build the image
- `docker run -d --name my-postgres-db -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -p 5432:5432 my-database`: Run the container
- `docker ps`: List all running containers

***1-2 Why do we need a multistage build? And explain each step of this dockerfile.***

-> A multistage build enables to reduce image size, the separation of build and runtime environment and better catching and reusability of layers, speeding up builds

```
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build  #Select Build Image
ENV MYAPP_HOME /opt/myapp   #Set Environment Variable
WORKDIR $MYAPP_HOME   #Set Working Directory
COPY pom.xml .      #Copy Files
COPY src ./src
RUN mvn package -DskipTests     #Build the Application

# Run
FROM amazoncorretto:17     #Select Runtime Image
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar    #Copy Artifact from Build Stage

ENTRYPOINT java -jar myapp.jar      #Define Entry Point
```

***1-3 Document docker-compose most important commands.***

- `docker-compose up`: Starts and runs the app
- `docker-compose down`: Stops and removes containers, etc... created
- `docker-compose build`: Builds or rebuilds services
- `docker-compose logs`: View output

***1-4 Document your docker-compose file.***

```
version: '3.7'

services:
  backend:
    build: ./path/to/backend/Dockerfile
    networks:
      - my-network
    depends_on:
      - database

  database:
    image: my-database
    networks:
      - my-network

  httpd:
    build: ./path/to/httpd/Dockerfile
    ports:
      - "80:80"
    networks:
      - my-network
    depends_on:
      - backend

networks:
  my-network:
```

***1-5 Document your publication commands and published images in dockerhub.***

- `docker login`: To login to dockerhub
- `docker tag my-db bribrix/my-db`: Tag the image
- `docker push bribrix/my-db`: Push the image to dockerhub

---

---
## $ TP 2 - Discover Github Actions

Historically many companies were using Jenkins, it is way less accessible than GitHub Actions but much more configurable. The goal of this TP enables to discover the process of setting up a continuous integration and deployment (CI/CD) pipeline for a software application using GitHub Actions, Docker and SonarCloud. The aim is to demonstrate the ability to automate tests, builds and code quality analysis to facilitate frequent and reliable deliveries.

***Tools***

In order to do that, we had to Dockerhub to conterize, Github for the Gits Actions and SonarCloud to evaluate the quality of our code. 
We've set up a .yml workflow to automate application tests and builds with every push and pull request on the specified branches. The actions are defined to run an Ubuntu environment and use JDK 17 for the Maven build.


***2-1 What are testcontainers ?***

-> Testcontainers are Java libraries that allow running Docker containers for testing

***2-2 Document your Github Actions configurations.***

- `main.yml`
```
- branches: [ main ]
- name: Set up JDK 17
    uses: actions/setup-java@v3
    with:
      java-version: '17'
      distribution: 'temurin'
- name: Build and test with Maven 
  run: mvn clean install --file simple-api/pom.xml
```

---
## $ Third Part - Ansible

***3-1 Document your inventory and base commands***

- `/ansible/inventories/setup.yml`

This file is used to define the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.
```
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: /home/briac/id_rsa
  children:
    prod:
      hosts: briac.marchandise.takima.cloud
```

- `ansible all -i inventories/setup.yml -m ping`

This command is used to ping all the hosts in the inventory file.

***3-2 Document your playbook***

- `install_docker.yml`: This playbook is used to install docker on the host.

```
  ---
  - hosts: all
  gather_facts: false
  become: true 

  roles:
    - docker

  tasks:
    - name: Install device-mapper-persistent-data
      yum:
      name: device-mapper-persistent-data
      state: latest

    - name: Install lvm2
      yum:
      name: lvm2
      state: latest

    - name: Add Docker repository
      command:
      cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

    - name: Install Docker
      yum:
      name: docker-ce
      state: present

    - name: Install python3
      yum:
      name: python3
      state: present

    - name: Install docker Python module with Python 3
      pip:
      name: docker
      executable: pip3
      vars:
      ansible_python_interpreter: /usr/bin/python3

    - name: Make sure Docker is running
      service:
      name: docker
      state: started
      tags: docker
```

### **Deploy your App**

***Document your docker_container tasks configuration.***

- `Database`

```
--- tasks file for roles/launch_database

- name: Run PostgreSQL Container
  docker_container:
  name: my-db
  image: bribrix/tp-devops-database
  networks:
  - name: my-network
```

- `API`
```
--- tasks file for roles/launch_app


- name: Run Simple-API Application Container
  docker_container:
  name: my-api
  image: luiar/tp-devops-simple-api
  networks:
  - name: my-network
```

- `Proxy`
```
--- tasks file for roles/launch_proxy

- name: Run HTTPD Container
  docker_container:
  name: my_httpd
  image: luiar/httpd-image
  published_ports:
  - "80:80"
  networks:
  - name: my-network
```

- `Network`
```
--- tasks file for roles/create_network

- name: Create a Docker network
  community.docker.docker_network:
  name: my-network
```

Unfortunately, I had few issues with the network of Efrei Paris. The adress https://briac.marchandise.takima.cloud/ was unreachable so it lost a lot of time and wasn't able to finish the TP before the AWS were killed. That's too bad

## Thanks for those lessons, that was hard but cool !


