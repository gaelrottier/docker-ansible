Docker-Ansible base images
===================

[![Circle CI](https://circleci.com/gh/William-Yeh/docker-ansible.svg?style=shield)](https://circleci.com/gh/William-Yeh/docker-ansible) [![Build Status](https://travis-ci.org/William-Yeh/docker-ansible.svg?branch=master)](https://travis-ci.org/William-Yeh/docker-ansible)


## Summary

Repository name in Docker Hub: **[williamyeh/ansible](https://registry.hub.docker.com/u/williamyeh/ansible/)**

This repository contains Dockerized [Ansible](https://github.com/ansible/ansible), published to the public [Docker Hub](https://registry.hub.docker.com/) via **automated build** mechanism.



## Configuration

These are Docker images for [Ansible](https://github.com/ansible/ansible) software, installed in a selected Linux distributions.

- OS: Debian (jessie, wheezy), Ubuntu (trusty, precise), CentOS (7, 6), Alpine (3).

- Ansible: usually the most recent *stable* and *experimental* versions (I didn't pin any specific version).


## Images and tags

### Stable series (installed from official PyPI repo):

- Normal series:

  - `williamyeh/ansible:debian8`
  - `williamyeh/ansible:debian7`
  - `williamyeh/ansible:ubuntu14.04`
  - `williamyeh/ansible:ubuntu12.04`
  - `williamyeh/ansible:centos7`
  - `williamyeh/ansible:centos6`
  - `williamyeh/ansible:alpine3`

- Onbuild series (*recommended for common cases*):

  - `williamyeh/ansible:debian8-onbuild`
  - `williamyeh/ansible:debian7-onbuild`
  - `williamyeh/ansible:ubuntu14.04-onbuild`
  - `williamyeh/ansible:ubuntu12.04-onbuild`
  - `williamyeh/ansible:centos7-onbuild`
  - `williamyeh/ansible:centos6-onbuild`
  - `williamyeh/ansible:alpine3-onbuild`

### Experimental series (building from the git `master` source tree):

- Normal series:

  - `williamyeh/ansible:master-debian8`
  - `williamyeh/ansible:master-debian7`
  - `williamyeh/ansible:master-ubuntu14.04`
  - `williamyeh/ansible:master-ubuntu12.04`
  - `williamyeh/ansible:master-centos7`
  - `williamyeh/ansible:master-centos6`

- Onbuild series (*recommended for common cases*):

  - `williamyeh/ansible:master-debian8-onbuild`
  - `williamyeh/ansible:master-debian7-onbuild`
  - `williamyeh/ansible:master-ubuntu14.04-onbuild`
  - `williamyeh/ansible:master-ubuntu12.04-onbuild`
  - `williamyeh/ansible:master-centos7-onbuild`
  - `williamyeh/ansible:master-centos6-onbuild`



## For the impatient

Here comes a simplest working example for the impatient.

First, choose a base image you'd like to begin with. For example, `williamyeh/ansible:ubuntu14.04-onbuild`.

Second, put the following `Dockerfile` along with your playbook directory:

```
FROM williamyeh/ansible:ubuntu14.04-onbuild

# ==> Specify requirements filename;  default = "requirements.yml"
#ENV REQUIREMENTS  requirements.yml

# ==> Specify playbook filename;      default = "playbook.yml"
#ENV PLAYBOOK      playbook.yml

# ==> Specify inventory filename;     default = "/etc/ansible/hosts"
#ENV INVENTORY     inventory.ini

# ==> Executing Ansible (with a simple wrapper)...
RUN ansible-playbook-wrapper
```

Third, `docker build .`

Done!

For more advanced usage, the role in Ansible Galaxy [`williamyeh/nginx`](https://galaxy.ansible.com/list#/roles/2245) demonstrates how to perform a simple smoke test (*configuration needs test, too!*) on a variety of (*containerized*) Linux distributions via [CircleCI](https://circleci.com/)'s Ubuntu 12.04 and [Travis CI](https://travis-ci.org/)’s Ubuntu 14.04 worker instances.




## Why yet another Ansible image for Docker?

There has been quite a few Ansible images for Docker (e.g., [search](https://registry.hub.docker.com/search?q=ansible) in the Docker Hub), so why reinvent the wheel?

In the beginning I used the [`ansible/ansible-docker-base`](https://github.com/ansible/ansible-docker-base) created by Ansible Inc. It worked well, but left some room for improvement:

- *Base OS image* - It provides only `centos:centos7` and `ubuntu:14.04`.  Insufficent for me.

- *Unnecessary dependencies* - It installed, at the very beginning of its Dockerfile, the `software-properties-common` package, which in turns installed some Python packages. I prefered to incorporate these stuff only when absolutely needed.

Therefore, I built these Docker images on my own.

**NOTE:** [`ansible/ansible-docker-base`](https://github.com/ansible/ansible-docker-base) announced in September 2015: “Ansible no longer maintains images in Dockerhub directly.”

### Comparison: image size

```
REPOSITORY                    TAG                   VIRTUAL SIZE
---------------------------   -------------------   ------------
ansible/centos7-ansible       stable                367.5 MB
ansible/ubuntu14.04-ansible   stable                286.6 MB

williamyeh/ansible            alpine3-onbuild        66.4 MB
williamyeh/ansible            centos6-onbuild       264.2 MB
williamyeh/ansible            centos7-onbuild       275.3 MB
williamyeh/ansible            debian7-onbuild       134.4 MB
williamyeh/ansible            debian8-onbuild       178.3 MB
williamyeh/ansible            ubuntu12.04-onbuild   181.9 MB
williamyeh/ansible            ubuntu14.04-onbuild   238.3 MB
```


## Usage

Used mostly as a *base image* for configuring other software stack on some specified Linux distribution(s).

Take Debian/Ubuntu/CentOS for example. To test an Ansible `playbook.yml` against a variety of Linux distributions, we may use [Vagrant](https://www.vagrantup.com/) as follows:

```ruby
# Vagrantfile

Vagrant.configure(2) do |config|

    # ==> Choose a Vagrant box to emulate Linux distribution...
    config.vm.box = "ubuntu/trusty64"
    #config.vm.box = "ubuntu/precise64"
    #config.vm.box = "debian/jessie64"
    #config.vm.box = "debian/wheezy64"
    #config.vm.box = "bento/centos-7.1"
    #config.vm.box = "bento/centos-6.7"
    #config.vm.box = "maier/alpine-3.1.3-x86_64"


    # ==> Executing Ansible...
    config.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
    end

end
```

Virtual machines can emulate a variety of Linux distributions with good quality, at the cost of runtime overhead.


Docker to be a rescue. Now, with these **williamyeh/ansible** series, we may test an Ansible `playbook.yml` against a variety of Linux distributions as follows:


```dockerfile
# Dockerfile

# ==> Choose a base image to emulate Linux distribution...
FROM williamyeh/ansible:ubuntu14.04
#FROM williamyeh/ansible:ubuntu12.04
#FROM williamyeh/ansible:debian8
#FROM williamyeh/ansible:debian7
#FROM williamyeh/ansible:centos7
#FROM williamyeh/ansible:centos6
#FROM williamyeh/ansible:alpine3


# ==> Copying Ansible playbook...
WORKDIR /tmp
COPY  .  /tmp

# ==> Creating inventory file...
RUN echo localhost > inventory

# ==> Executing Ansible...
RUN ansible-playbook -i inventory playbook.yml \
      --connection=local --sudo
```

You may also work with `onbuild` series, which take care of many routine steps for you:

```dockerfile
# Dockerfile

# ==> Choose a base image to emulate Linux distribution...
FROM williamyeh/ansible:ubuntu14.04-onbuild
#FROM williamyeh/ansible:ubuntu12.04-onbuild
#FROM williamyeh/ansible:debian8-onbuild
#FROM williamyeh/ansible:debian7-onbuild
#FROM williamyeh/ansible:centos7-onbuild
#FROM williamyeh/ansible:centos6-onbuild
#FROM williamyeh/ansible:alpine3-onbuild


# ==> Specify requirements filename;  default = "requirements.yml"
#ENV REQUIREMENTS  requirements.yml

# ==> Specify playbook filename;      default = "playbook.yml"
#ENV PLAYBOOK      playbook.yml

# ==> Specify inventory filename;     default = "/etc/ansible/hosts"
#ENV INVENTORY     inventory.ini

# ==> Executing Ansible (with a simple wrapper)...
RUN ansible-playbook-wrapper
```



With Docker, we can test any Ansible playbook against any version of any Linux distribution without the help of Vagrant. More lightweight, and more portable across IaaS, PaaS, and even CaaS (Container as a Service) providers!

If better OS emulation (virtualization) isn't required, the Docker approach (containerization) should give you a more efficient Ansible experience.



## License

Author: William Yeh <william.pjyeh@gmail.com>

Licensed under the Apache License V2.0. See the [LICENSE file](LICENSE) for details.
