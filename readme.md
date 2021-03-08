# How to run ansible playbooks on Docker.
Two possible ways:
- WSL2 - copy: Only for WSL2 use.
- Linux - Volumes: A much safer way. 

**WhatToDo.txt** files have simple instructions and you can find more detailed instructions in **readme.md** -files.
  Quickstart (below) has main commands.

# Quickstart
## WSL2 version, done using COPY in Dockerfile:
Used COPY because apparently using volumes to send SSH-keys doesn't work in WSL2.

I did this with **W10 computer** that has **WSL2** and **Ubuntu 18.04 distro**.  
My host-machine is **CentOS 7**.

**Main commands:**

Creates ssh-keys:
```
sudo ssh-keygen 
```
Build Docker Image:
```
sudo docker build -t ansible3:3.44 .
```
Runs the sshkey.yml -playbook that sends SSH-keys to hosts:
```
sudo docker run --rm -it -v /home/ansible/docker:/ansible/playbooks ansible3:3.44 --private-key=/home/ansible/.ssh/id_rsa sshkey.yml -Kk
```
Runs testUpdate.yml -playbook, this si how you run playbooks:
```
sudo docker run --rm -it -v /home/ansible/docker:/ansible/playbooks ansible3:3.44 --private-key=/home/ansible/.ssh/id_rsa testiUpdate.yml
```
##### If build command doesn't work in WSL2:
Solution is:  
In **~/.docker/config.json** change **credsStore** to **credStore**

  
## Linux version using volumes (doesn't work on wsl2)

I used volumes here to send ssh-keys and hosts-file to container on run.
Much better way than previous COPY one, bc of safety.

I did this with **CentOS8 Stream** virtual machine. 
Host was **CentOS 7** virtual machine.

**Main commands:**

Creates ssh-keys:
```
sudo ssh-keygen 
```
Chown ssh-keys:
```
sudo chown 1000 id_rsa
sudo chown 1000 id_rsa.pub 
```
Build Docker Image:
```
sudo docker build -t ansible1:1.0 .
```
Runs the sshkey.yml -playbook that sends SSH-keys to hosts:
```
sudo docker run --rm -it -v /home/ansible/docker/id_rsa:/home/ansible/.ssh/id_rsa -v /home/ansible/docker/id_rsa.pub:/home/ansible/.ssh/id_rsa.pub -v /home/ansible/docker/hosts:/etc/ansible/hosts -v /home/ansible/docker:/ansible/playbooks ansible1:1.0 --private-key=/home/ansible/docker/.ssh/id_rsa sshkey.yml -Kk
```
Runs testUpdate.yml -playbook, this is how you run playbooks:
```
 sudo docker run --rm -it -v /home/ansible/docker/id_rsa:/home/ansible/.ssh/id_rsa -v /home/ansible/docker/id_rsa.pub:/home/ansible/.ssh/id_rsa.pub -v /home/ansible/docker/hosts:/etc/ansible/hosts -v /home/ansible/docker:/ansible/playbooks ansible1:1.0 --private-key=/home/ansible/docker/.ssh/id_rsa testiupdate.yml
```