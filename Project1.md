# DOCUMENTATION FOR LINUX PROJECT

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
sudo is shortform for "superuser do". It will enable you perform tasks that require administrative rights.
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


## 4. ls Command
Here, we use this command to list the files and folders in our system. Executing the command without any parameters shows the contents of our current working directory. We can achieve this by simply entering:

`ls`

The output we get is shown below:

![ls](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cb5d5ddb-4c4c-4cff-84b6-00cf22b6a3c1)

Wwe can also use ls to see the files in another directory by specifying the path. From root user, we can see the files in the home directory of vboxuser by entering: 

`ls /home/vboxuser`

Although we are still in the root directory, we are able to see the content of the specified path:

![ls root](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c01a1c84-4e4f-463a-9f78-4c5b9042a746)

The ls command has some flags that can extend its functionality. `ls -R` lists all the files in the sub directories as shown below:

![ls -R](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce7fa8dd-8c4e-4945-bfb1-096a66736e65)

As shown in the next image, entering the command: `ls -a` lists all the files along with the hidden files while `ls -lh` lists all the files in a more easily readable format whilst showing more detail.

![ls -a](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a3c0d49e-9115-4be4-aa1b-fbde2bf6ff2d)


## 5. cat Command
We can use this to list, combine or write a file's content on to the Command Line Interface. So to show what we wrote in the DevOps file we simply enter the following command:

`cat DevOps`

The output of this command is as shown in the image below:

![cat](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/81189f4f-a763-476e-94e9-8ee48e74e6bd)


## 6. cp Command
We use this to copy files or folders and their contents to another location. It is used by entering the command, follwed by the file name and then the destination directory. For instance, here we want to copy the DevOps file to the Desktop directory, so we enter the following command:

`cp DevOps Desktop/`

The command does not return any value on the CLI as the operation has already beeen carried out in the background but for confirmation, we can run the `ls` command to list the files in the desktop directory and in the image below, we can see that the DevOps file has been successfully copied.

![cp](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9ebe1975-5ce4-4c17-83a7-eec3a1fe855a)


## 7. mv Command
This is the command we will use to move or rename files.  We can use it by entering the command, followed by the filename and then the destination directory. Here we want to move the DevOps2 folder to the Desktop folder, so we enter the following command:

`mv DevOps2 Desktop/`

Just like the `cp` command, there is no output to the CLI when `mv` is executed. But we can use ls to see its effect as shown in the image below. We see that the DevOps2 file has been removed from it's original location in the home directory and its current location is now the Desktop folder:

![mv](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/73eb7d7a-0571-4ced-82ac-c2e353639994)

To show how the `mv` command is used to rename a file, we simply enter mv and specify the filename (DevOps2) and its path followed by the new filename (DevOps_New) and its path:

`mv Desktop/DevOps2 Desktop/DveOps_New`

As mentioned earlier, the command does not return any value but by using the `ls` command, we can see its output. DevOps2 has now been renamed to DevOps_New:

![mv2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5a552f09-e7b7-4154-a5bb-5e1b576beac3)


## 8. mkdir Command

This is a command we can use to create one or multiple directories at once. We can also use a flag to set permissions for the directories as they are being created. For usage, you need to type the command and specify the directory name to be created as shown in the image below:

`mkdir DevOps_folder`

The output for this can be viewed with the `ls` command:

![mkdir](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/095866aa-d026-4e50-bd7a-5a334bc33894)

To create a folder called inside_folder inside the DevOps_folder:

`mkdir DevOps_folder/inside_folder/`

To create a folder called second_inside_folder inside the inside_folder:

`mkdir DevOps_folder/inside_folder/second_inside_folder`

To view the output we can use `cd` to change to the created directory and then we use `ls` to view its content:

![mkdir3](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/67407461-594c-4d8c-902b-06cc08db42e8)

We can also use flags with this command. To create a directory named movies with full read, write and execute permissions for all users, we enter the following:

`mkdir -m777 movies`

The output for this is shown below:

![mkdir4](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0614b866-aa92-4930-a6a4-ca152f5f9619)

## 9. rmdir Command
