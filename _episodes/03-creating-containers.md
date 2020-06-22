---
title: "Creating containers"
teaching: 35
exercises: 15
questions:
- "How do I get Docker to perform computation?"
objectives:
- "Demonstrate how to create an instance of a container from an image."
- "Explain how to list (container) images on your laptop."
- "Explain how to list running and completed containers."
- "Explain how to create a docker container and get interactive access."
keypoints:
- "Containers are usually created using command line invocations."
- "The `docker run` command creates containers from images."
- "The `docker image` command can list and manage images that are (now) on your computer."
- "The `docker container` command can list and manage containers that have been created."
- "The `-d -t` options to `docker run` allow you to create a container running in the background."
- "The `docker exec` command allows you to execute commands in a running container, including getting interactive access."
---

## Creating a "hello world" container

> ## Reminder of terminology: images and containers
> - Recall that a container *image* is the template from which particular instances of *containers* will be created.
{: .callout}

One of the simplest Docker container images just allows you to create containers that print a welcome message.

To create and run containers from named Docker images you use the `docker run` command. Open a shell window if you do not already have one open and try the following `docker run` invocation. Note that it does not matter what your current working directory is.
~~~
$ docker run hello-world
~~~
{: .language-bash}
~~~
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

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
~~~
{: .output}

Now try to run the above command again, and observe the difference in the output.

The message from "Hello from Docker!" is what's produced by instances of the hello-world container. It should be identical every time you make an instance of the hello-world container and run it.

 The first time you saw some initial output from the Docker command itself:
~~~
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest
~~~
{: .output}

What this says is that your host (laptop) didn't have its own local copy of the hello-world container's image.

Docker then connected to the Docker Hub to find and download the (container) image with that name.

The message notes information about the Docker Hub website, but we'll return to that in the next episode.

The "digest" is a secure fingerprint (a "hash") of the particular version of the container image that you now have... (Although of course the usefulness of that hash is minimal if you don't know what to compare it to!)

## Where did that "hello-world" container image get stored?

The `docker image` command is used to list and modify Docker images.
You can find out what container images you have local copies of using the following command ("ls" is short for "list"):
~~~
$ docker image ls
~~~
{: .language-bash}
~~~
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
hello-world            latest              fce289e99eb9        4 weeks ago         1.84kB
~~~
{: .output}

## Removing images

If you need to reclaim disk space, you can remove image files. The images and their corresponding containers can start to take up a lot of disk space if you don't clean them up occasionally. On macOS and Windows, when you uninstall the overall Docker software, it should have the effect of removing all of your image files (although I have not explicitly tested this!).

If you want to remove an image, you will need to find out details such as the image ID or name of the repository. For example, say my laptop contained the following image.
~~~
$ docker image ls
~~~
{: .language-bash}
~~~
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
ubuntu                 trusty              dc4491992653        12 months ago       222MB
hello-world            latest              fce289e99eb9        4 months ago        1.84kB
~~~
{: .output}

We can try to remove remove the `hello-world` image with a `docker image rm` command that includes the repository name, like so but...:

~~~
$ docker image rm hello-world
~~~
{: .language-bash}
~~~
Error response from daemon: conflict: unable to remove repository reference "hello-world" (must force) - container e7d3b76b00f4 is using its referenced image fce289e99eb9
~~~
{: .output}

We get this error because there are containers created that depend on this image. (Remember that a *container* is an instance of a particular *image*).)

In order to remove an image from our local image store, we first need to stop and then remove any containers that we’ve run that are based on this image.

## What containers are running?

As indicated by the error above there is still an existing container from this image.
We can list the running containers with the `docker container ls` command:

~~~
$ docker container ls
~~~
{: .language-bash}
~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
~~~
{: .output}

This command did not return any containers because our containers all exited and thus stopped running after they completed their work.

## What containers exist but are not running?

There is also a way to list all known containers, including those that are not currently running, which is to add the `--all`/`-a` flag to the `docker container ls` command as shown below.

~~~
$ docker container ls --all
~~~
{: .language-bash}
~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
9c698655416a        hello-world         "/hello"            2 minutes ago       Exited (0) 2 minutes ago                       zen_dubinsky
~~~
{: .output}

We will talk more about how you might use these containers that have exited and how to restart a container later in this lesson.

## How do I remove an container that has exited?

To delete an exited container you can run the following command, inserting the `NAME` for the container you wish to remove. It will repeat the `NAME` back to you, if successful.
~~~
$ docker container rm zen_dubinsky
~~~
{: .language-bash}
~~~
zen_dubinsky
~~~
{: .output}

> ## Names and IDs
> You can use either the `NAME` or `CONTAINER ID` with the `docker container rm` command, whichever you find easier. The names and IDs are randomly generated by Docker to be unique among your list of exisiting containers.
> 
> You’ve probably noticed that Docker generates some interesting default container names! Interested in where these come from? Take a look at the name generator code in the Docker GitHub repository. And, as highlighted in this short blog post, take a look at the code in the GetRandomName function!
{: .callout}

If you want to remove all exited containers at once you can use the `docker containers prune` command.

**Be careful** with this command. If you have containers you may want to reconnect to, you should not use this command.
It will ask you if to confirm you want to remove ALL stopped (exited) containers, see output below. If successful it will print the full `CONTAINER ID`s back to you.

~~~
$ docker container prune
~~~
{: .language-bash}
~~~
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
9c698655416a848278d16bb1352b97e72b7ea85884bff8f106877afe0210acfc
6dd822cf6ca92f3040eaecbd26ad2af63595f30bb7e7a20eacf4554f6ccc9b2b
~~~
{: .output}

## Removing images, for real this time

Now that we’ve removed the stopped container that was dependent on the hello-world image, we should be able to remove the image successfully:

~~~
$ docker image rm hello-world
~~~
{: .language-bash}
~~~
Untagged: hello-world:latest
Untagged: hello-world@sha256:5f179596a7335398b805f036f7e8561b6f0e32cd30a32f5e19d17a3cda6cc33d
Deleted: sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
Deleted: sha256:af0b15c8625bb1938f1d7b17081031f649fd14e6b233688eea3c5483994a66a3
~~~
{: .output}

The reason that there are a few lines of output, is that a given image may have been formed by merging multiple underlying layers. Any layers that are used by multiple Docker images will only be stored once. Now the result of `docker image ls` should no longer include the `hello-world` image.

> ## Removing all images
> Removing all images is a little more tricky than removing all containers, you need to combine two commands. The `docker image ls -q` command returns a list of all image IDs. You can use the result of this command combined with `docker image rm` to delete all images in one command:
> ~~~
> docker image rm $(docker image ls -q)
> ~~~
> {: .language-bash}
>
> This command assumes you are using a `bash` (or compatible) shell. If you happen to be using another shell such as `csh` or `tcsh`, this command will fail and you should instead wrap the `docker image ls -q` part of the command in “backticks”, e.g. docker image rm `docker image ls -q`.
{: .callout}

## Running containers in the background and interactive access

Often, you will want to use containers to run specific commands, analyses or tasks (as we have seen with the "hello-world" example above). Sometimes, however, you would prefer to have a container running and to get interactive access so you can run commands directly, within the container yourself. You could, for example, be undertaking some data analysis using tools in the container but do not yet know the exact steps you want the analysis to take - you are exploring the data.

For this to work, the container image has to have the software installed to support interactive access (e.g. a shell or terminal). Almost all of the images that are based on Linux distributions will come with this software installed so if you pick, for example, an Ubuntu image then this will work.

First, we need to run the container as we did for `hello-world` with a couple of additional flags to do two things:

   - `-t` flag: Allocate a psuedo-tty to allow us to access the container interactively
   - `-d` flag: Detach the container and run in the background

~~~
$ docker run -t -d ubuntu
~~~
{: .language-bash}
~~~
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
5c939e3a4d10: Pull complete 
c63719cdbe7a: Pull complete 
19a861ea6baf: Pull complete 
651c9d2d6c4f: Pull complete 
Digest: sha256:8d31dad0c58f552e890d68bbfb735588b6b820a46e459672d96e585871acc110
Status: Downloaded newer image for ubuntu:latest
dc5fae49118e449662ebdf6ab7790c98b828d2b8def3fca09491bc48c9063580
~~~
{: .output}

> ## Some images may also need you to define an entry point or override an existing entry point
> For the above approach to work the image has to run a shell by default (most Linux OS images do this). Some minimal OS images may also need you to add the `--entrypoint "/bin/sh"` option to `docker run`, or to place “`/bin/sh`” at the end of the `docker run` command.
> 
> *What’s the difference between the --entrypoint option and placing /bin/sh at the end of the command?* Some images are pre-configured with an entrypoint - the command to run by default within a container started from the image. If the default command that runs when you start the container is not the command you want to run, you can override it using the `--entrypoint` option. If an image does not have an entrypoint pre-configured, you tell Docker what command to run when starting a container from the image at the end of the `docker run` command.
{: .callout}

We can get the image ID and see the image is running in the background with `docker container ls`:

~~~
$ docker container ls
~~~
{: .language-bash}
~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
dc5fae49118e        ubuntu              "/bin/bash"         33 seconds ago      Up 31 seconds                           exciting_keller
~~~
{: .output}

What we have now is a running container with an open shell sitting waiting for input. The open shell and the fact that we assigned a pseudo-tty to connect to the shell’s input channel prevent the container from exiting straight away as it would have if we ran a command that exited immediately (for example `echo Hello World!`). We could now connect to this shell using the `docker attach` command. However, we will use a different approach, starting a new interactive shell that we can work in. The reasoning for this will be explained shortly. Let’s start an interactive shell directly in the container image using the `docker exec` command with the container image ID:

~~~
$ docker exec -i -t exciting_keller bash
~~~
{: .language-bash}
~~~
root@dc5fae49118e:/# 
~~~
{: .output}

Note that the prompt changed to show we are now in the container image. You can now run commands interactively in the container image.

To get out of the container image, we use the `exit` command:

~~~
root@dc5fae49118e:/# exit
~~~
{: .language-bash}
~~~
exit
$
~~~
{: .output}

Which takes us back to our original command prompt.

Finally, we need to stop the container (it is still running in the background). We use the `docker stop` command with the container image ID to do this:

~~~
$ docker container ls
$ docker stop exciting_keller
$ docker container ls
~~~
{: .language-bash}
~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
dc5fae49118e        ubuntu              "/bin/bash"         8 minutes ago       Up 8 minutes                            exciting_keller

exciting_keller

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
~~~
{: .output}

> ## Mounting local directories in the container
> It is likely that you will usually want to mount a local directory from the host system in the container that you start in order to make files from the host system available within the running container. We will discuss making local directories available in containers in more detail later in the course. For now, as a quick example, modify the `docker run` command above so that the container will have the current directory that you are in on the host machine mounted at `/data` in the container:
> ~~~
> docker run -t -d -v ${PWD}:/data ubuntu
> ~~~
> {: .language-bash}
{: .callout}

## Why did we start a new shell using `docker exec`?

It was highlighted above that we started a new shell in our running container using `docker exec` rather than connecting to the shell created when starting the container using `docker run`. Why did we do this? You may have noticed that when you run a container, as soon as the process that the container was started with finishes, the container exits. So, if you start your container and it is set to run `echo Hello World!`, as soon as the `echo` command exits (once it’s printed “Hello World!” to the screen), the container exits. Because we started our container running a shell with a connected pseudo-tty, it is waiting for shell input and doesn’t exit. If we connect to this shell using docker attach, we can interact with it in the same way as we did with the shell we opened above, BUT, when we type `exit` to leave the shell, the container will terminate because the shell process exits. In order to exit from our container without terminating the shell process when we’ve attached using `docker attach`, we use a special key combination `CTRL-p CTRL-q` to detach our terminal from the terminal in the running docker container. To avoid this, we simply started a second shell which we could exit as normal by typing `exit` without affecting the top-level shell started at container startup and, thus, leaving the container running.

> ## The Docker official documentation is helpful!
> There is lots of great documentation at <https://docs.docker.com/>, for example, detailed reference material and tutorials covering the use of the commands mentioned above.
{: .callout}

### Conclusion

You have now successfully downloaded a Docker image file to your computer, and have created a Docker container from it.
While this already represents a reproducible computational environment, the image contents are not under your control, so we look at this topic after a quick detour to the Docker Hub in the next episode.

{% include links.md %}

{% comment %}
<!--  LocalWords:  keypoints amd64 fce289e99eb9 zen_dubinsky links.md
 -->
<!--  LocalWords:  eager_engelbart endcomment
 -->
{% endcomment %}
