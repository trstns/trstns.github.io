---
layout: post
title: Setting up Docker in a home lab
categories: [Homelab, Docker]
---
Being an Windows admin I'm new to docker and containerisation, but I'm interested in getting my head around it all.  To accomplish this I have set up a VM at home to host docker containers.  This is what I've done.

I'm setting everything up in a Hyper-V VM, but this information should easily apply to everything else (VMware, Bare-metal, etc)
I chose to use RancherOS as my collegues at work use Rancher, and it is lightweight and designed for docker.  I can then run Rancher on top to simplify management of the containers.

Firstly, create a new VM with 4Gb of memory.  This may need to be increased depending on what you need to run, but it is a good starting point.
Grab the iso from [here](https://github.com/rancher/os/releases/) (the Hyper-V variant in my case) and boot it into the new VM.

I followed [this guide](https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/server/install-to-disk/) on the rancher site, but i'll outline the important steps below.

- If you don't have an SSH key, generate a new one by using ssh-keygen:

        $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

- Create a file called cloud-config.yml with the following contents (replacing "AAA..." with your public ssh key):

        #cloud-config
        hostname: <desired hostname>
        ssh_authorized_keys:
          - ssh-rsa AAA...

- Place the cloud-config.yml file somewhere you can get it from the rancher server (I used a web server)
- Install rancher to the hard-disk:

        $ sudo ros install -c https://link/to/cloud-config.yml -d /dev/sda


Once it has been installed, you will need to eject the install ISO and then reboot the server.
You can now ssh into the rancher server to install containers:

    $ ssh -i /path/to/private/key rancher@<ip-address>


## Installing Rancher in single-node mode

- Install Rancher server

        sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server

- Follow prompts to add host - I added my existing host for testing/simplicity.

## Adding services to Rancher



## Backing up docker volumes

It is very important to backup the persistant data for your containers in case something should happen to you server.
Rancher has a scheduler service that can be used to start a container on a schedule. 

First we need to enable and start the scheduler service from the console of the server.

    $ sudo ros service enable crontab
    $ sudo ros service up crontab

Rancher backup container - when added as a sidekick it can backup on schedule
https://github.com/mathuin/rancher-backup

https://rancher.com/docs/os/v1.1/en/system-services/custom-system-services/

<!--
## Installing Rancher in multi-node mode

The next step is to install rancher in a container so that we have a nice web interface for managing our containers.

    $ sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher


## Configuring Rancher

- Open into the Rancher web page (https://\<IP of rancher server\>)
- Enter the admin password you would like
- Verify the server address

Now we need to create a cluster and add nodes to it.

Click on add cluster.  I chose a custom cluster as I was running it on my own server, not in the cloud.  Enter a name for the cluster.  I left the rest of the options as default for now.

You are then given a command you need to run on any nodes you would like to add to the cluster.  For simplicity, I'm runnining this entire cluster on one VM, so I needed to select all of the node roles (etcd, Control Plane and Worker).  If/when you want to scale out your cluster, you should separate these roles onto different servers.
-->


## Upgrading Rancher OS

``$ sudo ros os list`` will list available versions of Rancher OS.

``$ sudo ros os version`` will show the current version of Rancher OS.

``$ sudo ros os upgrade`` will upgrade version of Rancher OS.
