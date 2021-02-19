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
RUN echo "[local]" >> /etc/ansible/hosts && \
    echo "localhost" >> /etc/ansible/hosts && \
    echo "ansible@192.168.140.193" >> /etc/ansible/hosts

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

COPY $USER_HOME/id_rsa /home/ansible/.ssh/id_rsa
COPY $USER_HOME/id_rsa.pub /home/ansible/.ssh/id_rsa.pub
RUN sudo chown ansible /home/ansible/.ssh/id_rsa.pub
RUN sudo chown ansible /home/ansible/.ssh/id_rsa

ENTRYPOINT ["ansible-playbook"]
