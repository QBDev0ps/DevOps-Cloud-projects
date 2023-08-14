# DOCUMENTATION FOR GIT PROJECT

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


## Working with Branches

Creating a git branch helps us to make a different copy of our source code i.e. a different copy of the main branch. By doing this, we can make edits and changes in our new copy as we please and these changes will remain independent of the original copy of our source code. This is an especially useful collaboration tool for programmers as different team members can create different branches to make changes or work on additional features and then merge all the changes to the main branch when all the work is done.

To make our first git branch which will be named my-new-branch, we execute the following command: 

`git checkout -b my-new-branch`

As shown in the output image below, the -b flag helps us to create and change into our new branch:

![mynewbranch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/973db837-9384-46c6-bb33-4ef7bbba4042)

To see all the branches in our local git repository, we enter the command below:

`git branch`

As we can see in the image, the command lists all the branches and identifies the branch we are currently in with an asterisk symbol.

![git branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ab00b3cf-0275-417a-a47f-80f0504a25d5)

To change back into an existing or old branch, for instance our 'main' branch, we simply execute the following command:

`git checkout main`

We can also merge a branch into another branch. To demonstrate this, we shall merge our branch titled 'my-new-branch' in to our 'main' branch. Please note that we made some changes to 'my-new-branch' by adding a line of text to the index.txt file as shown in the image below:

![index merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/063f54dd-0cf0-4a45-912f-da2720289280)

After adding the new line of text, we add our changed file to the staging environment with `git add .` and after this, we commit the change with `git commit -m "Added another line of text"`. 

To merge the edited index.txt file in 'my-new-branch' into the file in our unedited 'main' branch, we firstly switch to our 'main' branch with `git checkout main`  and then we execute the following command:

`git merge my-new-branch`

The image below shows the output of the merge process described above:

![git merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/21a81f5d-20ae-488e-bedb-20ffb0847358)

Now that we have merged the changes inside 'my-new-branch' into 'main' branch,  we can delete 'my-new-branch'. To execute this, we enter the following command:

`git branch -d my-new-branch`

Subsequently, when we enter `git branch` as shown in the image below, we can see that 'my-new-branch' has been deleted and no longer exists.

![git delete branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/de16ab50-a4b7-4d1f-8b9b-e7309511d25f)


## Collaboration and Remote Repositories

Github is a web based platform where git repositories are hosted. It enables collaboration between remote web or application development teams as they can work and make changes on different branches of the same code base in the same web based repository.

### To create a Github account, we carry out the following steps:
