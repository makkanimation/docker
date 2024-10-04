docker -v

# start docker service
systemctl start docker.service
# status of docker runing or not
systemctl status docker.service
# see docker container
docker ps
# permission denied while trying to connect to the Docker daemon socket at (Error)
![image](https://github.com/user-attachments/assets/647154f1-1203-483d-ba9a-88f41a2f6ad1)


```bash
# Afrer creating dockerfile (tutorial mengtion under dockerfile.md), start docker
docker run -p 3000:3000 imageid
#To freeup command prompt
docker run -d -p 3000:3000 imageid
# To run multiple port
docker run -p <host_port1>:<container_port1> -p <host_port2>:<container_port2>
docker ps -a # to see all docker
# to remove all unwanted docker either use
docker rm imageid
# or run docker with --rm
docker run -d --rm -p 3000:3000 imageid
# to put docker name
docker run -d -p --rm --name "mywebapp" 3000:3000 imageid
# to stop docker
docker ps
docker stop NAMES # showing under docker ps command
# to remove docker image with version
docker rmi repositoryname:version
```
# Update docker
```bash
#Simply build with new version
docker build mywebapp:02 .
#then run with reponame:version
docker run -d --rm --name "mywebapp" -p 3000:3000 reponame:version
```
# Push image to docker hub
```bash
# first login
docker login
# create image same mention under Docker hub commands
docker build -t app-created-in-hub:version .
# now run push command to display in hub
docker push app-created-in-hub:version
```
