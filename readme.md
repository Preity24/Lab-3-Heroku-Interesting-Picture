# 95-702 Distributed Systems for Information Systems Management
# Lab 3 - Creating Containers and Deploying to the Cloud


___Docker___ is a technology for creating containerized applications - that is, your application runs inside a portable container that has all the elements needed to run that application. "Container" means a __process__ (running program) that executes on a machine. "Portable" means the container can be executed on your laptop or moved to another machine - like a cloud server. The container is isolated from other processes on the host machine, so if it crashes, it shouldn't take any other processes down. There are a few issues with isolation, though - for example, the ports that a regular process can access via sockets must be mapped from the container's internal (virtual) ports to the host's actual ports. Systems can be composed of multiple containers that typically use some other technology (like [Kubernetes](https://kubernetes.io/)) to talk to each other (instead of low level port access). See Docker's [documentation](https://docs.docker.com/) for more details. At the end of the lab, there's a reading assignment about Docker.

A Docker __image__ is an executable file created from a Dockerfile containing all the Docker commands needed to make the image plus a self-contained application. For example, a Web servlet must be packaged as a [war](https://en.wikipedia.org/wiki/WAR_(file_format)#:~:text=In%20software%20engineering%2C%20a%20WAR,that%20together%20constitute%20a%20web) file for use here. When the image is created, it will have an auto-generated name (like "admiring_tereshkova") - you can rename it if you'd like - and the image will have a separate UUID (unique identifier) and image name different than the auto-generated name.

The Docker commands used in the lab are all preceded by "docker"; they are case-sensitive; and when you create the Dockerfile, it ___must___ be in Linux-style format: no ".txt" on the end, and Unix line endings (see below for details).

This lab will get you to install Docker on your laptop, run Interesting Picture as a Docker image, then push that image to the cloud (we'll be using Heroku). As you work through the commands, be sure to reflect on them by asking yourself these questions: What is each command's purpose? What software is being used by this command? How does this fit into a Distributed Systems context?

### Warning! If you are running Windows Home Edition, upgrade to Educational or Pro before doing this lab!

## Part 1: Running a Docker Image Locally

### 1.1 Install Docker

This part of the lab installs the Docker daemon on your laptop so that you can run Docker containers there.

1. Go to

https://docs.docker.com/install/  

and Install Docker CE (Community Edition) according to your system.  Scroll down to find links for Mac and Windows downloads.

2. After installation, make sure you have the Docker daemon running in the background. Open a CMD window (Windows) or terminal (Mac or Linux).
Use the command

        docker ps

to check if it is running.  You should see table headers with empty rows:

        CONTAINER ID    IMAGE   COMMAND CREATED STATUS  PORTS   NAMES


3. Run the Hello World test image to see if you Docker runs correctly:

        docker run hello-world

4. If you see the following message, that means you have installed Docker correctly.


        Hello from Docker!

        This message shows that your installation appears to be working correctly.

        To generate this message, Docker took the following steps:
        1. The Docker client contacted the Docker daemon.
        2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
        3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
        4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

        To try something more ambitious, you can run an Ubuntu container with:
         $ docker run -it ubuntu bash

        Share images, automate workflows, and more with a free Docker ID:
         https://hub.docker.com/

        For more examples and ideas, visit:
         https://docs.docker.com/get-started/

The command they suggest trying runs a container with an Ubuntu (linux) command shell; there's no need to run it.

### 1.2 Creating a Custom Docker Container

1. Creating a war file (see https://www.jetbrains.com/help/idea/creating-and-running-your-first-java-ee-application.html)

- create a directory called "docker"; remember where it is.
- in IntelliJ, go to your InterestingPicture project and select File -> Project Structure.
	- click on Artifacts
	- click +, select Web Application:Archive, and select "For InterestingPicture: war exploded"
	- if the message "Library 'lib' required .. the artifact" and a Fix button appear, DO NOT CLICK on Fix.
	- click Create Manifest and agree to the suggested location
	- click OK in the Project Structure
	- choose Build -> Build Artifacts. Point to InterestingPicture:war and choose Build
- right click on it to bring up its Properties, where you can see its directory
  path (or choose Reveal in Finder on a Mac).
- you should see the war file in the Project tab under out -> artifacts -> InterestingPicture_war, named InterestingPicture_war.war
- copy the war file to your docker directory, but re-name it __ROOT.war__

2. Creating a custom Docker container using Dockerfile
- in the docker directory, copy the file named "Dockerfile" (note: capitalized)
  from github. Save it to your docker directory. This file contains Docker commands to create a docker container with openjdk12 and Tomcat. It then removes the default web app and copies your war file to the tomcat/webapps directory.

- save this file ??? but if you???re using Windows, do these two things:
-- make sure the file uses UNIX/Linux line endings. In Notepad++, use Edit > EOL Conversion > UNIX Format.
-- ***make sure the file does NOT have a .txt extension*** if you're
  using Windows. For example, in Notepad++, choose Save As, type the name in
  quotes as "Dockerfile", change the File Type from Text Documents (\*.txt) to
  All Files (\*.\*), and click Save.

### 1.3. Test your Docker image locally:

-In the **docker** folder, you should have the following structure:

    docker/
    ????????? Dockerfile
    ????????? ROOT.war

- from the docker directory in a terminal or CMD window, build the docker image with the command (note the space and period at the end); this will take a few seconds, and you'll see docker output scrolling by.

        docker build -t interesting_picture .

- you can see all of your images by running:

        docker images

It will display something like this (details will vary):

        REPOSITORY            TAG       IMAGE ID        CREATED          SIZE
        interesting_picture   latest    861ece9f7deb    2 minutes ago    108MB

- deploy the image in a container locally. The -p flag (for "publish") maps the actual port on
  your machine to the Docker container's port. If 8080 is in use on your laptop, change the first number to something else (say, 9000) and use that number in the browser instead.

        docker run --rm -it -p 8080:8080 interesting_picture

- open a browser window with the address:

        http://localhost:8080/

- you should see your app running. Test it to make sure it works correctly (again, use the port number from the run command if 8080 didn???t work for you). After showing the running app to your TA, kill the program in the CMD or terminal window with ctrl-C.

## Part 2: Running on the Cloud

Recall that the ___cloud___ is a fancy term for "someone else's servers". Some useful properties of cloud computing include not having to buy hardware, not needing to keep system software (like operating systems) up-to-date, not worrying about exactly where your application is running (although you should think about some possible downsides of that - for example, how is security handled in the cloud?), the ability to scale  - up or down - the number of servers in use based on current demand, and geographic placement to put servers closer to clients.

In this lab, you'll use [Heroku](https://www.heroku.com/), which has a free tier for small-scale use. Other commercial cloud providers include [Amazon AWS](https://aws.amazon.com/), [Alibaba Cloud](https://us.alibabacloud.com/en), and [Microsoft Azure](https://azure.microsoft.com/en-us/).

As with Docker, the Heroku commands start with "heroku", sometimes have flags (starting with a single or double -), are case-sensitive, and use generated names for your application. The process is to **create** a Heroku app, **push** your Web servlet to it, **release** it, then **open** it (run it). Make sure you follow the directions carefully - in particular, make sure your .sh file is correct. The push may be slow, so be patient - and this is another reason to be careful: you don't want to repeat the commands.

### 2.1 Get started with the Heroku cloud provider
1. Create a Heroku account: Go to https://signup.heroku.com/login and register
an account. You may choose your role as Student.  

2. Follow the instructions and install heroku-cli on your system. Note that you must have Git installed in your system.  See https://devcenter.heroku.com/articles/heroku-cli

3. This website also contains the link to configure Git if you don't have it.
- Git installation
- First-time Git setup

4. After installation, open your terminal or CMD window and type

        heroku login

5. Press enter and it will prompt and navigate you to the browser. Then click log in. Your computer should have access to Heroku services, including a dashboard (although the steps below use the Command-Line Interface).

### 2.2 Creating and Pushing a Container

1. Create a new directory named "heroku" (just to keep it separate from the local Dockerfile
above in the docker directory), copy the Dockerfile from part 1 and make the following two changes ??? see the note above about Windows files: this needs to be a text file without the .txt extension, and it must use UNIX/Linux line endings.

***Change # 1:***

Before the last line (which says CMD ["catalina.sh", "run"] ), un-comment the following lines (i.e. remove the hashtags):

        ADD tomcat_starter.sh /home/
        CMD chmod +x /home/tomcat_starter.sh; /home/tomcat_starter.sh

***Change # 2:***

Comment out the last line with a hashtag so that it looks like this:

        #CMD ["catalina.sh", "run"]

2. Create this bash shell script, a text file named ???tomcat_starter.sh???.

___Again, in Windows, follow the directions above about line endings; this is a text file,
but with the .sh extension, *NOT* .txt or .sh.txt.___

Note: the first line is ***required*** to begin with a hashtag; it's not a comment.

        #!/bin/bash
        # Change the configuration of Tomcat so that it listens to
        # the port assigned by Heroku
        sed -i s/8080/$PORT/ /usr/local/tomcat/conf/server.xml
        # delete the default ROOT directory so ROOT.war is used
        rm -rf /usr/local/tomcat/webapps/ROOT
        # start the server
        catalina.sh run

3. Copy the ROOT.war file from the docker directory (or from IntelliJ, again). So there should be three files in this directory: ROOT.war, Dockerfile, tomcat_starter.sh

4. Run this series of heroku commands, one at a time; this may take a few
minutes, so be patient:

        heroku container:login
        heroku create

**->** This will display an system generated app-name; the commands below use "serene-basin-70362",
but yours will differ: heroku assigns this name to your app; copy it carefully.

        heroku container:push web -a serene-basin-70362
        heroku container:release web -a serene-basin-70362
        heroku open -a serene-basin-70362

The InterestingPicture app should show up in your browser.

***
___Fall 2021 Warning___

The free tier of Heroku has a memory limit. The new version of Tomee + Linux exceeds that limit. If you get an error page after searching for a picture, that may be why. Try this: use the back button in your browser to get back to the request screen, and try again. If it still doesn't work, sorry - we've tried quite a number of different Tomee installations, but they're either too big or don't work with Servlet 5.0.

***

A few utilities if things go wrong:

        heroku apps # which apps you've pushed
        heroku apps:destroy serene-basin-70362 # you're only allowed 5 on the free tier
        heroku logs --tail -a serene-basin-70362 # if you get a heroku error message in the browser


## Part 3: Cloud and Containers Concepts

1. Read this article:

https://www.infoworld.com/article/3077875/linux/containers-101-docker-fundamentals.html

2. Read "A Descriptive Literature Review and Classification of Cloud Computing Research", journal pages 36 through 39 (PDF pages 3 through 6).

http://aisel.aisnet.org/cgi/viewcontent.cgi?article=3672&context=cais

You must be on campus, or using the campus VPN, to view this article.

3. Think about the answers to these two questions - potential test questions.

 i) Is a service like AWS an example of IaaS, Paas, or SaaS?

 ii) What property makes Docker containers suitable for version control?
