## DOCUMENTATION FOR IMPLEMENTING WORDPRESS WEBSITE WITH LVM STORAGE MANAGEMENT

This project demonstrates how to build and manage a scalable WordPress website using AWS EC2 and LVM (Logical Volume Management) storage. The project will delve into the intricacies of LVM storage management on Ubuntu such as the processes of creating logical volumes, managing disk space and accomodating changing storage requirements. And it will also cover performance optimization techniques and best practices for managing and securing WordPress installation in the AWS cloud.

### <br>Introduction to Implementing WordPress Website with LVM Storage Management<br/>

This project consists of two parts:

1. Configure storage infastructure on two Linux OS based (Web and Database) servers.
   
2. Deploy the Web and Database tiers of a basic web solution by installing WordPress and connecting it to a remote MySQL database server.

WordPress is a free and open-source content management system written in **PHP** and paired with **MySQL** or **MariaDB** as its backend Relational Database Management System (RDBMS)

In terms of deploying tiers of a web solution, the **Three-tier Architecture** is generally used when implementing a web or mobile solution.

The **Three-tier Architecture** is a client-server architecture pattern that consists of three seperate layers.

![3 tier](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1ec51fa2-569f-4155-8310-1a24f9e80ef6)

**1. Presentation Layer (PL)**: This is the user interface and communication layer of the application, where the end user interacts with the application. This can for instance be the client server or browser on your machine. Its main purpose is to display information to and collect information from the user.

**2. Business Layer (BL)**: Also known as the Application Layer or the Logic Layer. This is the heart of the application. In this layer, information collected in the presentation tier is processed - sometimes against other information in the data layer - using business logic, a specific set of business rules.

**3. Data Access Layer (DAL)**: Also known as the Management Layeror Database Layer. This is where the information processed by the application is stored and managed. This can be a Database server such as PostgreSQL, MySQL, MariaDB etc. or  a File System Server such as FTP Server, NFS Server etc.

In the execution of this project, our three tier architecture shall be:

1. A Laptop or PC to serve as a Client.
   
2. An EC2 Linux Server to serve as a Web Server that will host our WordPress website.
   
3. An EC2 Linux Server to serve as our Database Server.
