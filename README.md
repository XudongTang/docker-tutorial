# Introduction to Docker

## Motivation
Imaging you are working on a research project. To have your code running, you need several fancy packages from various sources. You might also need software distributions that require you to compile them yourselves. After hours of setting up, you finally got everything working on your computer.
Sometimes it is not feasible to run everything on your local computer. Maybe you have 5,000 models based on different dataset to run, and it would take months to get all jobs finished on your local device. In this case, you might want to run your code in a remote environment, an environment that could handle large number of jobs, such as in [CHTC].
Now, remote servers are quite different from local environments. Under most circumstances, servers are based on a Linux OS, not the Windows or MacOS that we are familiar with. You probably do not have administrator permission as well, so you need to follow the instructions the server maintenance team provides you. Those instructions are quite different for different servers.
After you spent hours setting everything up and get the jobs runnning, thinking that hours of hard work finally paid off, the job got terminated, and something like this appears in the error message:
```
configure:4314: $? = 4
configure:4303: gcc -qversion >&5
gcc: error: unrecognized command line option '-qversion'
gcc: fatal error: no input files
compilation terminated.
```
or maybe this message appears:
```
Can't open file of random numbers (/var/lib/condor/execute/slot1/dir_3431663/software/net/../util/randfile)
```
Different reasons lead to different error messages. For the first one, the GCC compiler version on the server might be different from the one required in one of your packages. For the second error, the server might change the pathing of your files with it runs your code on an execution node. You would need super user permission to install newer version of GCC, and you might need to do significant changes to the package you are using to deal with the pathing issue. Both are very time-consuming, if not even possible to fix.
In this scenario, a docker container would be your best hope. As long as the server you are using supoorts docker (which is true is most cases), it is almost guaranteed that the code you managed to run locally would run successfully on remote servers.

## What is a Docker container?
A Docker container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. Consider the container as a suitcase, you pack up all your belongings you need for daily life, and you will be able to survive at any part of the world. The container would contain code, runtime, system tools, system libraries and settings, everything you need to have the code run. As long as the remote enviroment supoorts Docker container, you could expect your code to run in a container on the server exactly the same as it runs on your local computer.
As you might see, the Docker container aims to solve the kind of issue that raised in the previous part. Despite being very powerful, creating a Docker container is fairly easy. In the next few section, I will show you step by step how to create your own container.

## Setting up
To create a container, you need two things: a DockerHub account and a Docker Engine. Let's start from the easy one: setting up the DockerHub.
### Getting a DockerHub account
To register for a DockerHub account, go to [DockerHub website] and follow the sign-up steps. The DockerHub is used as a distribution site. You would create a Docker image locally (A Docker images act as a set of instructions to build a Docker container), and then push it to DockerHub. When the server execute your code that should run in your container, it will pull the image from your DockerHub and build the container with it. More on this later, for now you just need to register for an account on DockerHub.
### Getting a Docker Engine
Installing a Docker Engine may vary on different operating systems. There are two options: you could either download a Docker Desktop for your system, or directly download a Docker Engine if you are running a Linux system locally. I would recommand the first option, The Docker Desktop is basically a user-interface warpped around a Docker Engine. It is easier to use, and better maintained. For Windows users, download Docker Desktop on the [Windows link]. For Mac users, you could find it on the [Mac link]. Note that if you are using a MacOS, you need to make sure if you are using an Intel Chip or Apple Chip. You need to download your version accordingly. For Linux users, Docker Desktop only supports Ubuntu, Debian, and Fedora. You could find the package and installation instruction on the [Linux link]. If you are not using those three systems, you should download the Docker Engine directly from the [Engine link] and follow the instruction on this page.
Follow the installation instruction on the pages. Under most circumstances you install Docker Desktop like any other software on your system, but it does allow command line installation. It takes a while for the Docker to start. For Docker Desktop, you could see an interface. There will also be a short tutorial for linking this UI to your DockerHub. Follow these steps and you are all done with setup! For Docker Engine, you won't see a window, you’ll just see a little whale and container icon in one of your computers toolbars.
To use build and test the Docker container on your laptop, you wonn't be using this UI. Instead, you will do everything via command in a terminal.
There are a few common issue that a Windows user will run into in my experience. I will list the solutions to save your time.
#### Windows Issues
You might get this error message:
```
Hardware assisted virtualization and data execution protection must be enabled in the BIOS.
```
To solve this, restart your device and enter the BIOS. The way to enter BIOS varies for different devices, common methods are pressing F12 or F10 repeatedly once powering on your computer. Once entering the BIOS, find "Hardware assisted virtualization" and enable it. You might not find the exact option. The option with "hardware" and "virtual" is usually the one you want to enable.
After this, you will most likely run into another error message:
```
WSL 2 installation is incomplete
The WSL 2 Linux kernel is now installed using a separate MSI update package. Please click the link and follow the instructions to install the kernel update:
https://aka.ms/wsl2kernel
Press restart after installing the Linux kernel
```
WSL stands for Windows Subsystem for Linux. Since you need to work on your docker via command, you need a linux environment, so having WSL is essential for windows users. Click the link in that error message and follow the instructions in that page to install WSL 2. You should not run into any issue while doing that.

## Getting your first image
Now you finish all the setting up steps, it's time to get an actual image that you could play around with. All docker images have the following format: username/iamge:tag. "Username" is the DockerHub user that you want to pull from, "image" is the name of the image in the user's repository, and "tag" is like the version of the image. Let's experiment with a Ubuntu image that everyone likes. Open your terminal (for Windows users, it's the WSL terminal you just installed, not the CMD prompt), and type
```sh
docker pull ubuntu:22.04
```
Since Ubuntu is an official image, there is no username associated with it. After all the data transfer finished, type
```sh
docker image ls
```
you should see this
```
REPOSITORY             TAG       IMAGE ID       CREATED       SIZE
ubuntu                 22.04     2dc39ba059dc   3 weeks ago   77.8MB
```
The image ID might be different. 
Now you have your first image! Note that it is just a base Ubuntu 22.04 image with absolutely nothing in it. Let's build a container on this image and explore it a bit. Type 
```sh
docker run -it --rm=true ubuntu:22.04 /bin/bash
```
The meaning of the flags are as follows:
- -it : interactive
- --rm=true : clean up the runnining container after exit.
- /bin/bash : start a bash(command prompt) when running the container

Now you should see a prompt like this:
```sh
root@3569d1bd2785:/# 
```
You are now in a container that has the base Ubuntu 22.04! The characters after the @ is the container ID. You should be able to do anything that you can do in an actual Ubuntu system. Try
```sh
root@3569d1bd2785:/# apt-get update
```
this should update all packages in this container. Try
```sh
root@3569d1bd2785:/# apt-get install wget
```
This will install wget command in this container. Now, keep in mind that everything you just did happens ONLY in this container. You are not installing wget on your local system! After you exit the container, these should be all gone! Let's try:
```sh 
root@3569d1bd2785:/# exit
```
This will return back to your local terminal. Everything in that container is now gone. Try building the container on the image again:
```sh
docker run -it --rm=true ubuntu:22.04 /bin/bash
```
This give you a entirely new Ubuntu Container! You could check that the container ID changed. Now if you do 
```sh
root@452616f4644a:/# wget
```
you will get 
```
bash: wget: command not found
```
even though you installed it in the previous container.
Now you should have a sense of what a container or the image that builds it look like. But we don't want another Ubuntu container, we want a unique container that runs our code! To build your own image and container, you should create a Dockerfile.

## Creating a Dockerfile
The Dockerfile gives you the freedom to customize the environment you need to run your code. Whether it is getting a different verison of compiler, or installing a package, you could get all of those down in this one file! To create a Dockerfile, use your favorite editor (for me it's VIM).
```sh
vim Dockerfile
```
Note the file name MUST be exactly "Dockerfile", no file type, no other fancy name. 
In the new Dockerfile, the first thing you want to do is to set up a base image. It should be some kind of official image that you could build your environment on. The Ubuntu:22.04 is a good option. To do this, you should use the "FROM" keyword. The first line of the file should thus be 
```
FROM username/image:tag
```
To use the Ubuntu 22.04 as base image, you should type
```
FROM ubuntu:22.04
```
Now you have your have image, you would want to add a few things on it. Maybe update all the packages and install a few more. In a Debian-based system, this is done by "apt-get update" annd "apt-get install". To run those command, you need to use keyword "RUN". "RUN" command basically execute your command that follows it. 
Let's give the Ubuntu system an update and install wget in it. Your Dockerfile should now look like:
```
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install wget
```
You could use && to run 2 commands together, for example
```
RUN apt-get update && apt-get install wget
```
is equivalent to the previous one. RUN could be used in other commands such as "wget", "tar", or "make". Note that the only thing kept in the final image due to RUN command is changes to the filesystem. Other changes, such as current directory, will not be changed. For example, suppose you have a ./make file in a directory /program. Doing
```
RUN tar xf program.tar
RUN cd program
RUN ./make
```
will give you "file not found", as RUN does not change your current working directory. To do this, you need the key work "WORKDIR". So instead you should do 
```
RUN tar xf program.tar
WORKDIR program
RUN ./make
```
This will set your working directory to /program. Another important key word is ENV. For example, if you want to add /program/bin to the search path, normally you would do
```sh
export PATH=/program/bin:$PATH
```
however, if you do 
```
RUN export PATH=/program/bin:$PATH
```
Nothing will get changed! Instead, to set the environment variable $PATH, you should do 
```
ENV PATH="${PATH}:/program/bin"
```
This will set the environment properly. 
Another useful keywork is COPY. Suppose you have a file called "local-file.tar.gz" that you need in setting up the image, and you save it in the save directory as the Dockerfile, doing
```
COPY local-file.tar.gz /file-needed
```
will copy the "local-file.tar.gz" into a directory /file-needed in your image.
Those are the keywords that you will most likely use when building your docker image. There are many other keywords that might come to use. You could find then in the [Dockerfile reference] page.
Let's show a complete example of building a Python environment and install 2 python packages. This example is from the [CHTC] tutorial:
```
# Build the image based on the official Python version 3.8 image
FROM python:3.8
# Our base image happens to be Debian-based, so it uses apt-get as its system package manager
RUN apt-get update && apt-get install -y wget
# Use RUN to install Python packages (numpy and scipy) via pip, Python's package manager
RUN pip3 install numpy scipy
```
Here "-y" give "yes" to all prompt during the installation. Some system have those user-typed prompt, and dockerfile would terminate if not answered automatically. 
Now you would have a rough sense how to write a Dockerfile. It's time to turn your Dockerfile into an actual image.

## Build the image from Dockerfile
### Build
First, you need to open up a terminal and navigate to the directory where your Dockerfile locates using "cd". Then type the following into the terminal:
```sh
docker build -t your-username/your-image-name:tag .
```
replace your-username with your dockerHub username, replace your-image-name with an image name of your choice, and replace tag with a tag of your choice. 
### Test
First, you want to create a working directory. The directory should contain all the files needed for your job. Those include the dataset(.csv files) and your code(.py or .java, for example). If you have a shell script(.sh), you should also put it here. You could also run the command directly in the container, however. This is purely your choice.
Now, use "cd" command to naviagate to your working directory. We will use "docker run" command as we did with the simple Ubuntu image, but with more flags:
```sh
docker run --user $(id -u):$(id -g) --rm=true -it -v $(pwd):/scratch -w /scratch username/imagename:tag /bin/bash
```
The meaning of the flags are as follows:
- --user $(id -u):$(id -g): runs the container with more restrictive permissions
- --rm=true : clean up the runnining container after exit
- -it : interactive
- -v $(pwd):/scratch : put the current working directory (the one with data and code) into the container, name the directory /scratch
- -w /scratch: when the container starts, make /scratch the working directory
- /bin/bash : start a bash(command prompt) when running the container

You should see a new prompt like this:
```
I have no name!@2fe289f8b55d:/scratch$
```
Now you are running in the container! Do what you normally do to run your job. Either via command such as
```sh
python3 your_code.py
```
or by a shell script that contains all the command:
```sh
./sript_that_runs_everything
```
Check if the process and the output is exactly the same as what you get if you run everything locally. If so, then you pass the test! If something's wrong, there might be many possibilities. I would suggest to look into your Dockerfile, this is where most error occurs, ie, you did not set up the environment properly.
### Push
After you tested everything and you believe your image is ready to go, you should push it to Dockerhub so that when you run on servers, the server could pull your image from your DockerHub repository. If it's your first time pushing from this device, do the following first:
```
docker login
```
There would be a prompt that ask for your DockerHub username and password. After that, do 
```
docker push your-username/your-image-name:tag
```
the "your-username/your-image-name:tag" should be the same as the one you specified when doing "docker build". Now your image should be on DockerHub! You could check it in [DockerHub website] under the repository tab.

## Use your container on Servers?
Well, this is a tricky one. Depend on the server you are using, it could either be very easy or very hard, maybe even impossible if your server does not support container. Fortunately, [CHTC], the one server that we all love, supports container and is extremely easy to have your jobs running in a container. All you have to do is to change your universe from "vanilla" to "docker" and specify which image the CHTC server should pull, and you could keep the rest unchanged!
```
universe = docker
docker_image = your-username/your-image-name:tag
```


[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)
   [CHTC]: <https://chtc.cs.wisc.edu/>
   [DockerHub website]: <https://hub.docker.com/>
   [Windows link]: <https://docs.docker.com/desktop/install/windows-install/>
   [Mac link]: <https://docs.docker.com/desktop/install/mac-install/>
   [Linux link]: <https://docs.docker.com/desktop/install/linux-install/>
   [Engine link]: <https://docs.docker.com/engine/install/centos/>
   [Dockerfile reference]: <https://docs.docker.com/engine/reference/builder/>
