## DOCUMENTATION FOR DEVOPS TOOLING WEBSITE SOLUTION

In the [previous project](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project9.md), we implemented a WordPress solution that is ready to be filled with content and can be used as a blog or a fully fledged website. Subsequently, in this project, we will be adding some more value to our solutions by implementing a comprehensive DevOps tooling website solution that will integrate various tools and technologies to create a unified platform that enhances collaboration, automation and efficiency for software development and operations teams.

### <br>Introduction to DevOps Tooling Website Solution<br/>

The goal of this project is to build a tooling website solution which will enable easy access to DevOps tools within the corporate organisation. The solution we will implement will consist of the following components:

**1.** **Infrastructure**: Amazon Web Services (AWS)

**2.** **Web Server**: Red Hat Enterprise Linux 8

**3.** **Database Server**: Ubuntu 20.04 + MySQL

**4.** **Storage Server**: Enterprise Linux 8 + NFS Server

**5.** **Programming Language**: PHP

**6.** **Code Repository**: GitHub

Based on the infrastructure to be used and the goal of this project, it shall consist of three parts:

**1.** Configure Network File Server (NFS) Server with storage infastructure.

**2.** Configure Database Server.

**3.** Configure Web Servers.

As shown in the diagram below, we shall implement a solution where three stateless Web Servers will share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for the Web Servers it would look like a local file system from where they can serve the same files.

![3 TIER Tooling-Website-Infrastructure](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/61a271c9-4efd-45ff-a71e-94e62340faf8)

### <br>Configure Network File Server (NFS) Server with storage infastructure<br/>

