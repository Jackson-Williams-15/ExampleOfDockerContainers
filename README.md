# Docker Containers Introduction For Local Web App Development

## Downloading Docker and MySQL
- For this example, I recommend downloading MySQL if you haven't already. 
-  You need to download [Docker](https://www.docker.com/get-started/). I recommend downloading [Docker Desktop](https://www.docker.com/get-started/) as well because it provides nice visuals of the containers and it makes it easier to start and stop the containers.

## What is Docker and Why Use it?
- Docker is a tool that helps developers run applications in isolated environments called **containers**. For local development, Docker makes it easier to set up everything you need to run your app, from web servers to databases.
1. **Containers**
    - A container is like a box that holds everything your application needs to run: the code, settings, dependencies, and even system tools. The container runs independently of your computer’s operating system, which means it can work the same way on any machine.
    - Like in this example repository, if you have a web app that runs on .NET with a MySQL database, you can put your app into a container, and it should run the same no matter where you launch it.
2. **Images**
    - An image is like a blueprint for a container. It's a pre-built, ready-to-run package that includes your application and everything it needs. When you start a container, Docker uses an image as the starting point.
    - For a MySQL database, there's a MySQL image that includes MySQL’s database software. When you run a container from this image, MySQL will start up in that container, ready to be used by your application.
3. **What is a Dockerfile**
    - Automates Environment Setup:
        - Instead of manually setting up everything your application needs, the Dockerfile automates this entire process.
        - When you run the Dockerfile, Docker builds an image according to the steps you define, making sure that the environment is always the same.
    - Builds a Custom Docker Image:
        - A Dockerfile is used to create a Docker image specifically for your application. This image can include the app’s code, system dependencies, libraries, and tools.
        - Once the image is built, it can be run anywhere using Docker, which makes sure there is consistency across environments.
    - Reproducibility:
        - A Dockerfile ensures that your development environment are the same. This consistency reduces the chances of the "it works on my machine" problem.
        - Any teammate can build the same image using the Dockerfile, which means that everyone runs the app in the same environment.
4. **Why Docker is Useful For Local Development**
    - In web development, you often need several tools and services to run your app locally, like:
        - A web server (to serve the app).
        - A database.
        - An API.
    - With Docker, you can:
        - Package everything the app needs into containers.
        - Share the same setup with the **whole** team.
        - Make sure that the app runs consistently.
5. **How Docker is Used For Databases**
    - For local development, Docker makes it very easy to run a database in a container. Instead of installing a database like MySQL or PostgreSQL on your computer, you can simply run it in a container. Although I still recommend downloading the database you would like to use.
    - **Example**: You can use a MySQL Docker image to run a MySQL database locally. All the data is stored in the container, and you can access the database **as if** it were installed directly on your machine. When you're finished, you can stop the container, and it won't leave any data behind on your machine unless you want to save the data.
6. **How Docker is Used For APIs**
    -  With Docker, you can run an API in its own container, separate from the database and web server.
    - **Example**: Lets say we have a .NET API that handles user logins, you can package it into a container using Docker. You and your team can run the API in a Docker container locally, test it, and make sure it works before deploying it to production (for our project, everything can stay local).
7. **Using Docker Compose For Multiple Services**
    - In web apps, you need multiple components to work simultaneously: a web server, a database, and possibly some background services. Docker helps you manage this with Docker Compose.
    - Docker Compose is a tool that lets you define all the containers your app needs in a single file (docker-compose.yml). With one command `docker-compose up`, you can start all the containers, and they should work together
8. **Example WorkFlow**
    - Let’s say we are building a simple web app with:
        - A front-end (HTML, CSS, JavaScript).
        - A back-end API (built with .NET or Node.js).
        - A MySQL database for storing data.
    - With Docker:
        - We would create a container for the API.
        - We would run a MySQL container for the database.
        - We would define how these containers work together in a docker-compose.yml file.
        - The whole team could run the entire app locally with one command: `docker-compose up`.
    - Now, everyone is running the same environment!
## [Docker-compose file](./DockerContainerExample/docker-compose.yml)
- Creates a MySQL container using the latest version of MySQL.
- Exposes port 3307 for MySQL.
- Configures MySQL with a default database, user, and password using environment variables.
- Uses Docker volumes to persist MySQL data.
- The app service builds and runs your .NET Core application using the Dockerfile.
- The ```depends_on: db``` makes sure that the MySQL database container is up and running before the .NET application starts
- The network means that both containers (MySQL and .NET app) can communicate with each other.

## [DockerFile](./DockerContainerExample/dockerfile)

### Build Phase
- ```FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build```: This uses the .NET SDK to build the project.
- ```COPY```: It copies the necessary project files (like .csproj) and the rest of the source code to the container.
- ```RUN dotnet restore```: It restores dependencies.
- ```RUN dotnet publish```: This builds and publishes the app in Release mode to the /out directory.

### Run Phase
- ```FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime```: This uses the .NET Core runtime image to run the app.
- ```COPY --from=build```: This copies the published application from the build stage to the runtime stage.
- ```EXPOSE 80```: Exposes port 80 to allow external access to the app.
- ```ENTRYPOINT```: Starts the .NET application with the built .dll file

## Environment 
- The ```.env``` file is meant to store sensitive information like, passwords, database name, and user credentials. Typically, your connection string would go here as well, but for this example the connection string will be in the [```appsettings.json```](./DockerContainerExample/appsettings.json) for simplicity. 
- The ``.env`` file is typically in a gitignore file so it is not published to the cloud repository.

## Connection String
- ```"ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Port=3306;Database=MoviesDB;User=team_user;Password=TeamPassword;" }
This is what your connection string looks like if placed in the ```appsettings.json``` file. This will allow your .NET application to connect to the MySQL container running on localhost with the correct credentials.
- The set up is slightly different if the connection string is set in the ```.env``` file. 

## Docker Setup/Commands
- When you have pulled the repository containing the docker-compose.yml file, create your own ``.env`` and enter the correct environment variables (ensure the ``.env`` file is in the root folder of the project). They can start the MySQL container using the following command:
    - ``docker-compose up -d``
    - This command will:
        - Pull the MySQL image (if it's not already available locally).
        - Start the container with MySQL running and the environment variables for the database, user, and password in place.
        - Starts the containers defined in the ``docker-compose.yml`` file in the background.
        - The command immediately returns control to the terminal, and your containers keep running in the background.
        - __Note__: You do not necessarily *need* the `-d` this option simply means "detached mode" which means the containers will run in the background, freeing up the terminal for other tasks. 
        If you want to see the logs within the terminal for development or troubleshooting, you can run ``docker-compose up`` instead. The logs can also be viewed within the Docker Desktop app.
- Once the containers are downloaded to your machine you can use Docker Desktop to see if they are on your machine and running, or run ``docker ps`` in the terminal and it will show you a list of Docker containers on your machine. 
- ``docker-compose up --build``
    - This command will:
        - Rebuild and start the containers: This will confirm that any changes to your Dockerfile or application code are reflected in the container. It forces a rebuild of your services (both the MySQL container and the .NET container in this case)
            - __Note__: You can add `-d` at the end to not have the logs appear in the terminal.
        - Automatically pulls new images if needed: It downloads the container images from Docker Hub if they are not already present on your machine or if there are updates to the image.
- ``docker-compose down``
    - This command will:
        - Stop and Remove Containers: It stops and completely removes all containers defined in the docker-compose.yml file, including any networks and containers they are attached to.
        - Remove Networks: It removes the default network that Docker Compose creates for your services.
    - This can be used before ``docker-compose up --build`` for ensuring a clean slate and when there are changes in your Docker Compose configuration.
        - __Ensuring a Clean Slate__
            - When to Use: If you want to make sure you start from a completely clean environment.
            - Over time, your Docker containers, networks, and volumes might accumulate, and there could be old configurations still lingering around. Running ``docker-compose down`` will make sure all these resources are removed before you start fresh with a new build.
        - __Changes in Docker Compose__
            - When to Use: If you've made significant changes to your docker-compose.yml file.
            - docker-compose up --build will rebuild images, but it won’t remove old networks or containers that aren't being used anymore. `docker-compose down` ensures that all old containers, networks, and volumes are removed, preventing conflicts between old and new configurations.
- ``docker exec -it <container-name> mysql -u<user> -p<user-password>``
    - This command will allow you to:
        - Access the MySQL shell: You can log into MySQL directly from inside the running Docker container and start executing SQL commands like how you would in a normal MySQL environment.
        - Manage the MySQL database: Once inside the MySQL shell, you can:
            - Create databases, tables, and users.
            - Query the database.
        - Debug or inspect: Useful for checking the state of the database or running manual queries when you want to debug something in a live environment or development environment.
        
    - __Breakdown__:
        - `docker exec`: This command is used to execute a command inside a running Docker container. It runs a command in an already running container so make sure the containers are running first.
        - `-it`: This flag makes the session interactive and attaches the terminal to the container. It means you're running the command in an interactive mode (useful for a MySQL session).
            - `-i`: Stands for "interactive", keeping the input open.
            - `-t`: Allocates a pseudo-terminal, which allows you to interact with the shell or command line program (like MySQL).
        - `<container-name>`: This should be the name or ID of your MySQL container. You would replace ``<container-name>`` with the name of your container.
        - `mysql`: This is the MySQL command-line client. It runs the MySQL shell within the container.
        - `-u<user>`: This specifies the MySQL user you want to log in as. The `-u` flag is used to specify the user. 
        - `-p<user-password>`: This specifies the password for the MySQL user. The `-p` flag is used to pass the password. Replace ``<user-password>`` with the actual password for the specified user of your MySQL instance. 
            - ___Note___: Be careful with this, this will show your password on the CLI. To hide you password, use just `-p` instead and it will prompt you for your password. ``docker exec -it <container-name> mysql -u<user> -p``
        - **Examples**:
            - In this repository (using the user and password from the .env file) the command would look like:
                - ``docker exec -it mysql_db_container mysql -uteam_user -pTeamPassword``
                - This would give you access to the database but not as the root user.
            - If you wanted to log in as a root user, the command would look like:
                - ``docker exec -it mysql_db_container mysql -uroot -p<root-password>``
