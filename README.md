<!-- omit in toc -->
# Build and use your own Execution Environment

This page mainly shows you how to build and use your own custom Execution Environment.

<!-- omit in toc -->
## Table of Contents

- [Introduction](#introduction)
- [Build your own custom EE](#build-your-own-custom-ee)
  - [Environment in This Example](#environment-in-this-example)
  - [Install Ansible Builder](#install-ansible-builder)
  - [Prepare Required Files](#prepare-required-files)
  - [Build EE](#build-ee)
- [Use EE](#use-ee)
  - [Use EE in AWX](#use-ee-in-awx)
    - [Use EE in AWX with container registry](#use-ee-in-awx-with-container-registry)
    - [Use EE in AWX without container registry](#use-ee-in-awx-without-container-registry)
  - [Use EE in Ansible Runner](#use-ee-in-ansible-runner)
- [Tips](#tips)
  - [Get the `Dockerfile` of your custom EE](#get-the-dockerfile-of-your-custom-ee)
  - [Use Ansible collections without custom EE](#use-ansible-collections-without-custom-ee)

## Introduction

When the Job Template launched on AWX, the playbook runs on the Execution Environment (EE), which is a containerized environment completely isolated from your K3s host.

- [What's new in Ansible Automation Platform 2: automation execution environments](https://www.ansible.com/blog/whats-new-in-ansible-automation-platform-2-automation-execution-environments)

The default EE, [quay.io/ansible/awx-ee:latest](https://quay.io/repository/ansible/awx-ee), has some typical collections, Pip modules, and RPM packages by default, but sometimes your playbooks require additional (non-default) collections, modules or packages.

In such cases, you need to add Ansible collections, Python packages, and RPM packages to EE. There are two ways you can choose to achieve this.

1. **Build and use your own Execution Environment**
   - This method is used when additional Pip or RPM packages are required in addition to the Ansible collection.
   - Follow whole of this page to build and use custom EE.
2. **Place `collections/requirements.yml` in your project**
   - This is useful if you simply need additional Ansible collections only, and no additional Pip or RPM packages are required. However, this method cannot be used if your project type on AWX is Manual.
   - See [_Use Ansible collections without custom EE_](#use-ansible-collections-without-custom-ee) at the bottom of this page for instructions.

## Build your own custom EE

To build custom EE, there is a tool called Ansible Builder. You can build your own custom EE with any Ansible collections, Python packages, and RPM packages added.

- [ansible/ansible-builder](https://github.com/ansible/ansible-builder)
- [Introduction to Ansible Builder — ansible-builder documentation](https://ansible.readthedocs.io/projects/builder/en/latest/)

This repository includes ready-to-use files as an example to use Ansible Builder. You can clone my repository to start with my ready-to-use example files.

```bash
cd ~
git clone https://github.com/kurokobo/awx-on-k3s.git
cd awx-on-k3s/builder
```

### Environment in This Example

- CentOS Stream 8 (Minimal)
- Python 3.9
- Docker 20.10.17
- Ansible Builder 3.0.0

### Install Ansible Builder

You can install Ansible Builder by using Pip.

```bash
python3 -m pip install ansible-builder
```

### Prepare Required Files

At least, the file `execution-environment.yml` is required to build EE.

This repository contains [`execution-environment.yml` as a minimal ready-to-use example](execution-environment.yml). This file is made to achieve following requirements.

- Use `quay.io/centos/centos:stream9-minimal` as the base image
- Use Python `3.11` as Python interpreter
- Use Ansible `2.15.*` and Ansible Runner `2.3.*` to run playbooks on EE
- Add RPM Packages that listed in [`dependencies/bindep.txt`](dependencies/bindep.txt) for basic remote connection and debugging
- Add Python packages that listed in [`dependencies/requirements.txt`](ependencies/requirements.txt)
- Add Ansible collections that listed in [`dependencies/requirements.yml`](dependencies/requirements.yml)
- Customize configuration for `ansible-galaxy` (`additional_build_files` and `additional_build_steps`)
  - Add customized `ansible.cfg` to the context (`additional_build_files`)
  - Place customized file as `~/.ansible.cfg` for the stage that `ansible-galaxy` is invoked (`prepend_galaxy` under `additional_build_steps`)
- Run additional commands during build steps (`additional_build_steps`)
  - To allow the hard-coded interpreter name (`python3`) passed by AWX, `alternatives` command is appended under `append_base` to make the binary `/usr/bin/python3.11` executable as command `python3`

Note that since this example uses `*-minimal` image as the base image and added only few packages for SSH connection and debugging, there should be still missing packages and modules for some modules and collections.

You can review and modify [`execution-environment.yml`](execution-environment.yml) and any files referenced from this file to suit your requirements.

The syntax of `requirements.yml` is [the same as for Ansible Galaxy](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#install-multiple-collections-with-a-requirements-file). The syntax of `requirements.txt` is [the same as for Pip](https://pip.pypa.io/en/stable/reference/requirements-file-format/), and `bindep.txt` is [for Bindep](https://docs.opendev.org/opendev/bindep/latest/readme.html).

Other customization is possible besides this. Refer to [the official Ansible Builder documentation](https://ansible-builder.readthedocs.io/en/stable/) for details.

### Build EE

Once your files are ready, run `ansible-builder build` command to build EE as a container image according to the definition in `execution-environment.yml`. Specify a tag (`--tag`) to suit your requirements.

```bash
ansible-builder build --tag registry.example.com/ansible/ee:2.15-custom --container-runtime docker --verbosity 3
```

Below is an example output of this command.

```bash
$ ansible-builder build --tag registry.example.com/ansible/ee:2.15-custom --container-runtime docker --verbosity 3
Ansible Builder is generating your execution environment build context.
File context/_build/requirements.yml will be created.
File context/_build/requirements.txt will be created.
File context/_build/bindep.txt will be created.
Creating context/_build/configs
File context/_build/configs/ansible.cfg will be created.
File context/_build/scripts/assemble will be created.
File context/_build/scripts/install-from-bindep will be created.
File context/_build/scripts/introspect.py will be created.
File context/_build/scripts/check_galaxy will be created.
File context/_build/scripts/check_ansible will be created.
File context/_build/scripts/entrypoint will be created.
Ansible Builder is building your execution environment image. Tags: registry.example.com/ansible/ee:2.15-custom
Running command:
  docker build -f context/Dockerfile -t registry.example.com/ansible/ee:2.15-custom context
Sending build context to Docker daemon  50.18kB
Step 1/77 : ARG EE_BASE_IMAGE="quay.io/centos/centos:stream9-minimal"
...
Step 77/77 : CMD ["bash"]
 ---> Running in 2eff2dca84d2
Removing intermediate container 2eff2dca84d2
 ---> d804667597e9
Successfully built d804667597e9
Successfully tagged registry.example.com/ansible/ee:2.15-custom

Complete! The build context can be found at: /home/********/awx-on-k3s/builder/context
```

Once the command is complete, your custom EE image is built and stored on Docker or Podman.

```bash
$ docker image ls
REPOSITORY                        TAG               IMAGE ID       CREATED              SIZE
registry.example.com/ansible/ee   2.15-custom       d804667597e9   20 seconds ago       284MB
```

## Use EE

You can use your EE in your AWX or Ansible Runner.

### Use EE in AWX

To use your EE in AWX, in typical use cases, your EE should be stored on some container registry and should be pulled by AWX, but technically you can use your EE with your AWX without any container registry.

#### Use EE in AWX with container registry

Simply you can push your EE image to some container registry. Any registry can be acceptable. If you want to deploy your own private container registry, refer [additional guide on this repository](../registry).

```bash
$ docker push registry.example.com/ansible/ee:2.15-custom
The push refers to repository [registry.example.com/ansible/ee]
...
2.15-custom: digest: sha256:193b380672c64768231973c58a154fb8eec0f0aae1891e63aea3918308a06716 size: 3456
```

Then you can specify `registry.example.com/ansible/ee:2.15-custom` as your own custom EE in AWX. Specify registry credentials if your container registry requires authentication.

#### Use EE in AWX without container registry

The EEs in AWX are just Pods on Kubernetes. When executing a Job Template in AWX, AWX instructs Kubernetes to create a Pod using specified EE image. Of course Kubernetes requires the specified container image when creating the Pod. So while creating the Pod, Kubernetes usually pulls specified image from the container registry.

But this happens only if the image is not cached on the container runtime for Kubernetes. If Kubernetes has the cache of the specified image, Kubernetes never pulls the image from the container registry (Technically, this is the default behavior and stated as `imagePullPolicy` is `IfNotPresent`).

This means that if your Kubernetes has all the EE images you need in its cache in advance, you can use any EE from AWX without any container registry. To achieve this, you should export your EE image from Docker or Podman, and import it to containerd that the container runtime for K3s.

```bash
# Save your EE image as Tar file
docker save registry.example.com/ansible/ee:2.15-custom -o custom-ee.tar

# Import the Tar file to containerd
sudo /usr/local/bin/k3s ctr images import --compress-blobs --base-name registry.example.com/ansible/ee:2.15-custom custom-ee.tar
```

Ensure your imported image is listed.

```bash
$ sudo /usr/local/bin/k3s crictl images
IMAGE                             TAG           IMAGE ID            SIZE
...
registry.example.com/ansible/ee   2.15-custom   d804667597e9e       96.3MB
...
```

Now you can specify `registry.example.com/ansible/ee:2.15-custom` as your own custom EE in AWX without any container registry and any credentials.

In AWX, you can change the policy of pulling the image in `Edit` page of your EE. The default `Only pull the image if not present before running` is ok, but to be safe you should specify `Never pull container before running`.

### Use EE in Ansible Runner

Refer [the guide for Ansible Runner on this repository](../runner).

## Tips

Some additional information around Ansible Builder and EE.

### Get the `Dockerfile` of your custom EE

The `Dockerfile` is generated and stored under the `context` directory once your `ansible-builder build` command is completed.

```bash
$ cat context/Dockerfile
ARG EE_BASE_IMAGE="quay.io/centos/centos:stream9-minimal"
...

# Base build stage
FROM $EE_BASE_IMAGE as base
...

# Galaxy build stage
FROM base as galaxy
...

# Builder build stage
FROM base as builder
...

# Final build stage
FROM base as final
...
```

If you want to create `Dockerfile` without building image, you can use `ansible-builder create` command.

```bash
$ ansible-builder create --verbosity 3
Ansible Builder is generating your execution environment build context.
File context/_build/requirements.yml will be created.
File context/_build/requirements.txt will be created.
File context/_build/bindep.txt will be created.
Creating context/_build/configs
File context/_build/configs/ansible.cfg will be created.
File context/_build/scripts/assemble will be created.
File context/_build/scripts/install-from-bindep will be created.
File context/_build/scripts/introspect.py will be created.
File context/_build/scripts/check_galaxy will be created.
File context/_build/scripts/check_ansible will be created.
File context/_build/scripts/entrypoint will be created.
Complete! The build context can be found at: /home/********/awx-on-k3s/builder/context
```

### Use Ansible collections without custom EE

If you simply need additional Ansible collections only, and no additional Pip or RPM packages are required, this method is the simplest way. However, this method can't be applied for Manual type projects. If you want to add your project as Manual type, you should build custom EE.

The procedure is quite simple, create and place `collections/requirements.yml` on your project root including the list of the collections which you want to use.

The format of `collections/requirements.yml` is the same as [the `requirements.yml` for ansible-galaxy](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#install-multiple-collections-with-a-requirements-file).

If `collections/requirements.yml` is present in your project, AWX will install the collections accordingly while updating project.
