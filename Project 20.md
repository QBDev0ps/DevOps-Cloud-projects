Migrating to the Cloud with Containerization Part 1. Docker and Docker Compose
==============================================================================

Until now we have been using AWS EC2 instances in Amazon Virtual Private Cloud (AWS VPC) to deploy our web solutions. While this approach is straightforward and scalable, we can envisage some potential issues in a scenario where there is need to deploy many small applications (it can be web front-end, web-backend, processing jobs, monitoring, logging solutions, etc.) and some of the applications will require various Operating Systems and runtimes of different versions and conflicting dependencies - in such case we would need to spin up servers for each group of applications with the exact OS/runtime/dependencies requirements. When our use case scales out to tens/hundreds and even thousands of applications (e.g., when we talk of microservice architecture), this approach becomes very tedious and challenging to maintain.

In this project, we shall learn how to solve this problem and begin to practice the technology that revolutionized application distribution and deployment back in 2013! We are talking of Containers and imply Docker. Even though there are other application containerization technologies, Docker is the standard and the default choice for shipping an app in a container!

### <br>Introduction to Docker<br/>

Docker is an open-source platform that enables developers to automate the deployment, scaling, and management of applications within lightweight, portable containers. Containers package an application and its dependencies together, ensuring consistency across multiple environments from development to production.

#### Key Concepts

**i.** **Containers:** These are Lightweight, standalone, and executable packages that include everything needed to run a piece of software, including the code, runtime, system tools, libraries, and settings.Containers ensure consistency across environments, faster deployment, and efficient resource utilization.

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
