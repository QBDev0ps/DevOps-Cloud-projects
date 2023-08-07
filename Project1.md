# DOCUMENTATION FOR LINUX PRACTICE PROJECT

## This project shows the implementation of some command line operations in linux regarding aspects such as file manangement, directory navigation, user and permission management, package installation and network configuration.

## Pre Installations and Dependencies
Please note that the project is being executed with a windows system and to obtain a linux machine for the project, we need to download and install the following:
1. Oracle VM VirtualBox Manager:
Below is the outcome after downloading and installing VirtualBox from www.virtualbox.org

![Oracle VBox](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9451271b-d3f9-4261-b7d4-634d2d990bd4)

2. Ubuntu Linux: Below is the outcome after downloading the Ubuntu Linux Disc Image File and installing on the Virtual Machine i.e Oracle VirtualBox.
   
   ![ubuntu linux environment](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a4c9767c-26af-4393-a5a4-200cc228b43e)

3. Below shows the Linux Command Line Interface fully operational and ready for use:
   
   ![Linux terminal](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c305d9f6-237b-4ead-a80e-5a4f9d4b5976)
   

## LINUX COMMANDS

## 1. sudo Command
sudo is shortform for "superuser do". It enables the user perform tasks that require administrative rights.
To perform an administrative task such as installing the available upgrades of all packages currently installed on the system , you run the following command:

`sudo apt upgrade`

After running this command, the system will prompt us to authenticate ourselves with a password. Here, if we enter the password incorrectly, or the current user is not part of the sudo group (as shown in the image below), the system will log the activity as a security event.

![sudo error](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5cdc7ee4-a811-40ae-8ca6-aea998ab2dbb)

To fix the error in the image above, we can enter the command below and enter the root password to switch to the root user:

`su`

And then,  we can subsequently rerun the `sudo apt upgrade` command. Below is the correct output:

![sudo correct](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1d06bfb8-efee-4051-a870-39a0e9980b87)


## 2. pwd Command
This command shows the path of our current working directory. Enter the command as:

`pwd`

The output for this command is as shown below:

![pwd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/df80777c-cc7a-43a6-a077-1349e6da7e8b)


## 3. cd Command
We can use this command to change from our current working directory to another or we can use it to navigate to a folder by entering the command with the folder name (DevOps_folder) as shown below:

`cd DevOps_folder`

The image below represents the expected outcome:

![cd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/83510595-cd46-482c-bcd9-b2054b240ef6)

cd can also be used with flags. `cd -` to go to the last directory we were in. `cd ..` to go one directory up or we can enter `cd` alone to navigate back to the default home directory. The outputs of these commands should look like this: 

![cd flags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2be20299-5ddb-4052-910e-56934be6a074)





