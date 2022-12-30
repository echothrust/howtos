---
category: Windows
---

# Setting up docker on windows for windows dockers
date: 13/10/2020

In order to be able to setup a docker server on a windows with the ability to run windows dockers you will need to install a version that supports Hyper-V such as:
* Win 10 pro
* Win server >=2012

XXXFIXME add link for HyperV supported versions.

## On ESXI VM
* Create a new vm and ensure the compatibility of the vm is set as "ESXI 6.5 virtual machine"
* Edit the vm settings and expand the CPU section and on the "CPU/MMU Virtualization" choose "Hardware CPU and MMU"
* Boot the vm and setup windows as needed.
* Install the windows updates as they are needed by Docker for Windows
* Download docker for windows and install. Follow the installation instructions and do as instructed
* Once docker is installed restart your vm
* right click on docker icon and choose "Switch to Windows containers..."
* restart vm again
* Open C:\ProgramData\docker\config\daemon.json and configure the following entries
```json
{
  "registry-mirrors": [],
  "insecure-registries": ["10.0.0.254:5000"],
  "host": ["tcp://0.0.0.0:2375"],
  "debug": false,
  "experimental": false,
  "bridge": "none"
}
```
* Restart the VM once more
* Open a command prompt and remove all existing networks
```
docker network prune
```
* Ensure you only have two networks left `nat` & `none`
* Create a new transparent network for our containers
```
docker network create -d transparent --subnet=10.0.0.0/16 --gateway=10.0.0.254 aanet
```
* Run a container on our newly created network `aanet` and assign IP from DHCP
```
docker run -it --network=aanet mcr.microsoft.com/windows/nanoserver:1903
```
* If dhcp is not running you can assign IP (eg `10.0.104.123`) to the container by running
```
docker run -it --ip=10.0.104.123 --network=aanet mcr.microsoft.com/windows/nanoserver:1903
```
