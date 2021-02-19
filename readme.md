## Few tests about running ansible playbooks on Docker.
The newest version is wsl2_test. Others are really old.
Commands for that are at the bottom of this readme.md -file.

### build komento

```
Containers\testi3> docker build -t ansible3:3.16 --network=host .
```

### Ilman ssh-key voluumeita

```
Containers\testi3> docker run --rm -it -v ${pwd}:/ansible/playbooks ansible3:3.16 testiUpdate.yml
```

### ssh-key voluumeilla

```
Containers\testi3> docker run --rm -it -v ~/.ssh/private_key/id_rsa:/root/.ssh/id_rsa -v ~/.ssh/private_key/id_rsa.pub:/root/.ssh/id_rsa.pub -v ${pwd}:/ansible/playbooks ansible3:3.16 testiUpdate.yml
```

### WSL2 version, volumes do not work, so done using COPY in Dockerfile:
All the commands:

```
(sudo?) docker build -t ansible3:3.44 .

sudo docker run --rm -it -v /home/ansible/docker:/ansible/playbooks ansible3:1.42 --private-key=/home/ansible/.ssh/id_rsa sshkey.yml -Kk

sudo docker run --rm -it -v /home/ansible/docker:/ansible/playbooks ansible3:1.42 --private-key=/home/ansible/.ssh/id_rsa testi.yml
```
