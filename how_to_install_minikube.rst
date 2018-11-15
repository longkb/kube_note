=======================
How to install minikube
=======================

.. contents:: **Table of Contents**
   :depth: 2

Hypervisor choice for Minikube
==============================

Minikube supports both VirtualBox [1]_ and KVM [2]_ hypervisors. This guide will
cover both hypervisors.

Step 1: Update system
=====================

.. code-block:: console

    sudo apt-get update
    sudo apt-get install apt-transport-https
    sudo apt-get upgrade

Step 2: Install Hypervisor
==========================

For VirtualBox users, install VirtualBox using

.. code-block:: console

    sudo apt install virtualbox virtualbox-ext-pack

For KVM users, install KVM by following [3]_

* Install them using the commands

  .. code-block:: console

    sudo  apt-get -y install qemu-kvm libvirt-bin virt-top \
    libguestfs-tools virtinst bridge-utils

* Load and enable vhost-net module.

  .. code-block:: console

    sudo modprobe vhost_net
    sudo lsmod | grep vhost
    sudo echo vhost_net >> /etc/modules

* Install Docker-machine

  .. code-block:: console

      # Uninstall Old version of Docker. Old versions of docker had the name docker
      # or docker-engine. If you have it installed, first uninstall it.

      sudo apt-get remove docker docker-engine docker.io

      # Update the apt package index:
      sudo apt-get update

      # Install packages to allow apt to use a repository over HTTPS
      sudo apt-get install apt-transport-https ca-certificates \
      curl software-properties-common

      # Add Docker’s official GPG key
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

      # Add stable repository
      sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable"

      # Install docker ce
      sudo apt-get update
      sudo apt-get install docker-ce

* If you would like to use Docker as a non-root user, you should now consider
  adding your user to the “docker” group with something like

  .. code-block:: console

    sudo usermod -aG docker longkb

* Run the command below to see a version of docker installed.

  .. code-block:: console

    $ sudo sudo docker version

    Client:
     Version:           18.09.0
     API version:       1.39
     Go version:        go1.10.4
     Git commit:        4d60db4
     Built:             Wed Nov  7 00:48:57 2018
     OS/Arch:           linux/amd64
     Experimental:      false

    Server: Docker Engine - Community
     Engine:
      Version:          18.09.0
      API version:      1.39 (minimum version 1.12)
      Go version:       go1.10.4
      Git commit:       4d60db4
      Built:            Wed Nov  7 00:16:44 2018
      OS/Arch:          linux/amd64
      Experimental:     false

Step 3: Download minikube
=========================

You need to download the minikube binary. I will put the binary under
**/usr/local/bin** directory

.. code-block:: console

    curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    chmod +x minikube
    sudo mv -v minikube /usr/local/bin

Confirm version installed

.. code-block:: console

    $ minikube version
    minikube version: v0.30.0

Step 4: Install kubectl
=======================

We need kubectl which is a command line tool used to deploy and manage
applications on Kubernetes. To download the ‘kubectl’ binary file with **curl** [4]_

.. code-block:: console

    curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl \
    -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x kubectl
    sudo mv kubectl  /usr/local/bin/

Confirm version installed

.. code-block:: json

    $ kubectl version -o json
    {
      "clientVersion": {
        "major": "1",
        "minor": "12",
        "gitVersion": "v1.12.2",
        "gitCommit": "17c77c7898218073f14c8d573582e8d2313dc740",
        "gitTreeState": "clean",
        "buildDate": "2018-10-24T06:54:59Z",
        "goVersion": "go1.10.4",
        "compiler": "gc",
        "platform": "linux/amd64"
      }
    }

Step 4: Start Minikube
======================

1. Proxy configuration
----------------------

If your machine stands behind proxy ->
`click here <https://github.com/longkb/kube_note/blob/master/how_to_install_minikube.rst#fixing>`_
to get configuraion guide.


2. Start Minikube with Virtualbox
---------------------------------

.. code-block:: console

    $ minikube start

3. Start Minikube with KVM
--------------------------

KVM2 Driver plugin installation [5]_

* The KVM2 driver is intended to replace KVM driver. The KVM2 driver is
  maintained by the minikube team, and is built, tested and released with minikube.
* To install the KVM2 driver, first install and configure the prereqs:

  .. code-block:: console

    # Install libvirt and qemu-kvm on your system
    sudo apt install libvirt-clients libvirt-daemon-system qemu-kvm

    # Add yourself to the libvirt group so you don't need to sudo
    # NOTE: For older Debian/Ubuntu versions change the group to `libvirtd`
    sudo usermod -a -G libvirt $(whoami)

    # Update your current session for the group change to take effect
    # NOTE: For older Debian/Ubuntu versions change the group to `libvirtd`
    newgrp libvirt

  .. code-block:: console

    $ minikube start --vm-driver kvm2

    Starting local Kubernetes v1.10.0 cluster...
    Starting VM...
    Downloading Minikube ISO
    150.53 MB / 150.53 MB [============================================] 100.00% 0s
    Getting VM IP address...
    Moving files into cluster...
    Downloading kubeadm v1.10.0
    Downloading kubelet v1.10.0
    Finished Downloading kubeadm v1.10.0
    Finished Downloading kubelet v1.10.0
    Setting up certs...
    Connecting to cluster...
    Setting up kubeconfig...
    Starting cluster components...
    Kubectl is now configured to use the cluster.
    Loading cached images from config file.

  .. note::

    If you want to show deployment log, you can use `--logtostderr`

    .. code-block:: console

        $ minikube start --logtostderr

* Wait for the download and setup to finish then confirm that everything is
  working fine. You should see a running VM with a domain named minikube.

  .. code-block:: console

    $ sudo virsh list
     Id    Name                           State
    ----------------------------------------------------
     1     minikube                       running

Fixing
======

1. For those who stand behind proxy [6]_, [7]_


   Create a systemd drop-in directory for the docker service:

   .. code-block:: console

    sudo mkdir -p /etc/systemd/system/docker.service.d

   Create a file called **/etc/systemd/system/docker.service.d/http-proxy.conf**
   that adds the `HTTP_PROXY` environment variable:

   .. code-block:: console

    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80/"

   Or, if you are behind an HTTPS proxy server, create a file called
   **/etc/systemd/system/docker.service.d/https-proxy.conf** that adds the
   `HTTPS_PROXY` environment variable

   .. code-block:: console

    [Service]
    Environment="HTTPS_PROXY=https://proxy.example.com:443/"

   If you have internal Docker registries that you need to contact without
   proxying you can specify them via the NO_PROXY environment variable

   .. code-block:: console

    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,192.168.1.100"

   Or, if you are behind an HTTPS proxy server

   .. code-block:: console

    [Service]
    Environment="HTTPS_PROXY=https://proxy.example.com:443/" "NO_PROXY=localhost,127.0.0.1,192.168.1.100"

   Flush changes

   .. code-block:: console

    sudo systemctl daemon-reload


   Restart Docker

   .. code-block:: console

    sudo systemctl restart docker

   Verify that the configuration has been loaded

   .. code-block:: console

    systemctl show --property=Environment docker
    Environment=HTTP_PROXY=http://proxy.example.com:80/ NO_PROXY=localhost,127.0.0.1,192.168.1.100

2. Error while start VM [8]_

.. code-block:: console

  sudo rm -rf ~/.minikube

References
==========

.. [1] https://computingforgeeks.com/how-to-install-minikube-on-ubuntu-18-04/
.. [2] https://computingforgeeks.com/how-to-run-minikube-on-kvm/
.. [3] https://computingforgeeks.com/install-kvm-on-centos-7-ubuntu-16-04-debian-9-sles-12-arch-linux/
.. [4] https://linuxhint.com/install-minikube-ubuntu/
.. [5] https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm2-driver
.. [6] https://codefarm.me/2018/08/09/http-proxy-docker-minikube/
.. [7] https://docs.docker.com/config/daemon/systemd/#httphttps-proxy
.. [8] https://github.com/kubernetes/minikube/issues/2412#issuecomment-361243014
