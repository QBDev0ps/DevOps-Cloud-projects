# Documentation for GIT Project

## This project shows the implementation of some core Git concepts and functionalities such as Initializing a Repository, Making Commits, Working with Branches, Collaboration and Remote Repositories and Tagging Track Changes

## Pre Installations and Dependencies

Please note that before executing this project we will need to download and install git on our computer sytem. Below is the image of the opened Git bash terminal which comes with the installation of Git:

![Git Bash terminal](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fbe0d0a9-d743-4c2d-858c-506a44cc9ae9)

## Initializing a Git Repository

To initialize our Git repository, we firstly need to create our working folder or directory which will be named DevOps. To do this, we will execute the following command in our opened Git Bash terminal:

`mkdir DevOps`

Next, we move into the created folder which will be our working directory with the following command:

`cd DevOps`

Then while inside the folder, we run the command below to initialize our repository:

`git init`

As shown in the output in the image below, we can see that our initialization process was successful.

![git init](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3b0787b0-5496-42c7-9eb0-b2ea2000abe3)


## Making your First Commit

It is important to note that we have 3 different environments in Git. We have the Working Files which is where we make edits to our files, the Staging area which is where our files are held before we commit. and lastly we have the Commit Environment.

Making a commit constitutes saving changes made to the files in our directory. These changes can be in form of adding, deleting or modifying files. It means a snapshot our repository is staken at the current state when we commit and saved in the .git folder in our working directory.

To make our first commit, we need to first of all create a text file that will be called index.txt inside our working directory. WE do this by executing the following command: 

`touch index.txt`

The above command creates the text file, then as shown below, we enter a sentence in to our index file via a text editor and then we save our changes:

![index text](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fd33fff7-c606-404b-8508-584d40e5cb43)

The next step is for us to add our changes to the git staging area via the following command:

`git add .`

After executing the command above, we now have our files being held in the staging environment. From here we proceed to commit by entering the following command:

`git commit -m "initial commit"`

As shown in the output seen in the image below, the commit process was successful. Also we used the -m flag so that we can include a message ("initial commit") to provide some clarity and context about the commit.

![commit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2d4d8d59-07d7-4965-8d5b-b688acb1beb7)
