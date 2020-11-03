# Building Container Images with Advanced Dockerfile Instructions

OpenShift provides S2I builder images that contain operating system libraries, language runtimes, and frameworks and which use source code as input to create application container images. This guide shows how to use the RHEL Universal Base Image (UBI) with parent images to create child application images.

## Red Hat Universal Base Image

The Red Hat Universal Base Image aims to be a high-quality, flexible base container image for building containerized applications.

* It is based on 3 Red Hat RHEL-based sub-images, common language runtimes, and a set of yum repositories and channels that include RPM packages.
  - ubi, ubi-minimal, ubi-init
  - java, php, python 2/3, ruby, nodejs (UBI 8 contains perl, dotnet)
* The UBI updates when RHEL versions update
* Red Hat promises to maintain images for as long as they maintain RHEL
  

![RHEL Release and Support Timeline](/Building_With_Dockerfiles/images/RHEL-version-horizon.png)

### Reasons to Use UBI in your Applications

* Small (90-200 MB) and performant
* Certified by Red Hat's Security Team
* Compatible with broad ecosystem of RHEL without buyer lock-in
  * Resulting applications can be deployed on any container host (Ubuntu, Debian)

## Dockerfile Basics

A **Dockerfile** is a text file containing a set of instructions on how to build a container image. This section covers basic Dockerfile instructions, some advanced instructions, and practical ways to use instructions when building container images for deployment in OpenShift.

Each instruction in a Dockerfile results in a new layer for the final container image.

**RUN** Instruction - Executes commands in a new layer on top of the current image and then commits the results. The results are then used in the next step in the Dockerfile. The container build process uses /bin/sh to execute commands.

**Tip**: Use the && command separator to execute multiple commands within a single RUN instruction.

**LABEL** Instruction - Defines image metadata. Labels are key-value pairs attached to an image. OpenShift can parse labels to run commands.

![openshift-supported-labels](/Building_With_Dockerfiles/images/supported-dockerfile-labels.png)

**WORKDIR** Instruction – Sets the working directory for the following instructions: RUN, CMD, ENTRYPOINT, COPY, and ADD 

**ENV** Instruction – Defines environment variables that will be available to the container.

**USER** Instruction – By default, OpenShift does not honor the USER instruction and uses a random user ID other than root to run containers.

**VOLUME** Instruction – Creates a mount point inside the container and indicates to image consumers that externally mounted volumes can bind to this mount point. Use this instruction for all persistent data needs.

**ONBUILD** Instruction – Registers *triggers* in the container image. Use this to declare instructions executed only when building a child image. Often useful for preloading data or providing custom configurations to an app. The parent image provides commands that are common to all downstream child images.

 **Note**: ONBUILD is not currently supported by Podman or Buildah. Use the –format docker option to enable support.

### Adapting Dockerfiles for OpenShift

When writing applications to run on OpenShift, you need to address the following:

* Directories and files that are read from or written to by processes in the container should be owned by **root** and have read/write access (i.e. a server's logging)
* Files that are executable should have group execute permissions
* The processes running in the container must not listen on privileged ports (ports below 1024) because they are not running as privileged users

### How to Run Containers as Root in OpenShift

If you must build and run an application as root, you can disable one of OpenShift's Security Context Constraints (SCCs) to allow root access and ignore the random *userid* set for the container. To do so, **create a service account and add it to the appropriate SCC** (*anyuid* instead of *restricted*). 

1. Create a SA
   * `oc create sa myserviceaccount`
2. Modify the DeploymentConfig for the app to use the new SA, using `oc patch`
   * `oc patch dc/demo-app --path '{"spec": {"template": {"spec": {"serviceAccountName": "myserviceaccount"}}}}'`
3. Add the SA to the `anyuid` SSC to run using a fixed userid in the container
   * `oc adm policy add-scc-to-user anyuid -z myserviceaccount`

## Demo: Building a Dockerfile Image with Advanced Instructions

| Topic                                                  | Duration | Link                                                         |
| ------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| Building a Dockerfile Image with Advanced Instructions | 4:47 min | <video src="Building Dockerfiles with Advanced Instructions.mp4"></video> |

# Additional Resources

* [Docker - Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
* [Wikipedia - RHEL](https://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux)

