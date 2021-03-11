# kubernetes_jenkins

Contents of the repository

- [Dockerfile - to build the initial image](https://gitlab.com/gurpreet.singh_verti/kubernetes_jenkins/-/blob/master/Dockerfile)
- [docker-compose - build final image](https://gitlab.com/gurpreet.singh_verti/kubernetes_jenkins/-/blob/master/docker-compose.yml)
- [plugins.txt - holds the list of plugins](https://gitlab.com/gurpreet.singh_verti/kubernetes_jenkins/-/blob/master/plugins.txt)
- [1_jenkins_deployment.yaml](url)
- [2_jenkins_rbac.yaml](url)
- [3_jenkins_service.yaml](url)

This project holds the modified build of Jenkins which covers the

1. Elimination of the problem of Docker <Used-Docker-in-Docker>
2. Added Ansible (Deploying the Application using automation)
3. Added tools like GIT/VIM/SUDO/Python3.5/PIP3
4. Skipped login process
5. This image is ment for scaling purpose on kubernetes.
6. Created user <admin> with pass <admin> with role <admin>

# Usage

Create an image using Dockerfile

```
 docker build -t <image_name> .
```

use the image name generated above in the Docker-Compose file in jenkins: image_name and run the command below

```
docker-compose up -d
```

# Extras

if you want to install plugins first hand then

1. Put all the plugins in the plugin file.
2. Uncomment the last two line from the Dockerfile and create an image using
   ```
   docker build -t <image_name> .
   ```
3. use this image name in the docker-compose file and run
   ```
   docker-compose up -d
   ```
