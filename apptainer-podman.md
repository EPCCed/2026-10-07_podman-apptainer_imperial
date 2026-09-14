---
title: Using Docker Images with Apptainer
teaching: 5
exercises: 10
---

::::::::::::::::::::::::::::::::::::::: objectives

- Learn how to run Apptainer containers based on Podman/Docker images.
- Learn how to build Apptainer containers.

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do I use Podman/Docker images with Apptainer?
- How do I build container images to use with Apptainer?

::::::::::::::::::::::::::::::::::::::::::::::::



## Using Podman/Docker images with Apptainer

Apptainer can also start containers directly from Docker container images, opening up access to a huge number of existing container images available on [Docker Hub](https://hub.docker.com/) and other registries.

While Apptainer doesn't actually run a container using the Docker container image (it first converts it to a format suitable for use by Apptainer), the approach used provides a seamless experience for the end user. When you direct Apptainer to run a container based on a Docker container image, Apptainer pulls the slices or _layers_ that make up the Docker container image and converts them into a single-file Apptainer SIF container image.

For example, moving on from the simple _Hello World_ examples that we've looked at so far, let's pull one of the [official Docker Python container images](https://hub.docker.com/_/python). We'll use the image with the tag `3.9.6-slim-buster` which has Python 3.9.6 installed on Debian's [Buster](https://www.debian.org/releases/buster/) (v10) Linux distribution:

```bash
remote$ apptainer pull python-3.13.2.sif docker://python:3.13.2-slim-bookworm
```

```output
INFO:    Converting OCI blobs to SIF format
WARNING: 'nodev' mount option set on /tmp, it could be a source of failure during build process
INFO:    Starting build...
Getting image source signatures
Copying blob c29f5b76f736 done  
Copying blob 91af2433aaf0 done  
Copying blob 0d263a67e0db done  
Copying blob 34abb9d38fa3 done  
Copying config efb22c4342 done  
Writing manifest to image destination
Storing signatures
2025/02/09 09:15:56  info unpack layer: sha256:c29f5b76f736a8b555fd191c48d6581bb918bcd605a7cbcc76205dd6acff3260
2025/02/09 09:15:56  warn xattr{etc/gshadow} ignoring ENOTSUP on setxattr "user.rootlesscontainers"
2025/02/09 09:15:56  warn xattr{/tmp/build-temp-141758395/rootfs/etc/gshadow} destination filesystem does not support xattrs, further warnings will be suppressed
2025/02/09 09:15:56  info unpack layer: sha256:91af2433aaf067a250607e067549a710adc587e7f5cf57c4ee096224cc5834dc
2025/02/09 09:15:56  warn xattr{var/cache/apt/archives/partial} ignoring ENOTSUP on setxattr "user.rootlesscontainers"
2025/02/09 09:15:56  warn xattr{/tmp/build-temp-141758395/rootfs/var/cache/apt/archives/partial} destination filesystem does not support xattrs, further warnings will be suppressed
2025/02/09 09:15:57  info unpack layer: sha256:0d263a67e0dbb841a730efc09f84672f73b02427a19a57d9ef380b66eaaf0aec
2025/02/09 09:15:57  warn xattr{var/log/apt/term.log} ignoring ENOTSUP on setxattr "user.rootlesscontainers"
2025/02/09 09:15:57  warn xattr{/tmp/build-temp-141758395/rootfs/var/log/apt/term.log} destination filesystem does not support xattrs, further warnings will be suppressed
2025/02/09 09:15:57  info unpack layer: sha256:34abb9d38fa3fcf8526765f370a3f2c6341456eca879f3af027c42062521e5bc
INFO:    Creating SIF file...
```

Note how we see Apptainer saying that it's "_Converting OCI blobs to SIF format_". We then see the layers of the Docker container image being downloaded and unpacked and written into a single SIF file. Once the process is complete, we should see the python-3.9.6.sif container image file in the current directory.

We can now run a container from this container image as we would with any other Apptainer container image.

:::::::::::::::::::::::::::::::::::::::  challenge

## Running a Python container based on the image that we just pulled from Docker Hub

Try running a Python container. What happens? 

Try running some simple Python statements...

:::::::::::::::  solution

## Solution

```bash
remote$ apptainer run python-3.13.2.sif
```

This should put you straight into a Python interactive shell within the running container:

```output
Python 3.13.2 (main, Feb  6 2025, 22:37:50) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

Now try running some simple Python statements:

```python
>>> import math
>>> math.pi
3.141592653589793
>>> 
```

::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

In addition to running a container and having it run the default run script, you could also start a container running a shell in case you want to undertake any configuration prior to running Python. This is covered in the following exercise:

:::::::::::::::::::::::::::::::::::::::  challenge

## Open a shell within a Python container

Try to run a shell within a apptainer container based on the Python container image. That is, run a container that opens a shell rather than the default Python interactive console as we saw above.
See if you can find more than one way to achieve this.

Within the shell, try starting the Python interactive console and running some Python commands.

:::::::::::::::  solution

## Solution

Recall from the earlier material that we can use the `apptainer shell` command to open a shell within a container. To open a regular shell within a container based on the `python-3.13.2.sif` container image, we can therefore run:

```bash
remote$ apptainer shell python-3.13.2.sif
```

```output
Apptainer> echo $SHELL
/bin/bash
Apptainer> cat /etc/issue
Debian GNU/Linux 10 \n \l

Apptainer> python
Python 3.13.2 (main, Feb  6 2025, 22:37:50) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print('Hello World!')
Hello World!
>>> exit()

Apptainer> exit
$ 
```

It is also possible to use the `apptainer exec` command to run an executable within a container. We could, therefore, use the `exec` command to run `/bin/bash`:

```bash
remote$ apptainer exec python-3.13.2.sif /bin/bash
```

```output
Apptainer> echo $SHELL
/bin/bash
```

You can run the Python console from your container shell simply by running the `python` command.

::::::::::::::::::::::::

## How do I build Apptainer image files?

TODO : Use Podman/Docker

## Importing container images from archive files

TODO: add details on importing archive files

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::: keypoints

- Apptainer can start a container from a Podman/Docker image which can be pulled directly from a container registry.
- Even if you are running containers using Apptainer, you should build container images using Podman/Docker.

::::::::::::::::::::::::::::::::::::::::::::::::

## References

\[1\] [Gregory M. Kurzer, Containers for Science, Reproducibility and Mobility: Apptainer P2. Intel HPC Developer Conference, 2017](https://www.youtube.com/watch?v=DA87Ba2dpNM)
