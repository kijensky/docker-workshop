# docker-workshop
Tutorial that help you get hands-on experience using Docker &amp; Kubernetes

# Docker prakticky

Docker homepage: https://hub.docker.com/

Instalace: https://docs.docker.com/get-docker/

Pod Windows 10 doporučuji WSL2:
- https://docs.microsoft.com/en-us/windows/wsl/about
- https://docs.microsoft.com/en-us/windows/wsl/install-win10

Pískoviště:  https://labs.play-with-docker.com/

## Aplikace

Naše demonstrace vychází z: https://github.com/docker/getting-started

Stažení: 
```
git clone https://github.com/kijensky/docker-workshop
```

## Build

```
cd ./app
docker build -t todo .
docker history todo
```

## Size

```
docker ps -sa
```

## Spuštění

```
docker run –d -p 3000:3000 todo
```

## Aktualizace

```
docker build -t todo .
docker ps
docker ps -a
docker images
docker stop <container id>
docker start <container id>
docker rm -f <container id>
docker run –d -p 3000:3000 todo 
```

## Nejčastěji používané příkazy

```
docker –help
docker run <image>
docker exec -i –t <container> 
docker ps (-a, -s)
docker stop <container>
docker start <container>
docker rm <container>
docker rmi <image>
docker inspect <container
docker images
docker image –help
docker image ls
docker image prune
docker image inspect <image> 
docker image pull <image>
docker image push <image>
docker image rm <image>
docker image save
docker image load
```

## Registry a sdílení

```
docker tag <local-image-name> YOUR-USER-NAME/<shared-image-name>:<version>
docker login -u YOUR-USER-NAME
docker push YOUR-USER-NAME/<shared-image-name>:<version>

```

## Test

```
docker run –d -p 3000:3000 YOUR-USER-NAME/<shared-image-name>:<version> 
```

## Uložiště - Named Volumes

```
docker volume create todo-db
docker volume --help
docker volume inspect todo-db
docker run –d –p 3000:3000 –v todo-db:/etc/todos YOUR-USER-NAME/<shared-image-name>:<version>
```

Cesta k souborům na Windows 10: `\\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes`

## Bind mounts

```
docker run -dp 3000:3000 \
    -w /app -v "$(pwd):/app" \
    --name todo \
    node:12-alpine \
    sh -c "yarn install && yarn run dev"

docker logs –f <container name|container id>

```

## Multi-Container App

```
docker network create todo-app

docker run -d \
    --network todo --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7

docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo \
  --name todo \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

Ověření funkčnosti

```
docker exec -it <mysql container id> mysql –p
`
show database;
use todos;
select * from todo_items;
exit
`

docker run -it --network todo nicolaka/netshoot
dig mysql
cat /etc/hosts
```

# Orchestrace

## Docker-Compose

Instalace:  https://docs.docker.com/compose/install/

```
docker-compose version
```

[./samples/docker-compose.yml](./samples/docker-compose.yml)

```
docker-compose up –d
docker-compose logs -f
docker-compose stop
docker-compose start

docker exec -it <mysql container id> mysql –p
`
show database;
use todos;
select * from todo_items;
exit
`

docker-compose down
```

## Nejčastěji používané příkazy

```
docker-compose –help
docker-compose up (-d)
docker-compose down (--volumes)
docker-compose logs (-f)
docker-compose config
docker-compose build
docker-compose start
docker-compose stop
```

# Kubernetes

```
kubectl proxy
```

http://localhost:8001/api/v1


## K8s Demo

https://kubernetes.io 

https://www.katacoda.com/courses/kubernetes

https://labs.play-with-k8s.com/

https://kubernetes.io/docs/reference/kubectl/cheatsheet/
```
kubectl --help
kubectl version
kubectl cluster-info
kubectl get <resource>
kubectl describe
kubectl apply
kubectl run
kubectl inspect
kubectl delete
```

## Todo App v K8s

```
cd ./samples
kubectl apply -f deploy.yml
kubectl apply -f svc.yml

# kubectl apply -f nginx-ingress-controller.yml
kubectl apply -f ingress.yml

kubectl scale deploy/todo --replicas=2 
kubectl scale deploy/todo --replicas=1

```

http://kubernetes.docker.internal/


## Multi-Container App

```
cd ./samples
kubectl apply -f mysql-pv.yml
kubectl apply -f mysql.yml

kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
`
create database todos;
`
kubectl apply -f deploy-with-mysql.yml
```

## Todo App a security

```
echo -n "root" | base64
echo -n "password" | base64

kubectl apply -f secret.yml
kubectl apply -f deploy-with-mysql-secret.yml

```

## Uklid

```
kubectl delete deployment,svc mysql 
kubectl delete pvc mysql-pv-claim 
kubectl delete pv mysql-pv-volume
kubectl delete -f deploy-with-mysql-secret.yml
kubectl delete -f secret.yml
kubectl delete -f svc.yml
kubectl delete -f ingress.yml
```