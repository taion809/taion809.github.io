---
date: 2013-11-09T01:31:43Z
title: "Docker: Containing Madness"
description: "Use Docker to containerize crazy legacy software."
tags: [ "docker" ]
Section: blog
categories:
  - "Development"
slug: "docker-containing-madness"
---

Recently we had one of our important clients come to us with a node.js application they needed updated.  I'm not sure of the circumstances surrounding this application but it looked and felt like a ball of spaghetti was thrown in my face.  I punted the modifications to another developer in favor of doing the deployment with docker and supervisord.

### The Problem
This node.js application has many assumptions about libraries and file locations that make me cringe, for example it simply won't work unless packages are installed globally **and** locally.

### The Solution
Build a docker image containing the application so that it could be deployed without touching anything outside of its container, think like a chroot.

### Let's Do It!
The application we will be using can be found in my [github repository](https://github.com/taion809/hello-node).

The following Dockerfile will be used to build our image.  We are using Dockerfiles because it  makes building images simple, versioned, and easily distributable.

Our Dockerfile
{% gist 7380804 Dockerfile %}

In order to build our image from this Dockerfile we will execute the following command.

    sudo docker build -t='johnsn/webapp' . 

You should see output like the following

    # Excerpt
    Step 14 : ENTRYPOINT ["/usr/bin/nodejs"]
     ---> Running in 0db9a210f33c
     ---> a82cdefae38d
    Step 15 : CMD ["server.js"]
     ---> Running in c11ec2908881
     ---> 439913416e8f
    Successfully built 439913416e8f


You can now make the choice to push your image to the docker.io registry, a private registry, or not push at all.  If you chose to push it to a registry you will need to use the docker push command.

From here use docker run to create and start a container so we can verify the build worked.

    sudo docker run -d -p 5000:80 johnsn/webapp

You should receive output like the following, we used the -p flag to map our host port 5000 to the container port 80 which we exposed in the Dockerfile

    # Your container ID will be returned
    3f1a9b9ac4e4

Verify with curl

    curl -v localhost:5000

At this point we have a web application running inside a docker container we can serve on port 5000.  But, we have a problem, after a reboot we need to manually run the docker run command to bring up our web app again.  Let's solve that problem with supervisord.

Supervisord is really simple and convenient which is why I am using it over something like daemontools.

    [program:webapp]
    command=/usr/bin/docker run -p 5000:80 johnsn/webapp
    autostart=true
    autorestart=true

Notice we do not use the -d flag, this is because Supervisord will run our container as a daemon.  

Now restart the supervisord service with `service` or using `supervisorctl`

    # sudo service supervisor restart does not work :\

    sudo service supervisor stop
    sudo service supervisor start

Verify

    ps aux | grep webapp

    # root      8964  0.0  0.7 186948  3600 ?        Sl   03:54   0:00 /usr/bin/docker run -p 5000:80 johnsn/webapp

<!-- Logging options here... sometime. -->

Alright, so let's setup nginx properly so we can have webapp.com instead of webapp.com:5000

    ## HTTP server.
    server {
        listen 80; # IPv4

        server_name webapp.com;

        ## Access and error logs.
        access_log /var/log/nginx/webapp.com_access.log;
        error_log /var/log/nginx/webapp.com_error.log;

        location / {
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header X-Forwarded-Server $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:5000;
        }

    } # HTTP server

Restart the nginx and verify!

    curl -v webapp.com

That is it, 30 minutes later we have an immutable linux container for our web app that we can then deploy to a single server or many servers.  

Please let me know if you see errors, have corrections, or know a better way.

### References
* [Docker](http://docker.io)
* [Supervisord](http://supervisord.org/)
