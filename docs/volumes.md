# Volumes no Docker
## Criando um Volume

```bash
docker volume create myvol
```

Inspecionando o volume

```bash
docker volume inspect myvol
```
Ao inspecionar o volume notamos que o driver e o escopo são locais, significando que este volume só está disponível dentro deste dockerhost. 

Para deletar volumes há dois comandos possíveis: 
* docker volume prune --all - Deleta todos os volumes
* docker volume rm <volume>


Agora vamos criar um volume e montá-lo em um container

```bash
docker run -it --name voltainer \
    --mount source=bizvol,target=/vol \
    alpine

```
Ao sair do container e listar os volumes, é possível ver o bizbol listado. 
Se tentarmos deletar o volume receberemos um erro. 

Agora vamos escrever algo no volume

```bash

docker exec -it voltainer sh

# echo "teste" > /vol/file1

```

Agora vamos deletar o container e verificar se o volume permanece e vamos verificar se o conteúdo foi deletado. 

```bash
ls -l /var/lib/docker/volumes/bizvol/_data/
```
Vamos montar o volume em outro container

```bash

docker run -it \
  --name newctr \
  --mount source=bizvol,target=/vol \
  alpine sh

```
Quando temos um volume compartilhado por vários containers é importante saber como montá-los no formato read-only
```bash

docker container run -it --name writer \
    -v shared-data:/data \
    .alpine /bin/sh

docker container run -it --name reader \
    -v shared-data:/app/data:ro \
    ubuntu:22.04 /bin/bash
```

É possível montar subvolumes

```bash
docker volume create logs
 docker run --rm \
  --mount src=logs,dst=/logs \
  alpine mkdir -p /logs/app1 /logs/app2
 docker run -d \
  --name=app1 \
  --mount src=logs,dst=/var/log/app1/,volume-subpath=app1 \
  app1:latest
 docker run -d \
  --name=app2 \
  --mount src=logs,dst=/var/log/app2,volume-subpath=app2 \
  app2:latest

  ```


## Host Volumes

```bash
docker container run --rm -it \
    -v $(pwd)/src:/app/src \
    alpine:latest /bin/sh

```
## Docker compose

Utilizando volumes no docker compose

```bash

services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app
volumes:
  myapp:

```

É possível criar um volume através do ```docker volume create``` e depois montá-lo através do docker compose. 

```bash
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app
volumes:
  myapp:
    external: true
```

## Volumes no Swarm

```bash
docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```

O comando acima cria 4 replicas do nginx e cada uma delas utiliza um volume myvol2 local

A remoção do serviço não implica na remoção do volume. 






