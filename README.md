# postgresql on docker
## basic examples

> First clone this repo  and check docker engine is ready

~~~
git clone ddd
cd demo
cd test1
ogarciacaro:test1$ docker version
Client:
 Version:      17.06.0-ce
 API version:  1.30
 Go version:   go1.8.3
 Git commit:   02c1d87
 Built:        Fri Jun 23 21:23:31 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.06.0-ce
 API version:  1.30 (minimum version 1.12)
 Go version:   go1.8.3
 Git commit:   02c1d87
 Built:        Fri Jun 23 21:19:04 2017
 OS/Arch:      linux/amd64
 Experimental: true

~~~

### run postgresql10 container
~~~
cd 1basic

clear && docker rm -f mypostgresql && docker volume rm pgdata


#BUILD CUSTOM POSTGRESQL10 IMAGE
docker build -t mypg10 ./postgresql10

#CREATE VOLUME AN RUN CONTAINER
docker volume create pgdata
docker run -d -p 5432:5432 \
  --name mypostgresql  \
  -v pgdata:/var/lib/postgresql/data  \
  mypg10

# CHECK CONTAINER STATE
docker ps

# CONNECT VIA CONTAINER-PSQL
docker exec -it mypostgresql psql -U postgres

# CONNECT VIA PSQL
psql -h localhost -p 5432 -U postgres
Password for user postgres:

postgres=# \c mydb
You are now connected to database "mydb" as user "postgres".
mydb=# \d mytable
              Table "public.mytable"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 col1   | integer |           |          |

~~~

### run postgresql10 as stack
> before run enuser that you have swarm "docker swarm init"

~~~
cd 2stack

clear && docker stack rm test2 && docker volume rm pgdata

# CREATE VOLUME
docker volume create pgdata

# DEPLOY STACK
docker stack deploy -c docker-compose.yml test2

# CHECK STACK
docker stack ls && docker service ls && docker ps && docker volume ls

# CONNECT VIA PSQL
psql -h localhost -p 5432 -U postgres
Password for user postgres:

postgres=# \c mydb
You are now connected to database "mydb" as user "postgres".
mydb=# \d mytable
              Table "public.mytable"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 col1   | integer |           |          |

~~~


### run postgresql10 with usefuls tools

~~~
cd 3withTools

clear && docker stack rm 3withtools && sleep 5 &&docker volume rm pgdata

# CREATE PGHERO IMAGE
docker build -t pghero ./pghero/

# CREATE VOLUME
docker volume create pgdata

# DEPLOY STACK
docker stack deploy -c docker-compose.yml 3withtools

# CHECK STACK
docker stack ls && docker service ls && docker ps && docker volume ls

# CHECK PGADMIN & PGHERO
curl http://127.0.0.1:5050/ (pgadmin4@pgadmin.org/admin)
curl http://127.0.0.1:8080

~~~

### run postgresql10 with streaming replication

~~~
cd 4hotstandby

clear
docker stack ls && docker service ls && docker ps -a && docker images
docker stack rm 4hotstandby
docker rm -f $(docker ps -a -q)
docker build -t backend-a ./backend-a
docker build -t backend-b ./backend-b
docker stack deploy -c docker-compose.yml 4hotstandby
docker stack ls && docker service ls && docker ps -a && docker images

psql -d "postgresql://localhost:5432,localhost:5433/?target_session_attrs=read-write" -U postgres -c 'select inet_server_addr();'
~~~

### run postgresql10 with pgpool

~~~
cd 5pgpool

docker stack rm 2stack 3withtools 4hotstandby 5pgpool
docker rm -f $(docker ps -a -q)

docker build -t mypgpool ./mypgpool
docker build -t backend-a ./backend-a && docker build -t backend-b ./backend-b
docker stack deploy -c docker-compose.yml 5pgpool

psql -h localhost -p 9999 -U postgres -c "create table mitabla(col integer);"
psql -h localhost -p 9999 -U postgres -c "insert into mitabla values(1);"
psql -h localhost -p 9999 -U postgres -c "select count(col),inet_server_addr() from mitabla;"

~~~

### run postgresql10 with BDR

~~~
cd 6bdr

docker stack deploy -c docker-compose.yml 6bdr


psql -h localhost -p 5432 -U admin mydb -c "create table mitabla(col integer);"
clear
psql -h localhost -p 5433 -U admin mydb -c "insert into mitabla values(1);"
psql -h localhost -p 5432 -U admin mydb -c "select count(col),inet_server_addr() from mitabla;"
psql -h localhost -p 5433 -U admin mydb -c "select count(col),inet_server_addr() from mitabla;"
psql -h localhost -p 5434 -U admin mydb -c "select count(col),inet_server_addr() from mitabla;"

~~~
