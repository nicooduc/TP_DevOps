## 1-1: Document your database container essentials: commands and Dockerfile.

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

