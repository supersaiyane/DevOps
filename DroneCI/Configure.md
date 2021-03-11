# Preparation

## Create an OAuth application

Create a GitHub OAuth application. The Consumer Key and Consumer Secret are used to authorize access to GitHub resources.The authorization callback URL must match the below format and path, and must use your exact server scheme and host.

![](https://github.com/supersaiyane/DevOps/blob/7862790fef7222a96ef3336223da838aacc9aa31/DroneCI/resources/github_application_create.png)
![][https://github.com/supersaiyane/devops/blob/master/droneci/resources/github_application_created.png]

## Create a Shared Secret

Create a shared secret to authenticate communication between runners and your central Drone server.

You can use openssl to generate a shared secret:

```
$ openssl rand -hex 16
bea26a2221fd8090ea38720fc445eca6

```

# Download

The Drone server is distributed as a lightweight Docker image. The image is self-contained and does not have any external dependencies.

```
$ docker pull drone/drone:1
```

## Configuration

The Drone server is configured using environment variables. This article references a subset of configuration options, defined below. See [Configuration](https://docs.drone.io/server/reference/) for a complete list of configuration options.

- _DRONE_GITHUB_CLIENT_ID_ Required string value provides your GitHub oauth Client ID generated in the previous step.
- _DRONE_GITHUB_CLIENT_SECRET_ Required string value provides your GitHub oauth Client Secret generated in the previous step.
- _DRONE_RPC_SECRET_ Required string value provides the shared secret generated in the previous step. This is used to authenticate the rpc connection between the server and runners. The server and runner must be provided the same secret value.
- _DRONE_SERVER_HOST_ Required string value provides your external hostname or IP address. If using an IP address you may include the port. For example drone.company.com.
- _DRONE_SERVER_PROTO_ Required string value provides your external protocol scheme. This value should be set to http or https. This field defaults to https if you configure ssl or acme.

## Start the Server

The server container can be started with the below command. The container is configured through environment variables. For a full list of configuration parameters, please see the configuration reference.

```
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID={{DRONE_GITHUB_CLIENT_ID}} \
  --env=DRONE_GITHUB_CLIENT_SECRET={{DRONE_GITHUB_CLIENT_SECRET}} \
  --env=DRONE_RPC_SECRET={{DRONE_RPC_SECRET}} \
  --env=DRONE_SERVER_HOST={{DRONE_SERVER_HOST}} \
  --env=DRONE_SERVER_PROTO={{DRONE_SERVER_PROTO}} \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1
```
