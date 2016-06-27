---
date: 2013-08-20T13:23:00Z
title: "Jenkins CI Deployment with Bitbucket"
description: "Setting up Jenkins and Bitbucket for fun and adventure."
tags: [ "git", "jenkins" ]
Section: blog
categories:
  - "Development"
slug: "jenkins-ci-deployment-with-bitbucket"
---

I recently had someone ask me how to get started with Jenkins-CI, git, and ssh. My team is using it now for a CI-git deployment scheme but I was not the one to install or configure it. I didn't really want to let down my friend with an "I don't know, google it" answer so I spun up a new EC2 instance on Amazon AWS, installed Jenkins, pushed a new private repo to bitbucket, pulled and deployed that push on my Jenkins test server.

Setting up a t1.micro EC2 instance in Amazon AWS is simple enough so we can skip that.

Steps for installing Jenkins-CI can be found on its homepage so we can safely skip that as well. (Note: for this test I was not using jenkins behind a webserver proxy)

For the following steps I executed them as the jenkins user, you may do it another way if you wish, this is simply how I did it.

#### Create SSH Keypair

Creating an SSH keypair for use with jenkins and bitbucket:
From the server:

    ssh-keygen -b 2048 -t rsa -c "Jenkins Deployment Keys" -f /output/path/privatekey
    chmod 0600 /output/path/privatekey

Note: for my test I built the keys and configured git as the jenkins user

These sets of keys should be created with an empty password, for which you can simply press enter on the prompt.
Configure SSH

#### Create a config to map the host and identity files

    vim /var/lib/jenkins/.ssh/config

My config file looks like this

    Host jenkins-test-bitbucket
        HostName bitbucket.org
        User git
        IdentityFile /var/lib/jenkins/.ssh/privatekey
        IdentitiesOnly yes

#### Add SSH public key to bitbucket

At this point you have created your public and private keypair and created a config file to map the identity file to a host. Now, the public key needs to be added to your bitbucket account, the steps to accomplish this (should it change in the future) can be found on the bitbucket.org website but at the time of this writing you can

* Login to bitbucket.org
* Navigate to your repository
* Select the Gear icon
* Select Deployment Keys
* Click Add Keys
* Give it a witty label (or not)
* Copy and paste your public key
* Click Add Key

#### SSH Test Connection

Now, back to your server, you will need to approve the host. The easiest way is to simply try to connect.

    ssh jenkins-test-bitbucket

When prompted to continue the connection and add the host to your known hosts file type yes

If everything has gone smooth so far you should see the following output

    jenkins@localhost:/var/lib/jenkins$ ssh jenkins-test-bitbucket
    PTY allocation request failed on channel 0
    conq: logged in as myusername.

    You can use git or hg to connect to Bitbucket. Shell access is disabled.
    Connection to bitbucket.org closed.

Now that you have verified you can connect to bitbucket via ssh you need to install git, the jenkins git plugin, and configure the global settings for git.

#### Configure Git

Git: on debian / ubuntu you can simply

    apt-get install git

Git Plugin: the Jenkins-CI website maintains this file, download it and place it in the JENKINS_HOME/plugins directory (mine is located /var/lib/jenkins/plugins)

Configure git: you need to set the username and email address for your user
Example:

    git config --global user.email "jenkins@localhost"
    git config --global user.name "jenkins"

Obviously replace those values with what you like.

#### Configure the Jenkins Job

* Login to your Jenkins website
* Create a new job
* Configure the job
* Select Git for your SCM option
* The git repo address will be in a format such as this jenkins-test-bitbucket:username/path/to/repo.git
* Continue configuring (for example, if you have a shell script or ant script to run after you pull the repo that runs unit tests)
* Save
* BUILD!

If everything was successful and the planets are aligned you should see your repo pull and if you configured any scripts they should run.

I'm sure there are others that have a better, faster way to accomplish this but this is simply how I did it and hopefully this helps someone.
