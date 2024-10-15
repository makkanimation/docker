# Api 

```bash
## api_json we need to import outside python plugin, see in api folder Dockerfile
RUN pip3 install requests

docker build .
# check all imahges 
docker images 
```

# sql
```bash
ip a 
## find your ip address put itno docker 
## I got response 
##  wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
## inet 172.16.0.186/24
docker build .
docker run -it --rm e9f26d416841
```
 
 # Network between two container mysql and python
 ```bash
 #Create a netwrok for connection between two or more container 
 docker network --help
 docker network create --driver bridge my-net
 # check network 
 docker network ls
 # check network details 
 docker network inspect my-net
 # create mysql image container
 docker pull mysql # taken from https://hub.docker.com/_/mysql
 # Run mysql 
 docker run -d --env MYSQL_ROOT_PASSWORD="root" --env MYSQL_DATABASE="demodocker" --name mysqldb --network my-net mysql
 # docker build for python
 docker build . 
 # run container with network 
 docker run -it --rm --network my-net image_id
