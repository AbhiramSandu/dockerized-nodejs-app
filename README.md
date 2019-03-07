# How to build docker images inside a Jenkins container.
Every now and then, I come across cases where I need to build docker images inside a Jenkins container, as part of CI/CD process. Although we can run docker inside docker using dind, but it’s not a recommended practice due to various risks of data corruption that comes with it( you can read the details about dind in this blog post)

After researching for sometime about this topic, I came across a solution which involves mounting the host machine’s docker socket to the Jenkins container, This way Jenkins can start new sibling containers( note, we are using the word siblings here instead of child containers due to the fact that, newly created container will run along side the Jenkins container rather then running inside Jenkins container).

We still need to install the docker client binaries inside our Jenkins container so that our container can talk to the mounted docker daemon.

Step-1: Build the docker image.
This Dockerfile will use a Jenkins container as a base image and then install the docker client , So that it can communicate with the docker daemon.

from jenkinsci/jenkins:lts
 
USER root
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common 
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get install docker-ce=17.12.1~ce-0~debian -y
RUN usermod -aG docker jenkins
(you can clone the Dockerfile from this github repo)

Build the image with below command

$ docker image build -t jenkins-docker .

Note: Change the version of docker inside the Dockerfile based on your own requirements.

Step-2: Start the container with mounted docker daemon
After successfully building the container in step 1 we need to run the container with mounted docker daemon

$ docker container run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock jenkins-docker

Note: The docker daemon running on host machine must be compatible with docker client running in Jenkins container. check the version of the host with docker --version and install the appropriate version of docker client inside the Jenkins container.

Conclusion:
Your Jenkins container now should have a functional docker installation. verify it by running

docker container ls

From inside the Jenkins container. The output should have same as running docker container ls from the docker host.

# Jenkins Building Docker Image and Sending to Registry
https://medium.com/@gustavo.guss/jenkins-building-docker-image-and-sending-to-registry-64b84ea45ee9
https://medium.com/@cryptolukas/you-should-add-this-as-last-stage-or-post-task-d69fb384a361
