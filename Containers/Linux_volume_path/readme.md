    
# Detailed instructions for Linux_volume_path     

This way you can install Ansible inside of a Docker container with few commands and use it directly with one run command. Linux distro is needed. Doesn't work on WSL2. 
These instructions made using **CentOS8 Stream** virtual machine.
   
Basic assumption: You already have some Linux distro and docker installed and ready to use.
     
  
- Create user ansible:
```
adduser ansible
```
- Change user:
```
su - ansible
```
- Create directory docker for the user:
```
mkdir /home/ansible/docker
```
- Move in to the directory docker and use it now on. Create all the files and SSH-keys inside it.
```
cd docker
```
- Create Dockerfile, which will build the image:

For example:
```
 sudo nano Dockerfile
```
**Dockerfile**: 

> Installs Alpine (3.7) into the image.  
> Installs required packages.  
> Creates hosts -file folder.  
> Installs Ansible.  
> Creates directory /ansible/playbooks and sets it as working directory.  
> Sets required Environment arguments.  
> Creates user ansible, takes it to sudoers -file and sets it as used user.  
> Creates directory /home/ansible/.ssh .    
> Creates Entrypoint: ansible-playbook .  

```
FROM alpine:3.7
ARG VERSION=2.9.3
ARG USER_HOME

RUN \
# apk add installs the following
apk add \
	curl \
	python \
	py-pip \
	py-boto \
	py-dateutil \
	py-httplib2 \
	py-jinja2 \
	py-paramiko \
	py-setuptools \
	py-yaml \
	openssh-client \
        sshpass \
	sudo \
	bash \
	tar && \
 pip install --upgrade pip

RUN mkdir /etc/ansible/ /ansible

RUN \
  curl -fsSL https://releases.ansible.com/ansible/ansible-${VERSION}.tar.gz -o ansible.tar.gz && \
  tar -xzf ansible.tar.gz -C ansible --strip-components 1 && \
  rm -fr ansible.tar.gz /ansible/docs /ansible/examples /ansible/packaging

RUN mkdir -p /ansible/playbooks
WORKDIR /ansible/playbooks

ENV ANSIBLE_GATHERING smart
ENV ANSIBLE_HOST_KEY_CHECKING false
ENV ANSIBLE_RETRY_FILES_ENABLED false
ENV ANSIBLE_ROLES_PATH /ansible/playbooks/roles
ENV ANSIBLE_SSH_PIPELINING True
ENV PATH /ansible/bin:$PATH
ENV PYTHONPATH /ansible/lib

RUN adduser -u 1000 -h /home/ansible -D ansible
RUN passwd -d ansible
RUN echo "ansible ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER ansible

RUN mkdir /home/ansible/.ssh

ENTRYPOINT ["ansible-playbook"]
```

- Then let's create required playbooks. First, playbook that sends SSH-keys to hosts. 
Second, playbook that tests if everything works.

For example: 
```
sudo nano sshkey.yml
```
**sshkey.yml:**

> Ansible-playbook -file, that sends the SSH-keys from Docker container to hosts.

```
---
- hosts: all
  become: true

  tasks:

    - name: Send sshkey
      authorized_key:
        user: ansible
        state: present
        key: "{{ lookup('file', '/home/ansible/.ssh/id_rsa.pub') }}"
```        
For example: 
```
sudo nano testiUpdate.yml
```
**testiUpdate.yml**

> Easy, simple playbook to test running playbooks without passwords. Installs security updates on host. 
requires CentOS host. Test some other playbook if some other distro.

```
---
 - hosts: all
   become: true  
   
   tasks:

     - name: install security updates
       yum:
         name: '*'
         security: yes
         state: latest
```
  
- Create SSH-keys, when location asked answer /home/ansible/docker :
```
sudo ssh-keygen
```
Chown ssh-keys (1000 is my ansible user's UID in container):
```
sudo chown 1000 id_rsa
sudo chown 1000 id_rsa.pub 
```
- Build Image (Give some name, here I use ansible1 and version number 1.0):
```
sudo docker build -t ansible1:1.0 .
```
- Run sshkey.yml, at this part you will be asked for host passwords:
```
sudo docker run --rm -it -v /home/ansible/docker/id_rsa:/home/ansible/.ssh/id_rsa -v /home/ansible/docker/id_rsa.pub:/home/ansible/.ssh/id_rsa.pub -v /home/ansible/docker/hosts:/etc/ansible/hosts -v /home/ansible/docker:/ansible/playbooks ansible1:1.0 --private-key=/home/ansible/docker/.ssh/id_rsa sshkey.yml -Kk
```
- Run testiUpdate.yml
```
 sudo docker run --rm -it -v /home/ansible/docker/id_rsa:/home/ansible/.ssh/id_rsa -v /home/ansible/docker/id_rsa.pub:/home/ansible/.ssh/id_rsa.pub -v /home/ansible/docker/hosts:/etc/ansible/hosts -v /home/ansible/docker:/ansible/playbooks ansible1:1.0 --private-key=/home/ansible/docker/.ssh/id_rsa testiupdate.yml
```
- You can now run playbooks using this run command. just change the name of the playbook from the end of the command. :)      