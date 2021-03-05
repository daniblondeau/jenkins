# Jenkins on Docker running containers

## Network

1. Create a bridge network in Docker using the following docker network create command:

```
sudo docker network create jenkins

81564ee8346481240f2b6284f343af12ce6c7ceff59695684cf474e73e14b762
```

## Docker in Docker

2. In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image using the following link:
https://docs.docker.com/engine/reference/run/ [docker run] command:

```
sudo docker run \
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
  docker:dind
```
## Jenkins Docker

3. Customise official Jenkins Docker image, by executing below two steps:

  1.  Create Dockerfile with the following content:
      ```
      FROM jenkins/jenkins:2.249.3-slim
      USER root
      RUN apt-get update && apt-get install -y apt-transport-https \
             ca-certificates curl gnupg2 \
             software-properties-common
      RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
      RUN apt-key fingerprint 0EBFCD88
      RUN add-apt-repository \
             "deb [arch=amd64] https://download.docker.com/linux/debian \
             $(lsb_release -cs) stable"
      RUN apt-get update && apt-get install -y docker-ce-cli
      USER jenkins
      RUN jenkins-plugin-cli --plugins blueocean:1.24.3
      ```

    2. Build a new docker image from this Dockerfile and assign the image a meaningful name, e.g. "myjenkins-blueocean:1.1":
       ```
       sudo docker build -t myjenkins-blueocean:1.1 .
       ```
    >Note: Keep in mind that the process described above will automatically download the official Jenkins Docker image if this hasn’t been done before.


4. Run your own myjenkins-blueocean:1.1 image as a container in Docker using the following docker run command:

```
sudo docker run \
  --name jenkins-blueocean \
  --rm \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:1.1
```

5. Create a Pipeline
```
pipeline {
    agent {
        docker { image 'node:14-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```

6. Access to docker container and you will see that alpine image has been downloaded.
