#This image will have GIT/VIM/SUDO/Python3.5/PIP3/ANSIBLE/DOCKER-ce

FROM jenkins/jenkins:lts
USER root

RUN uname -a && cat /etc/*release

# Update apt packages
RUN apt-get -qq update && apt-get -qqy install \
    apt-transport-https \
    ca-certificates \
    lxc \
    iptables \
    curl \
    gnupg2 \
    software-properties-common

#Checking any previous version of python, removing and installing latest
RUN python --version
RUN apt-get remove --auto-remove python2.7 -y
RUN apt purge -y python2.7

#Addign Python repository
RUN apt-get -y update
RUN add-apt-repository ppa:deadsnakes/ppa

# Install python 3.5
RUN apt-get -y install python3.5

# Make python 3.7 the default
RUN echo "alias python=python3.5" >> ~/.bashrc
RUN export PATH=${PATH}:/usr/bin/python3.5
RUN /bin/bash -c "source ~/.bashrc"

# Install pip
RUN apt install python3-pip -y
RUN python3 -m pip install --upgrade pip

# Add 3.5 to the available alternatives
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3.5 1

# Set python3.7 as the default python
RUN update-alternatives --set python /usr/bin/python3.5

# Makes the Ansible directories
RUN mkdir /etc/ansible /ansible
RUN mkdir ~/.ssh
RUN mkdir /opt/ansible
RUN mkdir /opt/ansible/inventory/
RUN mkdir /opt/ansible/inventory/production
RUN mkdir /opt/ansible/inventory/development
RUN mkdir /opt/ansible/inventory/qa
RUN mkdir /opt/ansible/inventory/stage

#install Ansible
RUN pip3 install ansible

# Sets environment variables
ENV ANSIBLE_GATHERING smart
ENV ANSIBLE_HOST_KEY_CHECKING False
ENV ANSIBLE_RETRY_FILES_ENABLED False
ENV ANSIBLE_ROLES_PATH /ansible/playbooks/roles
ENV ANSIBLE_SSH_PIPELINING True
ENV PATH /ansible/bin:$PATH
ENV PYTHONPATH /ansible/lib
ENV JAVA_OPTS: -Djenkins.install.runSetupWizard=false -Dmail.smtp.starttls.enable=true -Djenkins.model.Jenkins.crumbIssuerProxyCompatibility=true
ENV JENKINS_OPTS: --argumentsRealm.role.user=admin --argumentsRealm.passwd.admin=admin --argumentsRealm.role.admin=admin

#install docker
RUN curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh

# Docker Access to Jenkins User
RUN usermod -a -G root jenkins
RUN usermod -aG docker jenkins

#Install Sudo
RUN apt-get update \
    && apt-get install -y sudo \
    && rm -rf /var/lib/apt/lists/*

#install git
RUN apt-get -y install git

# Install vim
RUN apt-get -y install vim    

#Cheking Versions Intalled
RUN python3 --version
RUN pip3 --version
RUN git --version
RUN docker --versions
RUN ansible --versions

#Setting Reverse proxy and starttls for mail services
RUN JENKINS_JAVA_OPTIONS="-Dmail.smtp.starttls.enable=true"
RUN JENKINS_JAVA_OPTIONS="-Djenkins.model.Jenkins.crumbIssuerProxyCompatibility=true"

# Good idea to switch back to the jenkins user.
USER jenkins
#COPY plugins.txt /usr/share/jenkins/plugins.txt
#RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/plugins.txt