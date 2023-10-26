# localjenkins
Contains local setup for Jenkins using Docker

Create a new network using command

```
docker network create jenkins
```

In order to execute Docker commands inside Jenkins nodes, first download and run the docker:dind Docker image using the following docker run command

```
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```

