Migrating to the Cloud with Containerization Part 1. Docker and Docker Compose
==============================================================================

Until now we have been using AWS EC2 instances in Amazon Virtual Private Cloud (AWS VPC) to deploy our web solutions. While this approach is straightforward and scalable, we can envisage some potential issues in a scenario where there is need to deploy many small applications (it can be web front-end, web-backend, processing jobs, monitoring, logging solutions, etc.) and some of the applications will require various Operating Systems and runtimes of different versions and conflicting dependencies - in such case we would need to spin up servers for each group of applications with the exact OS/runtime/dependencies requirements. When our use case scales out to tens/hundreds and even thousands of applications (e.g., when we talk of microservice architecture), this approach becomes very tedious and challenging to maintain.

In this project, we shall learn how to solve this problem and begin to practice the technology that revolutionized application distribution and deployment back in 2013! We are talking of Containers and imply Docker. Even though there are other application containerization technologies, Docker is the standard and the default choice for shipping an app in a container!

### <br>Introduction to Docker<br/>

Docker is an open-source platform that enables developers to automate the deployment, scaling, and management of applications within lightweight, portable containers. Containers package an application and its dependencies together, ensuring consistency across multiple environments from development to production.

#### Key Concepts

**i.** **Containers:** These are Lightweight, standalone, and executable packages that include everything needed to run a piece of software, including the code, runtime, system tools, libraries, and settings. Containers ensure consistency across environments, faster deployment, and efficient resource utilization.

**ii.** **Images:** These are immutable templates that create containers. They include the application code, libraries, dependencies, and other filesystem content needed to run the application. They are built from a Dockerfile using the docker build command.

***iii.** Dockerfile: This is a text file containing a series of instructions on how to build a Docker image. It specifies the base image, application code, dependencies, environment variables, and commands to run.

**iv.** **Docker Hub:** This is a cloud-based repository where Docker users can store and share Docker images. It allows users to upload their images and use images created by others.

#### Advantages of Docker

**i.** **Portability:** Applications run consistently on any environment where Docker is installed, eliminating the "works on my machine" problem.

**ii.** **Efficiency:** Containers share the same OS kernel, which reduces overhead and improves performance compared to traditional virtual machines.

**iii.** **Isolation:** Each container runs in its own isolated environment, ensuring that applications do not interfere with each other.

**iv.** **Scalability:** Easily scale applications horizontally by adding more container instances.

**v.** **Rapid Deployment:** Containers can be started quickly, speeding up the development and deployment process.

#### Use Cases for Docker

**i.** **Microservices:** Each microservice can run in its own container, making it easier to manage, scale, and deploy.

**ii.** **Continuous Integration and Continuous Deployment (CI/CD):** Docker simplifies the build, test, and deployment process by providing consistent environments for each stage.

**iii.** **Development Environments:** Developers can create and share reproducible development environments.

**iv.** **Cloud Migration:** Applications packaged in containers can be easily migrated to different cloud providers.

**v.** **Batch Processing:** Containers can be used for running batch jobs and data processing tasks in a controlled and reproducible manner.

### <br>Install Docker and prepare for migration to the Cloud<br/>

First we need to install [Docker Engine](https://docs.docker.com/engine/install/) which is a client-server application that contains:

- A server with a long-running daemon process dockerd.
- APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
- A command-line interface (CLI) client docker.

### MySQL in container

Let us start assembling our application from the Database layer - we will use a pre-built MySQL database container, configure it, and make sure it is ready to receive requests from our PHP application.

#### Step 1: Pull MYSQL Docker image from [Docker Hub Registry](https://hub.docker.com)

**i.** We begin by pulling the appropriate [Docker image for MySQL](https://hub.docker.com/_/mysql):

**`docker pull mysql/mysql-server:latest`**

**ii.** Then we subsequently enter the following command to list the images and check that we have downloaded them successfully:

**`docker images ls`**

#### Step 2: Deploy the MySQL Container to your Docker Engine

**i.** Once we have the image, we move on to deploying a new MySQL container with:

**`docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest`**

**Notes**
- Replace `<container_name>` with the name of your choice. If you do not provide a name, Docker will generate a random one
- The -d option instructs Docker to run the container as a service in the background
- Replace `<my-secret-pw>` with your chosen password
- In the command above, we used the latest version tag. This tag may differ according to the image you downloaded

**ii.** Then, we check to see if the MySQL container is running: Assuming the container name specified is **`mysql-server`**

**`docker ps -a`**

```
CONTAINER ID   IMAGE                                COMMAND                  CREATED          STATUS                             PORTS                       NAMES
7141da183562   mysql/mysql-server:latest            "/entrypoint.sh mysq…"   12 seconds ago   Up 11 seconds (health: starting)   3306/tcp, 33060-33061/tcp   mysql-server
```

Now we see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from `health: starting` to `healthy`, once the setup is complete.

#### Step 3: Connecting to the MySQL Docker Container

We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.

**Approach 1**

Connecting directly to the container running the MySQL server:

**`docker exec -it <container_name> mysql -uroot -p`**

Then we provide the root password when prompted. With that, we have connected the MySQL client to the server.

Finally, we change the server root password to protect our database.

**Approach 2**

First, we create a network:

**`docker network create --subnet=172.18.0.0/24 tooling_app_network`**

Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers we run. By default, the network we created above is of **`DRIVER`** **`Bridge`** is a requirement to control the **`cidr`** range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the **`--subnet`**

For clarity's sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.

We run the MySQL Server container using the created network.

First, let us create an environment variable to store the root password:

**`export MYSQL_PW=<root-secret-password>`**

Then, we pull the image and run the container, all in one command like below:

```
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest 
```

**Notes**

Flags used:

- **`-d`** runs the container in detached mode
- **`--network`** connects a container to a network
- **`-h`** specifies a hostname

If the image is not found locally, it will be downloaded from the registry.

Subsequently, we verify the container is running:

**`docker ps -a`**

```
CONTAINER ID   IMAGE                                COMMAND                  CREATED          STATUS                             PORTS                       NAMES
7141da183562   mysql/mysql-server:latest            "/entrypoint.sh mysq…"   12 seconds ago   Up 11 seconds (health: starting)   3306/tcp, 33060-33061/tcp   mysql-server
```

As you already know, it is best practice not to connect to the MySQL server remotely using the root user. Therefore, we will create an **`SQL`** script that will create a user we can use to connect remotely.

We create a file and name it **`create_user.sql`** and add the below code in the file:

```
CREATE USER '<user>'@'%' IDENTIFIED BY '<client-secret-password>';
GRANT ALL PRIVILEGES ON * . * TO '<user>'@'%';
```

Next, we run the script:

**`docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql`**




#### Connecting to the MySQL server from a second container running the MySQL client utility

The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.

Run the MySQL Client Container:

```
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u <user-created-from-the-SQL-script> -p
```

Flags used:

- `--name` gives the container a name
- `-it` runs in interactive mode and Allocate a pseudo-TTY
- `--rm` automatically removes the container when it exits
- `--network` connects a container to a network
- `-h` a MySQL flag specifying the MySQL server Container hostname
- `-u` user created from the SQL script
- `-p` password specified for the user created from the SQL script
 

#### Prepare database schema 

Now you need to prepare a database schema so that the Tooling application can connect to it.

1. Clone the Tooling-app repository from [here](https://github.com/darey-devops/tooling)

```
git clone https://github.com/darey-devops/tooling.git
```

2. On your terminal, export the location of the SQL file

```
export tooling_db_schema=~/tooling/html/tooling_db_schema.sql
```

You can find the `tooling_db_schema.sql` in the `html` folder of cloned repo.

3. Use the SQL script to create the database and prepare the schema. With the `docker exec` command, you can execute a command in a running container.

```
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema
```

4. Update the `db_conn.php` file with connection details to the database

```
$servername = "mysqlserverhost";
$username = "<user>";
$password = "<client-secret-password>";
$dbname = "toolingdb";
```

5. Run the Tooling App

Containerization of an application starts with creation of a file with a special name - 'Dockerfile' (without any extensions). This can be considered as a 'recipe' or 'instruction' that tells Docker how to pack your application into a container. In this project, you will build your container from a pre-created `Dockerfile`, but as a DevOps, you must also be able to write Dockerfiles. 

You can watch [this video](https://www.youtube.com/watch?v=hnxI-K10auY) to get an idea how to create your `Dockerfile` and build a container from it.

And on [this page](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/), you can find official Docker best practices for writing Dockerfiles.

So, let us containerize our Tooling application; here is the plan:

- Make sure you have checked out your Tooling repo to your machine with Docker engine
- First, we need to build the Docker image the tooling app will use. The Tooling repo you cloned above has a `Dockerfile` for this purpose. Explore it and make sure you understand the code inside it.
- Run `docker build` command
- Launch the container with `docker run`
- Try to access your application via port exposed from a container

Let us begin:

Ensure you are inside the folder that has the `Dockerfile` and build your container:

```
docker build -t tooling:0.0.1 .
```
In the above command, we specify a parameter `-t`, so that the image can be tagged `tooling"0.0.1` - Also, you have to notice the `.` at the end. This is important as that tells Docker to locate the `Dockerfile` in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the `Dockerfile`.

6. Run the container:

```
docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1
```

Let us observe those flags in the command.

- We need to specify the `--network` flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
- The `-p` flag is used to map the container port with the host port. Within the container, `apache` is the webserver running and, by default, it listens on port 80. You can confirm this with the `CMD ["start-apache"]` section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

**Note:** *You will get an error. But you must troubleshoot this error and fix it. Below is your error message.*

```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.18.0.3. Set the 'ServerName' directive globally to suppress this message
```
**Hint:** *You must have faced this error in some of the past projects. It is time to begin to put your skills to good use. Simply do a google search of the error message, and figure out where to update the configuration file to get the error out of your way.*

If everything works, you can open the browser and type `http://localhost:8085`

You will see the login page.

<img src="https://darey-io-nonprod-pbl-projects.s3.eu-west-2.amazonaws.com/project20/Tooling-Login.png" width="936px" height="550px">
The default email is `test@gmail.com`, the password is `12345` or you can check users' credentials stored in the `toolingdb.user` table.


