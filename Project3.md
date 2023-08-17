# DOCUMENTATION FOR LAMP STACK IMPLEMENTATION PROJECT

## This project shows the implementation of the LAMP Stack. It covers essential aspects of the LAMP stack implementation such as the process of setting up a Linux Environment, configuring the Apache Web server, managing MySQL databases, and writing PHP code for server side functionality.

## Pre Installations and Dependencies

In order to execute this project successfully we need to first of all complete the following prerequisites:

1. Setup an AWS Account : [Create a free tier AWS Account](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) by following these [steps](https://repost.aws/knowledge-center/create-and-activate-aws-account).

2. Spin up an EC2 Instance with Ubuntu Server: Launch an EC2 instance by following [these steps.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) __IMPORTANT: From the Amazon Machine Image (AMI Image) tab, be sure to select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM) rather than the HVM version of Amazon Linux 2 Server as directed in the link.__

3. Download and Install an SSH client: Download and install [PuTTY](https://putty.org/) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

4. Establish connection with your EC2 instance: Connect to your EC2 instance via your SSH client by following [these instructions.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)

*After completing the necessary prerequisites, we proceed to execute the following steps to fully implement our LAMP Stack and deploy it in AWS Cloud:*

## STEP 1: Installing Apache and Updating the Firewall
