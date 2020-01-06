docker -swarm

## 준비작업: swarm init

> docker swarm init

``` power
docker exec -it manager docker swarm init
docker exec -it worker01 docker swarm join
docker exec -it worker02 docker swarm join
docker exec -it worker03 docker swarm join
```

> check docker swarm is ready (1 leader(=manager) + 3 nodes)

``` powershell
docker exec -it manager docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
3io4h7thm2clip4catq5nqxq3     0ab1e048cc40        Ready               Active                                  19.03.5
fbv6sdnw1ldfre8a7ryw3ycp3     96dc32360132        Ready               Active                                  19.03.5
q0ruimsckgkk2e7dcu2tk6lmw     8630f9ce0554        Ready               Active                                  19.03.5
do83gjypx2hxxomzue7mwxzs9 *   e66c8d708afc        Ready               Active              Leader              19.03.5

```



## 01.도커 registry에 이미지 등록하기 (전제: swarm exists)

> 만약 pull 할 이미지가 아직 없다면 dockerfile 생성

1. @docker>day01>go에서 Dockerfile 생성

(해당 디렉토리에는 2개의 파일이 있다)

go/

 	-Dockerfile

​	 -main.go

``` visual basic
FROM golang:1.10
 
RUN mkdir /echo
COPY main.go /echo
CMD ["go", "run", "/echo/main.go"]
```

2. build image

   > docker build -t example/echo:latest .

   ``` visual basic
   PS C:\Users\HPE\work\docker\day01\go> docker build -t example/echo:latest .
   Sending build context to Docker daemon  4.096kB
   Step 1/4 : FROM golang:1.10
   1.10: Pulling from library/golang
   741437d97401: Pull complete
   34d8874714d7: Pull complete
   0a108aa26679: Pull complete
   :
   :
   Digest: sha256:6d5e79878a3e4f1b30b7aa4d24fb6ee6184e905a9b172fc72593935633be4c46
   Status: Downloaded newer image for golang:1.10
    ---> 6fd1f7edb6ab
   Step 2/4 : RUN mkdir /echo
    ---> Running in bd7fdbbaf914
   Removing intermediate container bd7fdbbaf914
    ---> 989071bb1b36
   Step 3/4 : COPY main.go /echo
    ---> 3b69aca703eb
   Step 4/4 : CMD ["go", "run", "/echo/main.go"]
    ---> Running in 1df2280341b5
   Removing intermediate container 1df2280341b5
    ---> bb08cb1c19a1
   Successfully built bb08cb1c19a1
   Successfully tagged example/echo:latest
   SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
   
   ```

   

3. (optional) run image

   > docker run -p 8080:8080 example/echo:latest

   ``` visual basic
   PS C:\Users\HPE\work\docker\day01\go> docker run -p 8080:8080 example/echo:latest
   2020/01/03 00:50:38 start server
   
   ```

   //check when typing '"localhost:8080"' @web browser

   

4. create tag for image

   >  도커 레지스트리용 이미지 생성

``` power
docker tag example/echo:latest localhost:5000/example/echo:latest
docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
example/echo                  latest              bb08cb1c19a1        9 minutes ago       760MB
localhost:5000/example/echo   latest              bb08cb1c19a1        9 minutes ago       760MB
```



5. push image //도커 레지스트리에 이미지 등록

   >  docker push localhost:5000/example/echo:latest

```  power
#shell01에서 일단 swarm server 기동
docker-compose up

#shell02에서 pull image
docker push localhost:5000/example/echo:latest
The push refers to repository [localhost:5000/example/echo]
10bfefe366b8: Pushed
b15b04f47be6: Pushed
7b9a9415bf3a: Pushed
facf15440126: Pushed
77b4b6493272: Pushed
6257fa9f9597: Pushed
578414b395b9: Pushed
abc3250a6c7f: Pushed
13d5529fd232: Pushed
latest: digest: sha256:bf6a796938ab72a5cd0c09aa5f8f7d747c410cf9b23298416db6ed9fa8c1c75d size: 2209
```

6. pull image // node 들이 manager 에서 image 를 가져온다

   >  #worker(01,02,03) 컨테이너에 이미지 설치
   >
   > docker exec -it worker01(/02/03) docker pull registry:5000/example/echo:latest

   ```power
   PS C:\Users\HPE> docker exec -it worker01 sh
   / # docker pull registry:5000/example/echo:latest
   latest: Pulling from example/echo
   741437d97401: Pull complete
   34d8874714d7: Pull complete
   0a108aa26679: Pull complete
   7f0334c36886: Pull complete
   d35724ed4672: Pull complete
   c0eaf021aeaf: Pull complete
   d3d9c96611f1: Pull complete
   45fec5c50d60: Pull complete
   baa046af8665: Pull complete
   Digest: sha256:bf6a796938ab72a5cd0c09aa5f8f7d747c410cf9b23298416db6ed9fa8c1c75d
   Status: Downloaded newer image for registry:5000/example/echo:latest
   registry:5000/example/echo:latest
   
   ```

   

**!주의!**

` localhost` 대신 ` registry` 를 입력하는 이유: 

-> 앞선 설정에서 우리가 `localhost port` 를 `worker01` , `worker02` `worker03` 에게 허용한다고 지정하지 않았기 떄문이다

->> 그래서 `node` 들은 모두 `registry` 로 인식한다



``` power
#worker01의 bash shell 에 접속후 pull image
docker exec -it worker01 sh
/ # docker pull registry:5000/example/echo:latest
latest: Pulling from example/echo
741437d97401: Pull complete
34d8874714d7: Pull complete
0a108aa26679: Pull complete
7f0334c36886: Pull complete
d35724ed4672: Pull complete
c0eaf021aeaf: Pull complete
d3d9c96611f1: Pull complete
45fec5c50d60: Pull complete
baa046af8665: Pull complete
Digest: sha256:bf6a796938ab72a5cd0c09aa5f8f7d747c410cf9b23298416db6ed9fa8c1c75d
Status: Downloaded newer image for registry:5000/example/echo:latest
registry:5000/example/echo:latest

#check
/ # docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
registry:5000/example/echo   latest              bb08cb1c19a1        20 minutes ago      760MB

```



###  >1 line command is also possible!!

한 줄에 모든 command 를 작성하는 것을`command chaining` 이라고 한다

``` powershell
PS C:\Users\HPE> docker exec -it worker03 docker pull registry:5000/example/echo:latest
latest: Pulling from example/echo
741437d97401: Pull complete
34d8874714d7: Pull complete
0a108aa26679: Pull complete
7f0334c36886: Pull complete
d35724ed4672: Pull complete
c0eaf021aeaf: Pull complete
d3d9c96611f1: Pull complete
45fec5c50d60: Pull complete
baa046af8665: Pull complete
Digest: sha256:bf6a796938ab72a5cd0c09aa5f8f7d747c410cf9b23298416db6ed9fa8c1c75d
Status: Downloaded newer image for registry:5000/example/echo:latest
registry:5000/example/echo:latest
PS C:\Users\HPE> docker exec -it worker03 docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
registry:5000/example/echo   latest              bb08cb1c19a1        36 minutes ago      760MB
```



## 02. Service 

- swarm 을 사용하는 경우, docker-compose를 사용하여 container를 실행하는 방법과 달리, 서비스(service) 를 사용하여 container들을 관리한다

2.1 manager 컨테이너에서 service 만들기

> #docker exec -it manger docker service create --replicas 1 --publish 8000:8080 --name echo\registry:5000/example/echo:latest

``` powersh
docker exec -it manager sh
/ # docker service create --replicas 1 --publish 8000:8080 --name echo registry:5000/example/echo:latest
7p6kpl397ji9rzve4ta7ty0w7
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged

/ # docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                               PORTS
7p6kpl397ji9        echo                replicated          1/1                 registry:5000/example/echo:latest   *:8000->8080/tcp
```



//1 container 에 6개의 service가 할당되는 것이다

>  #docker service scale echo=6

``` powee
/ # docker service scale echo=6
echo scaled to 6
overall progress: 6 out of 6 tasks
1/6: running   [==================================================>]
2/6: running   [==================================================>]
3/6: running   [==================================================>]
4/6: running   [==================================================>]
5/6: running   [==================================================>]
6/6: running   [==================================================>]
verify: Service converged
```



//check : 6services created //but replicated service only can use 1 same image -> 그래서 1개 이상의 이미지를 사용한다면 docker service 를 설정 및 기동해야한다

//아래와 같이 echo.1 의 `hostname` 과 manager의  `hostname` 과 같다는 것을 알 수 있다.

``` powe
/ # docker service ps echo
ID                  NAME                IMAGE                               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
ugditivypmq5        echo.1              registry:5000/example/echo:latest   e66c8d708afc        Running             Running 4 minutes ago

/ # hostname
e66c8d708afc

```



## 03. Stack

- 스택: 하나 이상의 서비스를 그룹을 묶은 단위
- 스택을 사용한 서비스 그룹은 `overlay network` 에 속한다!!
- 2개 이상의 이미지 사용 가능
- cf. 서비스는 애플리케이션 이미지를 하나밖에 못다룸



3.1 overlay network 를 생성한다(stack 의 전재조건!!!)

> /# docker network create --driver=overlay --attachable ch03

``` power
#manager 컨테이너로 접속해서 현재 network 리스트 확인
PS C:\Users\HPE> docker exec -it manager sh
/ # docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1d68ac246504        bridge              bridge              local
2d2b9d59495e        docker_gwbridge     bridge              local
4f76f38a7e94        host                host                local
vrulobya1cu4        ingress             overlay             swarm
c3ff5e6bfcd8        none                null                local

#overlay network인 ch03 생성
/ # docker network create --driver=overlay --attachable ch03
gg74fv7vzd44kglkjfu1ttybr

#check
/ # docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1d68ac246504        bridge              bridge              local
gg74fv7vzd44        ch03                overlay             swarm
2d2b9d59495e        docker_gwbridge     bridge              local
4f76f38a7e94        host                host                local
vrulobya1cu4        ingress             overlay             swarm
c3ff5e6bfcd8        none                null                local
```



3.2 overlay 네트워크를 사용하는 ch03-webapi.yml 파일을 생성

> // create ”ch03-webapi.yml”	
>
> // image : gihyodocker/nginx-proxy from hub.docker.com
>
> // 매니저는 뺴고 다른 곳에 놓는다 
>
> > placement: [node.role != manager]
>
> //backend_host -> 아래의 “api”에게 port 8080을 넘겨주기 위해 설정
>
> //echo_api -> “echo”라는 스택의 밑의 “api” service 를 뜻함
>
> > environment:      BACKEND_HOST: echo_api:8080

``` visual basic
version: "3"
services:
    nginx:
        image: gihyodocker/nginx-proxy
        deploy:
            replicas: 3
            placement: 
                constraints: [node.role != manager]
        environment:
            BACKEND_HOST: echo_api:8080
        depends_on:
            api
        networks:
            - ch03
 
    api:
        image: registry:5000/example/echo:latest
        deploy:
            replicas: 3
            placement:
                constraints: [node.role != manager]
        networks:
 
networks:
    ch03: 
    external: true
```



3.3 check 

>  // docker-compose file의 volume mount
>
> //현재 디렉토리(day03>swarm)의 하위 폴더인 stack 의 path manager의 root/stack 으로 mount
>
> > **volume: “./stack:/stack”**



3.4 stack 배포

> #docker stack deploy -c /stack/ch03-webapi.yml echo



``` power
/ # docker stack deploy -c /stack/ch03-webapi.yml echo

#show stack list
/ # docker stack ls
NAME                SERVICES            ORCHESTRATOR
echo                2                   Swarm

//해당 스택에 배포된 서비스 목록을 확인할 수 있음
#docker stack services echo
/ # docker stack  services echo
ID                  NAME                MODE                REPLICAS            IMAGE                               PORTS
cmnmyj64duz3        echo_api            replicated          3/3                 registry:5000/example/echo:latest
qis48kgdv7g2        echo_nginx          replicated          3/3                 gihyodocker/nginx-proxy:latest

#check

/ # docker stack ps echo
ID                  NAME                IMAGE                               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
odxnt2qzfm2t        echo_api.1          registry:5000/example/echo:latest   8630f9ce0554        Running             Running 14 minutes ago
dran9bxewzo5        echo_nginx.1        gihyodocker/nginx-proxy:latest      8630f9ce0554        Running             Running 14 minutes ago
6qvf6ais9rh7        echo_api.2          registry:5000/example/echo:latest   96dc32360132        Running             Running 14 minutes ago
s1uyknfbsaj9        echo_nginx.2        gihyodocker/nginx-proxy:latest      0ab1e048cc40        Running             Running 14 minutes ago
pj1ktzuyszc8        echo_api.3          registry:5000/example/echo:latest   0ab1e048cc40        Running             Running 14 minutes ago
17z404lrnadr        echo_nginx.3        gihyodocker/nginx-proxy:latest      96dc32360132        Running             Running 14 minutes ago
```



외와 같은 container 관계를 시각화한 자료는 아래와 같다

![stack constructure](https://github.com/CeciliaJung/TIL/blob/master/images/stack.png)



3.5 visualizer 를 이용해서 컨테이너 배치 시각화하기 //visualizer.yml을 작성

이미 스웤 클러스터에 컨테이너 그룹이 어떤 노드에 어떻게 배치됐는지 시각화해주는 visualizer라는 애플리케이션이 있다

> //visualizer 라는 이미지로 배포
>
> // image : dockersamples/visualizer from hub.docker.com

다음과 같은 visualizer.yml 파일을 작성한다

``` power
version: "3"
services:
    app:
        image: dockersamples/visualizer
        ports:
            - "9000:8080"
        volumes:
            - /var/run/docker.sock:var/run/docker.sock
        deploy:
            mode: global
            placement: 
                constraints: [node.role == manager]
```

**!주의!** 

- 항상 docker-compose up 명령으로 server를 기동시키고, 

  manager -bash shell로 접속하여 service / stack 을 배포하고 실행한다

- **docker-compsoe.yml** 의 위치는\docker\day03\swarm>

- visualizer.yml 의 위치는       \docker\day03\swarm>**stack**>



3.6 visualizer를 배포한다

> **# docker exec -it manager docker stack deploy -c /stack/visualizer.yml visualizer**

``` power
PS C:\Users\HPE\Work\docker\day03\swarm> docker exec -it manager sh
/ # docker stack deploy -c /stack/visualizer.yml visualizer
Creating network visualizer_default
Creating service visualizer_app
```



3.7 check

``` power
#현재 기동되는 services 체크
ID                  NAME                MODE                REPLICAS            IMAGE                               PORTS
cmnmyj64duz3        echo_api            replicated          3/3                 registry:5000/example/echo:latest
qis48kgdv7g2        echo_nginx          replicated          3/3                 gihyodocker/nginx-proxy:latest
axwawn3g7b03        visualizer_app      global              1/1                 dockersamples/visualizer:latest     *:9000->8080/tcp
//manager 9000 -> visualizer 8080

//manager 를 만들 떄 작성한 docker-compose.yml 파일
//windows 의 9000 -> manager 9000 -> 8080

```



web-browser에서  ` localhost:9000 ` 를 입력한다

![visualizer](https://github.com/CeciliaJung/TIL/blob/master/images/visualizer.png)



## 04. HAProxy //외부에서 서비스 사용

- proxy란? -> 회사 서버의 모든 외부 사이트로의 요청을 한 곳에서 관리

- visualizer 는 외부 호스트에서 접속 가능
  - host -> manager 사이는 port forwarding 설정

- HAProxy외부 호스트에서 요청되는 트래픽을 목적 서비스로 보내주는 프록시 서버 설정

  - dockercloud/hapoxy 이미지로 배포

    

- Dockerfile 있을 떄,이미지 배포 순서 
  - dockerfile -> run : 바로 이미지를 pull 받고, dockerfile로 설정을 지정

- Dockerfile 없을 떄,이미지 배포 순서 :
  - pull image -> dockerfile 작성 -> 이미지 배포(run)



4.1 ingress.yml , ch03-webapi.yml 작성

위치: day03>swarm>

- stackch03-ingress.yml 작성

- ch03-webapi.yml 수정

  

deploy 두가지 모두 배포

- ch03-webapi.yml  echo

- ch03-ingress.yml ingress



## 05.  Swarm 을 이용하여 TODOAPP 개발

![todoapp 아키텍처](https://github.com/CeciliaJung/TIL/blob/master/images/architecture-todoapp.png)



5.0 stack setting plans

//stack을 3개 만들고, 각 stack 의 이름을 다음과 같이 설정
-stack 01 - stack name : mysql

-stack 02 -stack name : app

-stack 03 -stack name : frontend



<master/slave 구조 구축> 

- docker hub의 mysql:5.7 이미지 1개를 생성

- master/slave ㅋ너테이너는 두 역활을 모두 수행할 수 있는 1개의 이미지로 생성
- mysql_master 현경 변수 -> master,slave 결정
- replicas 값을 조정해서 slave 갯수 결정



5.1 overlay network 구축

//to create 3 stacks above, we need **overlay network**

``` powe
PS C:\Users\HPE\Work\docker\day03\swarm> docker exec -it manager sh
/ # docker network create --driver=overlay --attachable todoapp
voeu1v4feti49472d1csy0xct
/ # docker network ls
NETWORK ID          NAME                 DRIVER              SCOPE
voeu1v4feti4        todoapp              overlay             swarm
fss3yeaxxxv1        visualizer_default   overlay             swarm
```



5.2 conf 파일 작성



5.3 build image

> #docker build -t ch04/tododb:latest .

``` power
PS C:\Users\HPE\Work\docker\day03\swarm\todo\tododb> docker build -t ch04/tododb:latest .
Sending build context to Docker daemon   29.7kB
Step 1/16 : FROM mysql:5.7
 ---> db39680b63ac
:
:
Successfully built 9786ca999710
Successfully tagged ch04/tododb:latest

#check
PS C:\Users\HPE\Work\docker\day03\swarm\todo\tododb> docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED              SIZE
ch04/tododb                   latest              9786ca999710        About a minute ago   479MB
example/echo                  latest              bb08cb1c19a1        7 hours ago          760MB

```



5.4 tag image

> #**docker tag ch04/tododb:latest localhost:5000/ch04/tododb:latest**

``` power
PS C:\Users\HPE\Work\docker\day03\swarm\todo\tododb>docker tag ch04/tododb:latest localhost:5000/ch04/tododb:latest
PS C:\Users\HPE\Work\docker\day03\swarm\todo\tododb>docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
ch04/tododb                   latest              9786ca999710        3 minutes ago       479MB
localhost:5000/ch04/tododb    latest              9786ca999710        3 minutes ago       479MB

```



5.5 push image 

> #docker push localhost:5000/ch04/tododb:latest

``` power
PS C:\Users\HPE\Work\docker\day03\swarm\todo\tododb>docker push localhost:5000/ch04/tododb:latest
The push refers to repository [localhost:5000/ch04/tododb]
```



//worker01,02,03 에서 pull images

>  #docker exec -it worker01 sh
>
> / # docker pull registry:5000/ch04/tododb:latest

``` power
PS C:\Users\HPE\work\docker\day03\swarm\todo\tododb> docker exec -it worker01 sh
/ # docker pull registry:5000/ch04/tododb:latest
latest: Pulling from ch04/tododb
```



5.6 stack deploy

> docker exec -it manager docker stack deploy -c /stack/todo-mysql.yml todo_mysql



5.7 check

> docker exec -it manger docker stack service ls 



**! 주의!**

run image 할 떄 오류가 나는 이유:

>  linux 환경이 아닌 windows에서 encoding 하다가 오류 발생



해결책>

> 모든 .sh 파일을 ‘’LF ‘’ 로 바꾸기!

