docker -v

# start docker service
systemctl start docker.service
# status of docker runing or not
systemctl status docker.service
# see docker container
docker ps
# permission denied while trying to connect to the Docker daemon socket at (Error)
![image](https://github.com/user-attachments/assets/647154f1-1203-483d-ba9a-88f41a2f6ad1)

# for creating docker file go to dockerfile.md

```bash
# Afrer creating dockerfile (tutorial mengtion under dockerfile.md), start docker
docker run imageid
# to stop docker
docker ps
docker stop NAMES # showing under docker ps command
```
