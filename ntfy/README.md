# NTFY + caddy for a secure deployment

This is an example config for my ntfy service that i expose via a VPS

# ntfy-config.yml

If you have experience with kubernetes deployments this should be no difference.

For those who just started learning podman or containers in general, this file is the equivalent of 2 config files.

## ntfy-config

contains the server.yml file with all the settings to make the server work

for more information about what settings is available go to the [official github example of the file](https://github.com/binwiederhier/ntfy/blob/main/server/server.yml)

## caddy-config

Contains the CaddyFile with the minial configuration needed to get the let's encrypt certificate

for more information about caddy head to their [offical website](https://caddyserver.com/docs/)

# ntfy-deploy.yml

This is the main file that contains most of what we are deploying with podman Quadlet

## components

### Volume (PersistentVolumeClaim)

The first part of the yaml file is to declare a volume that will be provisionned to the ntfy container for caching

### Containers (Pod)

A pod can contain multiple containers, the key detail here is that pods are like subnets or vlans that segragates networks so other pods can't contact other pods.

#### ntfy

The first container is ntfy.

we mount the configmap and volume that we mentionned before to their respective path

#### caddy

Caddy is pretty minimal, it only need the configmap and that's pretty much it

#### volumes

This is where we associate the configmaps and the volumes to a name or alias to that will tell podman to use these volumes or configmaps to the right place

# ntfy-deploy.network

This is where you declare the network for the pod.

**if the subnet and gateway collide with your local network change it to something else withing the private subnets ranges**

# ntfy-deploy.kube

This is the main file that stick everything together and create a service file with quadlet.

## Yaml

this is the main file that has all the components/containers

## Network

I use pasta since it's available since podman 5.0.0 but you can use slirp4netns which was supported before.

To configure pasta you simply need to do the command pasta --net-config and then exit the shell

## PublishPort

You can declare more than once this setting if you have more than one port to expore like this case 80 and 443

## ConfigMap

This is where you specify the configmaps yml file

** can only be declared once **
