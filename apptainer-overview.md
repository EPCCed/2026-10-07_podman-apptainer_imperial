---
title: Overview of Apprainer
teaching: 15
exercises: 10
---

::::::::::::::::::::::::::::::::::::::: objectives

- Understand what Apptainer is and when you might want to use it.
- Run a simple Apptainer container.
- Learn how to access local files inside an Apptainer container.
- Learn how to set the environment inside an Apptainer container.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- What is Apptainer?
- How do I create Apptainer image files and run Apptainer containers?
- How can I make data on the host available in Apptainer containers?

::::::::::::::::::::::::::::::::::::::::::::::::::

This episode provides a quick introduction to the Apptainer container platform building on the material we have already covers with Podman. It is not an exhaustive tour of Apptainer features - you can check out the [Apptainer User Guide](https://apptainer.org/docs/user/main/index.html) for more information.

## What is Apptainer?

[Apptainer](https://apptainer.org/) (or [Singularity](https://sylabs.io/singularity/), we'll get to this in a minute...) is another container platform.

in some ways, Apptainer is similar to Podman. However, in other ways, particularly in terms of the system's architecture, it is fundamentally different. These differences mean that Apptainer is particularly well-suited to running on shared platforms such as distributed, High Performance Computing (HPC) platforms.

Apptainer runs containers from container images which, as we discussed, are essentially a virtual computer disk that contains all of the necessary software, libraries and configuration to run one or more applications or undertake a particular task, e.g. to support a specific research project. This saves you the time and effort of installing and configuring software on your own system or setting up a new computer from scratch, as you can simply run a Apptainer container from an image and have a virtual environment that is equivalent to the one used by the person who created the image. Apptainer/Apptainer is increasingly widely used in the research community for supporting research projects due to its support for shared computing platforms.

System administrators will not, generally, install Docker on shared computing platforms such as lab desktops, research clusters or HPC platforms because the design of Docker presents potential security issues for shared platforms with multiple users. Apptainer, on the other hand, can be run by end-users entirely within "user space", that is, no special administrative privileges need to be assigned to a user in order for them to run and interact with containers on a platform where Apptainer has been installed.


:::::::::::::::::::::::::::::::::::::::::  callout

## A little history...

Singularity is open source software and was initially developed within the research
community. A couple of years ago, the project was "forked" something that is
not uncommon within the open source software community, with the software
effectively splitting into two projects going in different directions. The Singularity fork
is being developed by a commercial entity, [Sylabs.io](https://sylabs.io/) who
provide both the free, open source [SingularityCE (Community
Edition)](https://sylabs.io/singularity) and Pro/Enterprise editions of the
software. The original open source Singularity project has recently been
[renamed to
Apptainer](https://apptainer.org/news/community-announcement-20211130/) and has
moved into the Linux Foundation. While earlier versions of this course
focused on versions of Singularity released before the project fork, we now
base the course material on recent Apptainer releases. Despite this, the basic
features of Apptainer/Singularity remain the same and so this material is
equally applicable whether you're working with a recent Apptainer release or a
slightly older Singularity version. Nonetheless, it is useful to be aware of
this history and that you may see both Apptainer and Singularity being used
within the research community over the coming months and years.

Another point to note is that some systems that have a recent Singularity release
installed may also provide a `apptainer` command that is simply a link to the
`singularity` executable on the system. This helps to ensure workflows
that use the original `singularity` command can continue to work as intended.

For now, the remainder of this material refers to Apptainer but where you
have a release of Singularity installed on your local system, you can simply
replace references to `apptainer` with `singularity`, if you wish.

::::::::::::::::::::::::::::::::::::::::::::::::::

## Checking Apptainer works

[Login to Cirrus](https://docs.cirrus.ac.uk/user-guide/connecting/) using the
login address `login.cirrus.ac.uk`:

```bash
ssh -i /path/to/ssh-key user@login.cirrus.ac.uk
```

Now check that the `apptainer` command is available in your terminal:

```bash
remote$ apptainer --version
```

```output
apptainer version 1.4.2-1.el9
```

:::::::::::::::::::::::::::::::::::::::::  callout

## Loading a module

HPC systems often use *modules* to provide access to software on the system so you may need to use the command:

```bash
remote$ module load apptainer
```

before you can use the `apptainer` command on remote systems. However, this depends on how the system is configured.
You do not need to load a module on Cirrus. If in doubt, consult the documentation for the system you are using
or contact the support team.

::::::::::::::::::::::::::::::::::::::::::::::::::

## Images and containers: reminder

A quick reminder on terminology: we refer to both *container images* and *containers*. What is the difference between these two terms? 

*Container images* (sometimes just *images*) are bundles of files including an operating system, software and potentially data and other application-related files. They may sometimes be referred to as a *disk image* or *image* and they may be stored in different ways, perhaps as a single file, or as a group of files. Either way, we refer to this file, or collection of files, as an image.

A *container* is a virtual environment that is based on a container image. That is, the files, applications, tools, etc that are available within a running container are determined by the image that the container is started from. It may be possible to start multiple container instances from an image. You could, perhaps, consider an image to be a form of template from which running container instances can be started.

## Getting a container image and running a Apptainer container

Apptainer uses the [Singularity Image Format (SIF)](https://apptainer.org/docs/user/main/cli/apptainer_sif.html) and container images are provided as single `SIF` files (usually with a `.sif` or `.img` filename extension). Apptainer container images can be obtained from standard container repositories liek the ones we saw earlier in the course. Apptainer will convert them to SIF container image files automatically.


### Pulling a container image

Let's begin by creating a `test` directory, changing into it and _pulling_ the same simple container image we did for Podman at the start of this workshop:

```bash
remote$ mkdir test
remote$ cd test
remote$ apptainer pull hello.sif docker://quay.io/podman/hello
```

```output
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
INFO:    Fetching OCI image...
332.7KiB / 332.7KiB [==================================================] 100 %0s
INFO:    Extracting OCI image...
INFO:    Inserting Apptainer configuration...
INFO:    Creating SIF file...
[=====================================================================] 100 % 0s
```

What just happened? We pulled a container image from a remote repository using the `apptainer pull` command and directed it to store the container image in a file using the name `hello.sif` in the current directory. The `docker://quay.io` part of the command tells Apptainer that the container image is in Docker/OCI format (the standard container image format) and Apptainer converted the OCI image to a SIF file. If you run the `ls` command, you should see that the `hello.sif` file is now present in the current directory.

```bash
remote$ ls -lh
```

```output
total 60M
-rwxr-xr-x. 1 auser group 360K Sep 17 08:43 hello.sif
```

### Running a Apptainer container

We can now run a container based on the `hello.sif` container image:

```bash
remote$ apptainer run hello.sif
```

```output
!... Hello Podman World ...!

         .--"--.           
       / -     - \         
      / (O)   (O) \        
   ~~~| -=(,Y,)=- |         
    .---. /`  \   |~~      
 ~/  o  o \~~~~.----. ~~   
  | =(X)= |~  / (O (O) \   
   ~~~~~~~  ~| =(Y_)=-  |   
  ~~~~    ~~~|   U      |~~ 

Project:   https://github.com/containers/podman
Website:   https://podman.io
Desktop:   https://podman-desktop.io
Documents: https://docs.podman.io
YouTube:   https://youtube.com/@Podman
X/Twitter: @Podman_io
Mastodon:  @Podman_io@fosstodon.org
```

The above command ran a *hello* container based on the container image we downloaded from the online repository and the resulting output was shown. 

What just happened? When we use the `apptainer run` command, Apptainer does three things:

| 1. Starts a Running Container | 2. Performs Default Action | 3. Shuts Down the Container
| --------------------|-----------------|----------------|
| Starts a running container, based on the container image. Think of this as the "alive" or "inflated" version of the container -- it's actually doing something. | If the container has a default action set, it will perform that default action. This could be as simple as printing a message (as above) or running a whole analysis pipeline! | Once the default action is complete, the container stops running (or exits). |

### Default action

How did the container determine what to do when we ran it? What did running the container actually do to result in the displayed output?

When you run a container from a Apptainer container image using the `apptainer run` command, the container runs the default run script that is embedded within the container image. This is a shell script that can be used to run commands, tools or applications stored within the container image on container startup. We can inspect the container image's run script using the `apptainer inspect` command:

```bash
remote$ apptainer inspect -r hello.sif
```


This shows us the script within the `hello.sif` container image configured to run by default when we use the `apptainer run` command.

## Accessing local files in Apptainer containers

The key concept to remember when running a Singularity container, you only have the same permissions to access files as the user on the host system that you start the container as. 

This is different from Podman if we use it on our local system (as many people did in the early part of this course) where you generally have access to any files you wish with administrator rights. (Podman can be made available on shared systems where access to data is restricted as it is for Apptainer - but using a different mechanism.)

## Users within a Apptainer container

The first thing to note is that if you run `whoami` within a container shell you should have seen the same username that you have on the host system when you ran the container. 


For example, if we download an official Python image, open a terminal inside a running container and check our username, we should see it is the same as on the HPC system itself

```
remote$ apptainer pull python-slim.sif docker://docker.io/python:slim
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
INFO:    Fetching OCI image...
28.4MiB / 28.4MiB [===========================================] 100 % 0.0 b/s 0s
11.8MiB / 11.8MiB [===========================================] 100 % 0.0 b/s 0s
4.1MiB / 4.1MiB [=============================================] 100 % 0.0 b/s 0s
INFO:    Extracting OCI image...
INFO:    Inserting Apptainer configuration...
INFO:    Creating SIF file...
[=====================================================================] 100 % 0s
remote$ whoami
auser
remote$ apptainer shell python-slim.sif 
Apptainer> whoami 
auser
Apptainer> exit
exit
remote$ 
```

:::::::::::::::::::::::::::::::::::::::::  callout

## How does the container know my username?

But wait! I downloaded the standard, public version of the Python container image from Docker Hub. I have not customised
it in any way. How is it configured with my own user details?!

If you have any familiarity with Linux system administration, you may be aware that in Linux, users and their Unix groups are configured in the `/etc/passwd` and `/etc/group` files respectively. In order for the running container to know of my
user, the relevant user information needs to be available within these files within the container.

Assuming this feature is enabled within the installation of Apptainer on your system, when the container is started, Apptainer appends the relevant user and group lines from the host system to the `/etc/passwd` and `/etc/group` files within the container.

This means that the host system can effectively ensure that you cannot access/modify/delete any data you should not be able to on the host system from within the container and you cannot run anything that you would not have permission to run on the host system since you are restricted to the same user permissions within the container as you are on the host system.

::::::::::::::::::::::::::::::::::::::::::::::::::

## Files and directories within an Apptainer container

Apptainer also *binds* some *directories* from the host system where you are running the `apptainer` command into the container that you are starting. As we saw for Podman, this bind process is not copying files into the running container, it is making an existing directory on the host system visible and accessible within the container environment.

Your *home directory* will almost always be available by default in running containers. You may also find that the directory from which you issued the `apptainer` command (the *current working directory*) is also bound.

:::::::::::::::::::::::::::::::::::::::::  callout

## Defaults are a local config choice

There is a default configuration of which files and directories are bound into the container but ultimate control of how things are set up on the system where you are running Apptainer is determined by the system administrator. As a result, this section provides an overview but you may find that things are a little different on the system that you're running on.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::  challenge

## Files in Singularity containers

**Q1:** What do you notice about the ownership of files in a container started from the `lolcow.sif` image? (e.g. take a look at the ownership of files in the root directory (`/`) and your home directory (`~/`)).

**Exercise 1:** In this container, try creating a file in the root directory `/` (e.g. using `touch /myfile.dat`). What do you notice? Try removing the `/singularity` file. What happens in these two cases?

**Exercise 2:** In your home directory within the container shell, try and create a simple text file (e.g. `echo "Some text" > ~/test-file.txt`). Is it possible to do this? If so, why? If not, why not?! If you can successfully create a file, what happens to it when you exit the shell and the container shuts down?

:::::::::::::::  solution

## Answers

**A1:** Use the `ls -l /` command to see a detailed file listing including file ownership and permission details. You should see that most of the files in the `/` directory are owned by `root`, as you would probably expect on any Linux system. If you look at the files in your home directory, they should be owned by you.

**A Ex1:** We've already seen from the previous answer that the files in `/` are owned by `root` so we would nott expect to be able to create files there if we're not the root user. However, if you tried to remove `/singularity` you would have seen an error similar to the following: `cannot remove '/singularity': Read-only file system`. This tells us something else about the filesystem. It's not just that we do not have permission to delete the file, the filesystem itself is read-only so even the `root` user would not be able to edit/delete this file. We will look at this in more detail shortly.

**A Ex2:** Within your home directory, you _should_ be able to successfully create a file. Since you're seeing your home directory on the host system which has been bound into the container, when you exit and the container shuts down, the file that you created within the container should still be present when you look at your home directory on the host system.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Binding additional host system directories to the container

You will sometimes need to bind additional host system directories into a container you are using over and above those bound by default. For example:

- There may be a shared dataset in a location that you need access to in the container
- You may require executables and software libraries from the host system in the container

The `-B` option to the `apptainer` command is used to specify additional binds. For example, to bind the `/opt/cray` directory (where the HPE Cray programming environment is stored) into a container you could use:

```bash
remote$ apptainer shell -B /opt/cray python-slim.sif
Apptainer> ls -la /opt/cray
```

Note that, by default, a bind is mounted at the same path in the container as on the host system. You can also specify where a host directory is mounted in the container by separating the host path from the container path by a colon (`:`) in the option:

```bash
remote$ apptainer shell -B /opt/cray:/cpe python-slim.sif
Apptainer> ls -la /cpe
```

You can specify multiple binds to `-B` by separating them by commas (`,`).

Another option is to specify the paths you want to bind in the `APPTAINER_BIND` environment variable. This can be more convenient when you have a lot of paths you want to bind into the running container For example, to bind the locations that contain both the HPE Cray programming environment and the CSE centrally installed software into a running container, we would use:

```bash
remote$ export APPTAINER_BIND="/opt/cray,/epccfs/y07/shared/cirrus-ex"
remote$ apptainer shell python-slim.sif
Apptainer> ls -la /epccfs/y07/shared/cirrus-ex
```

Finally, you will remember from discussing building container images using Podman that you can also copy data into a container image at build time if there is some static data required in the image. 

:::::::::::::::::::::::::::::::::::::::: keypoints

- Apptainer is another container platform and it is often used in cluster/HPC/research environments.
- Apptainer has a security model suited to shared access systems - users can only access data they already have access to.
- Apptainer has its own container image format (SIF).
- The `apptainer` command can be used to pull images from online repositories and convert them into SIF files automatically.
- You can use the `-B` option to make host files available inside a running container.

::::::::::::::::::::::::::::::::::::::::::::::::::