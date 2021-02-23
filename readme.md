## Few tests about running ansible playbooks on Docker.
The newest version is Containers/wsl2_test .  
Older not (at the moment) working commands are at the bottom of this readme -file.


### WSL2 version, done using COPY in Dockerfile:
Used COPY because apparently using volumes to send SSH-keys doesn't work in WSL2.

I did this with W10 computer that has WSL2 and Ubuntu 18.04 distro.
My host-machine is CentOS 7.

All the commands:

```
(sudo?) ssh-keygen 
```
```
sudo docker build -t ansible3:3.44 .
```
```
sudo docker run --rm -it -v /home/ansible/docker:/ansible/playbooks ansible3:3.44 --private-key=/home/ansible/.ssh/id_rsa sshkey.yml -Kk
```
```
sudo docker run --rm -it -v /home/ansible/docker:/ansible/playbooks ansible3:3.44 --private-key=/home/ansible/.ssh/id_rsa testiUpdate.yml
```

#### If build command doesn't work in WSL2:
Solution is:  
In **~/.docker/config.json** change **credsStore** to **credStore**


## These are old and did not work for me, saved here for possible future use:
### build command (old)
```
Containers\testi3> docker build -t ansible3:3.16 --network=host .
```
### ssh-key without volumes (old)
```
Containers\testi3> docker run --rm -it -v ${pwd}:/ansible/playbooks ansible3:3.16 testiUpdate.yml
```
### ssh-key with volumes (old)
```
Containers\testi3> docker run --rm -it -v ~/.ssh/private_key/id_rsa:/root/.ssh/id_rsa -v ~/.ssh/private_key/id_rsa.pub:/root/.ssh/id_rsa.pub -v ${pwd}:/ansible/playbooks ansible3:3.16 testiUpdate.yml
```
