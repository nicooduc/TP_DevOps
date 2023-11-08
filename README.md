## 1-1: Document your database container essentials: commands and Dockerfile.

---

### PostgreSQL Database Container Setup

This repository contains the essential files and instructions to set up a PostgreSQL database using Docker.

### Dockerfile:

```Dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

The Dockerfile sets up a PostgreSQL container based on the `postgres:14.1-alpine` image. It defines environment variables for the database.

### Building the Docker Image:

```bash
docker build -t database .
```

This command builds the Docker image and tags it as `database`.

### Creating a Docker Network:

```bash
docker network create app-network
```

Create a Docker network named `app-network` for inter-container communication.

### Running the PostgreSQL Container:

```bash
docker run -d --name database --network app-network database
```

Launch the PostgreSQL container with the specified configuration and connect it to the `app-network`.

### Running Adminer for Database Management:

```bash
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer
```

Start Adminer, a lightweight database management tool, to interact with the PostgreSQL database. Adminer is accessible at `localhost:8090` in your browser.

## 1-2: Why do we need a multistage build? And explain each step of this dockerfile.

---

A multistage build in Docker is used to optimize the size of the final Docker image. It involves using multiple `FROM` statements in a single Dockerfile to create intermediate images and copy only the necessary artifacts from one stage to another. This approach helps reduce the overall size of the final image by eliminating unnecessary build dependencies and intermediate files.

Let's break down the provided Dockerfile for the multistage build:

```Dockerfile
FROM eclipse-temurin:17-jdk-alpine AS builder
COPY Main.java /src/Main.java
WORKDIR /src
RUN javac Main.java

FROM eclipse-temurin:17-jre-alpine
COPY --from=builder /src/Main.class /app/Main.class
WORKDIR /app
CMD ["java", "Main"]
```

1. **`FROM eclipse-temurin:17-jdk-alpine AS builder`**: This sets up the first build stage using the `eclipse-temurin:17-jdk-alpine` base image. It's named as `builder` for reference.

2. **`COPY Main.java /src/Main.java`**: Copies the `Main.java` source code into the build context at `/src/Main.java`.

3. **`WORKDIR /src`**: Sets the working directory to `/src` where the Java source code is located.

4. **`RUN javac Main.java`**: Compiles `Main.java` to produce `Main.class` (Java bytecode).

5. **`FROM eclipse-temurin:17-jre-alpine`**: Starts the second build stage with a fresh base image, this time using the lightweight JRE variant.

6. **`COPY --from=builder /src/Main.class /app/Main.class`**: Copies the compiled `Main.class` file from the `builder` stage to the `/app` directory in the current stage. This is the key step in the multistage build, where only the necessary artifact is carried over to the final image.

7. **`WORKDIR /app`**: Sets the working directory to `/app` where the final Java class file is located.

8. **`CMD ["java", "Main"]`**: Specifies the default command to run when the container starts, executing the `Main.class` file using the Java runtime.

Explanation:
- The first stage (`builder`) compiles the Java source code to bytecode.
- The second stage uses a smaller base image and only copies the compiled `Main.class` file, eliminating unnecessary build tools and intermediate files.
- This approach results in a final image containing only the minimal required components to run the Java application, reducing its size and improving security by minimizing the attack surface.

In summary, the multistage build optimizes the Docker image by separating the build process into stages, allowing for a smaller and more efficient final image while still preserving the ability to compile and execute the Java code.

## 1-3: Document docker-compose most important commands.

---

1. **`docker-compose up`**: Builds, (re)creates, starts, and attaches to containers for services defined in the `docker-compose.yml` file. Use `-d` flag for running in detached mode.
   ```bash
   docker-compose up -d
   ```

2. **`docker-compose down`**: Stops and removes containers, networks, volumes, and images created by `docker-compose up`.
   ```bash
   docker-compose down
   ```

3. **`docker-compose build`**: Builds or rebuilds the services defined in the `docker-compose.yml` file.
   ```bash
   docker-compose build
   ```

4. **`docker-compose ps`**: Lists the containers for services defined in the `docker-compose.yml` file.
   ```bash
   docker-compose ps
   ```

5. **`docker-compose logs`**: Displays log output from services defined in the `docker-compose.yml` file.
   ```bash
   docker-compose logs
   ```

6. **`docker-compose exec <service-name> <command>`**: Executes a command in a running service container.
   ```bash
   docker-compose exec backend bash
   ```

7. **`docker-compose up --force-recreate`**: Recreates containers even if their configuration and image haven't changed.
   ```bash
   docker-compose up --force-recreate
   ```

## 1-4: Document your docker-compose file.

---

```yaml
version: '3.7'

services:
    backend:
        build: ./backendAPI
        networks:
            - app-network
        depends_on:
            - database

    database:
        build: ./database
        networks:
            - app-network

    httpd:
        build: ./httpServer
        ports:
            - "8080:80"
        networks:
            - app-network
        depends_on:
            - backend

networks:
   app-network:
```

Explanation:
- `backend`: Builds the backend service using the Dockerfile in the specified context, connects to the `app-network`, and depends on the `database`.
- `database`: Builds the database service using the Dockerfile in the specified context, connects to the `app-network`.
- `httpd`: Builds the HTTP server service using the Dockerfile in the specified context, maps port 80 to the host machine, connects to the `app-network`, and depends on the `backend`.
- `networks`: Defines a custom network named `app-network` for inter-container communication.

This Docker Compose configuration sets up the services and their dependencies, allowing them to communicate over the specified network while ensuring proper orchestration and dependency resolution.

## 1-5: Document your publication commands and published images in dockerhub.

---

1. **Connect to Docker Hub:**
   ```bash
   docker login
   ```

2. **Tag Your Image:**
   ```bash
   docker tag database USERNAME/database:latest
   ```
   This command tags your local Docker image (`database`) with a meaningful version (`latest`) and associates it with your Docker Hub username (`USERNAME`).

3. **Push Your Image to Docker Hub:**
   ```bash
   docker push USERNAME/database:latest
   ```
   This command pushes the tagged image (`USERNAME/database:latest`) to your Docker Hub repository.

## 2-1: What are testcontainers?

---

Testcontainers is an open-source Java library that simplifies integration testing by enabling developers to run Docker containers as part of their tests. This library provides reusable, pre-configured Docker containers for popular services such as databases (like PostgreSQL, MySQL, etc.) and many others. By using Testcontainers, developers can create, launch, and manage Docker containers directly from their unit and integration tests.

Here, Testcontainers is utilized to create and manage a Docker container running PostgreSQL. This container allows integration tests to accurately verify the insertion and retrieval of data within the PostgreSQL database. This approach ensures a consistent and reproducible testing environment, even when tests are run across different platforms or configurations.

## 2-2: Document Your GitHub Actions Configurations:

---

### GitHub Actions Configuration:

#### Workflow Description:

The GitHub Actions workflow is named "test-backend" and is triggered on every push to the `main` and `develop` branches and pull requests.

#### Jobs:

- **`test-backend` Job:**
   - **Runs on:** Ubuntu 22.04
   - **Steps:**
      - **Checkout Code:**
        ```yaml
        - uses: actions/checkout@v2.5.0
        ```
        Uses the `actions/checkout@v2.5.0` action to fetch the GitHub repository code.
      - **Set Up JDK 17:**
        ```yaml
        - name: Set up JDK 17
          uses: actions/setup-java@v3
          with:
            java-version: '17'
            distribution: 'adopt'
        ```
        Uses the `actions/setup-java@v3` action to set up JDK 17 with AdoptOpenJDK distribution.
      - **Build and Test with Maven:**
        ```yaml
        - name: Build and test with Maven
          run: mvn -B verify sonar:sonar -Dsonar.projectKey=nicooduc_TP_DevOps -Dsonar.organization=nicooduc-devops -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./backendAPI/pom.xml
        ```
        Executes the Maven build and test commands. This includes SonarQube analysis, with results sent to SonarCloud for evaluation.

#### Workflow Execution Steps:

- When code changes are pushed to the `main` or `develop` branches or when a pull request is opened, the workflow is triggered.
- The workflow checks out the code from the repository.
- It sets up JDK 17 using AdoptOpenJDK distribution for the build environment.
- The application is built and tested using Maven. SonarQube analysis results are sent to SonarCloud for continuous code quality evaluation.

#### Success Criteria:

- The workflow is considered successful if all steps are executed without errors, and the final status is displayed as "GREEN" in the GitHub Actions interface.
- Sensitive data, such as the SonarCloud token, is securely handled using GitHub Secrets (`secrets.SONAR_TOKEN`).

## 2-3: SonarCloud Quality Gate Configuration:

---

### Overview of Quality Gate Configuration:

- **Analysis Parameters:**
   - `sonar.projectKey`: Uniquely identifies the project in SonarCloud.
   - `sonar.organization`: Specifies the organization key in SonarCloud.
   - `sonar.host.url`: SonarCloud server URL.
   - `sonar.login`: Authentication token for SonarCloud (securely stored as a secret).

- **Analysis Process:**
   - Analysis is automatically triggered with every commit to the main branch of the repository.
   - SonarCloud conducts a comprehensive analysis of your codebase.
   - It evaluates various aspects of the code, including code duplication, code smells, bugs, vulnerabilities, and test coverage.

- **Quality Gate Criteria:**
   - SonarCloud applies predefined criteria from the quality gate to the analysis results.
   - If the code meets the specified quality standards, the quality gate passes, indicating acceptable code quality.
   - If the code does not meet the standards, the quality gate fails, indicating the presence of issues to be resolved before merging.

- **Continuous Monitoring:**
   - SonarCloud continuously monitors code quality, ensuring that new changes either maintain or enhance the overall code quality.
   - It prevents the introduction of code with low maintainability, security vulnerabilities, or other issues into the main codebase.

By configuring and utilizing SonarCloud within your GitHub Actions workflow, you establish a robust quality gate process that enhances the overall quality and maintainability of your codebase, leading to more reliable software development practices.

## 3-1 Documenting Updated Ansible Inventory and Base Commands:

---

### Updated Ansible Inventory (setup.yml):

```yaml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: id_rsa
  children:
    prod:
      hosts:
        - nicolas.duchene.takima.cloud
```

- **Explanation:**
   - `ansible_user`: Specifies the SSH user to connect to hosts (`centos` in this case).
   - `ansible_ssh_private_key_file`: Path to the private SSH key file for authentication (`id_rsa` in this case).
   - `prod`: Represents a group of hosts.
   - `hosts`: Lists the hostname under the `prod` group (`nicolas.duchene.takima.cloud` in this case).

### Base Ansible Commands (Using Updated Inventory):

1. **Ping Test:**
   ```bash
   ansible all -i inventories/setup.yml -m ping
   ```
   This command tests the connection to the host specified in the updated inventory.

2. **Gather Facts (Retrieve OS Distribution):**
   ```bash
   ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
   ```
   Uses the setup module to retrieve OS distribution information from the specified host.

3. **Remove Apache httpd Server:**
   ```bash
   ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
   ```
   Removes the Apache httpd server from the specified host. Ansible ensures the desired state of the server.

## 3-2 Documenting Ansible Playbook:

---

### Playbook Name: Deploy Application Stack

```yaml
---
- name: Deploy Application Stack
  hosts: all
  gather_facts: false
  become: true

  tasks:
    - name: Test connection
      ping:
      # This task tests the connection to all hosts in the inventory.

  roles:
    - docker
    - network
    - database
    - backend
    - httpd
    - front
```

- **Description:**
  This Ansible playbook automates the deployment of an application stack on multiple servers. The playbook includes tasks to test the connectivity to the hosts and roles for setting up Docker, creating a network, and deploying various components of the application stack.

## 3-3: Documenting docker_container Tasks Configuration:

### Role: Docker

```yaml
---
- name: Install Docker and Dependencies
  tasks:
    - name: Install device-mapper-persistent-data
      yum:
        name: device-mapper-persistent-data
        state: latest

    - name: Install lvm2
      yum:
        name: lvm2
        state: latest

    - name: add repo docker
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

    - name: Install docker with Python 3
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

- **Description:**
    - Installs Docker and its dependencies on the target servers.
    - Installs `device-mapper-persistent-data` and `lvm2` packages.
    - Adds Docker repository to the system configuration.
    - Installs Docker Community Edition (CE).
    - Ensures Python 3 is installed.
    - Installs Docker Python library using pip for Ansible.
    - Ensures Docker service is running.

### Role: Network

```yaml
---
- name: Create Docker Network
  tasks:
    - name: Create a network
      community.docker.docker_network:
        name: network_one
      vars:
        ansible_python_interpreter: /usr/bin/python3
```

- **Description:**
    - Creates a Docker network named `network_one` for communication between containers.

### Role: Database

```yaml
---
- name: Set Up Database Container
  tasks:
    - name: Create a data container
      community.docker.docker_container:
        name: database
        image: sylavana/database:latest
        networks:
          - name: network_one
      vars:
        ansible_python_interpreter: /usr/bin/python3
```

- **Description:**
    - Sets up the database container for the application using the `sylavana/database:latest` image.
    - Connects the container to the `network_one` Docker network.

### Role: Backend

```yaml
---
- name: Set Up Backend Server Container
  tasks:
    - name: Create a data container
      community.docker.docker_container:
        name: backend
        image: sylavana/backend:latest
        pull: true
        recreate: true
        networks:
          - name: network_one
      vars:
        ansible_python_interpreter: /usr/bin/python3
```

- **Description:**
    - Configures the backend server container using the `sylavana/backend:latest` image.
    - Pulls the latest image, recreates the container if necessary, and connects it to the `network_one` Docker network.

### Role: Httpd

```yaml
---
- name: Set Up HTTP Server Container
  tasks:
    - name: Create a data container
      community.docker.docker_container:
        name: httpd
        image: sylavana/frontend:latest
        ports:
          - "80:80"
        networks:
          - name: network_one
      vars:
        ansible_python_interpreter: /usr/bin/python3
```

- **Description:**
    - Configures the HTTP server container using the `sylavana/frontend:latest` image.
    - Maps port 80 on the host to port 80 on the container.
    - Connects the container to the `network_one` Docker network.