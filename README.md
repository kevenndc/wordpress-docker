# Stack Wordpress com monitoramento Visualizer

Este repositório destina-se a implementação de uma stack Wordpress rodando em containers no Docker no modo Swarm. Para criar o cluster, foram utilizadas máquinas EC2 da AWS.

# Pré-requisitos

- [Docker] instalado e configurado na máquina;
- Porta **81** da rede liberada para o Wordpress
- Porta **3306** da rede liberada para o banco de dados MYSQL
- Porta **8888** da rede liberada para o monitoramento com o Visualizer
- Na AWS, é necessário que todas as máquinas tenham livre acesso de uma a outra. Recomendo que todas estejam no mesmo grupo de segurança com livre acesso para todas as máquinas do grupo.


# Introdução
Uma stack Wordpress no docker é composta essencialmente pelo uso das imagens do Wordpress e do banco de Dados MYSQL, porém, além disso, foi utilizado o Visualizer como serviço para monitorar os nodes e serviços executados no cluster Docker. 

É possível declarar todos os serviços e suas respectivas configurações em um arquivo **.yml** ou **yaml**. A da declarações dos serviços e suas configurações, pode-se utilizar o comando `` docker stack deploy`` para subir todos os serviços declarados em containers em todos os nodes do cluster. 

Este projeto foi feito para ser utilizado em um cluster de no máximo 3 máquinas. É possível aumentar a capacidade máxima de réplicas de cada serviço alterando a configuração `` replica`` de cada derviço. Abaixo é mostrado onde fazer essas alterações no arquivo [docker-compose.yaml]

```
version: "3.2"

services:
  
  wordpress:

    image: wordpress:latest
    deploy:
      replicas: 3 //trocar a quantidade de replicas máxima de wordpress aqui
    restart: always
    ports:
      - 81:80
    volumes:
      - wordpress-data:/var/www/public_html
    depends_on: 
      - mysql
    env_file:
      - wordpress.config
    links:
      - mysql

  mysql:

    image: mysql:latest
    deploy:
      replicas: 3 //trocar a quantidade de replicas máxima de MYSQL aqui
    restart: always
    ports:
      - 3306:3306
    volumes:
      - mysql-data:/var/lib/mysql
    env_file:
      - db.config
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - 8888:8080
    deploy:
      placement:
        constraints: [node.role == manager] //faz com que o serviço do  Visualizer rode somente no nó manager do cluster
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    networks:
      - webserver

volumes:

  wordpress-data:
  mysql-data:

networks:

  webserver:
````

O repositório é composto de 3 arquivos:
* [docker-compose.yaml] - contém a declaração dos serviços utilizados na stack
* [wordpress.config] - contém a configuração de ambiente do Wordpress 
* [db.config] - contém a configuração de ambiente do banco de dados MYSQL 


# Como implantar os serviços

1 - Para implantar o serviço, baixo os arquivos deste repositório para sua a máquina com o nó gerente do cluster utilizando o comando:
```sh
$ git clone https://github.com/kevenndc/wordpress-docker.git
```
2 - Acesse a pasta do repositório:
```sh
$ cd wordpress-docker
```

3 - Execute o comando a seguir para subir os serviços para todos os nós do cluster:
```sh
$ docker stack deploy -c docker-compose.yaml wp
```

4 - É possível verificar o estado de cada serviço utilizando o comando:
```sh
$ docker service ls
```

5 - No navegador, acesse ```http://<ip-publico-de-alguma-maquina-do-cluster>:81``` para acessar o Wordpress
6 - No navegador, acesse ```http://<ip-publico-de-alguma-maquina-do-cluster>:8888``` para acessar o Visualizer

[//]:


   [docker-compose.yaml]: <https://github.com/kevenndc/wordpress-docker/blob/master/docker-compose.yaml>
   [db.config]: <https://github.com/kevenndc/wordpress-docker/blob/master/db.config>
   [wordpress.config]: <https://github.com/kevenndc/wordpress-docker/blob/master/wordpress.config>
   [Docker]: <https://docs.docker.com/get-docker/>

