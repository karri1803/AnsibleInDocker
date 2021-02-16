# Few tests about running ansible playbooks on Docker.
The Main test is testi3, others are for alternate ideas.

## build komento
Containers\testi3> docker build -t ansible3:3.16 --network=host .

## Ilman ssh-key voluumeita
Containers\testi3> docker run --rm -it -v ${pwd}:/ansible/playbooks ansible3:3.16 testiUpdate.yml

## ssh-key voluumeilla
Containers\testi3> docker run --rm -it -v ~/.ssh/private_key/id_rsa:/root/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/root/.ssh/private_key/id_rsa.pub -v ${pwd}:/ansible/playbooks ansible3:3.16 testiUpdate.yml
