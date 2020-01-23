# APP_Stack 

# 준비작업

1. #docker-compose up : R, M .W1.W2.W3 생성
2. (M 접속 후) #docker swarm init
3. (W1/W2/W3 접속 후) #docker swam join
4. (M 접속 후) #docker node ls : 모든 컨테이너의 hostname , id 가 보인다 (1 m, 3 w)
   //visualizer yml 파일 설치 
5. (M 접속 후) #docker stack deploy -c /stack/visualizer.yml visualizer 
6. (M 접속 후) #docker service ls
   //check 어느 노드에 visualizer 가 설치 되었는지
7. (M 접속 후) #docker service ps visualizer_app
8. (M 접속 후)//반드시 Dockerfile 있는 곳에서 visualizer 설치   //visualizer.yml 파일 작성 후 배포)
9. mysql 이미지 생성 후 push (registry)  //image tag 설정은 이번에는 스킵!		
   1. #docker build -t localhost:5000/ch04/tododb:latest .		
   2. #docker push localhost:5000/ch04/tododb:latest		
10. todo-mysql stack 추가 
    1. #ls -al /stack
    2. #docker stack deploy -c /stack/todo-mysql.yml todo_mysql
    3. #docker service ls

# 01.중첩 명령어 입력

2020.01.03 금요일에 이미 Mysql Stack 구성이 끝났다

컨테이너에 초기 데이터를 넣기 위해 

마스터 컨테이너가 스웜 `node` 중에 어느 것에 배치됐는지 확인해야한다

-> 따라서 manager/worker01/02/03 으로 첫번째로 접속, 두번쨰로 각 노드에 속한 컨테이너 (master/slave) 에 접속한다



(비교)단일 명령 2번으로 컨테이너에 접속 -총 4개 명령어 입력

``` powershe
//01.service 목록을 조회
#docker exec -it manager sh
/#docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
w1849n2iyoaa        todo_mysql_master   replicated          1/1                 registry:5000/ch04/tododb:latest
aew79p6iwpru        todo_mysql_slave    replicated          2/2                 registry:5000/ch04/tododb:latest

//02.master 의 id로 해당 service 조회 
// docker service ps <master/slave01/02의 node>
/ # docker service ps <w1849n2iyoaa>
ID                  NAME                      IMAGE                              NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
myl4qv811w74        todo_mysql_master.1       registry:5000/ch04/tododb:latest   c933edae1204        Running             Running 25 minutes ago

//03.windows 로 나와서 master(or slave)의 node로 다시 접속
//docker exec -it <master/slave의 node>
#docker exec -it <c933edae1204> sh
/ # docker ps
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS                 NAMES
a7a1eef1e6a9        registry:5000/ch04/tododb:latest   "prehook add-server-…"   27 minutes ago      Up 27 minutes       3306/tcp, 33060/tcp   

//04.master/slave container 로 접속
//docker exec -it <container id> bash
/ # docker exec -it a7a1eef1e6a9 bash
root@a7a1eef1e6a9:/#

```



중첩 명령어 입력

- 노드의 id와 태스크의 id만 알면 다음과 같이 docker exec 중첩 명령 가능하다

- 앞서 4번의 입력에 걸친 접속을 1번의 표준 출력 명령으로 접속할 수 있다

``` power
//windows cmd 창에서 바로 아래와 같은 표준 데이터 출력 명령을 입력
//단 아래와 같이 service(master/slave)의 이름을 알아야한다
#docker exec -it manager docker service ps todo_mysql_master \
--no-trunc --filter "desired-state=running" \
--format "docker exec -it {{.Node}} \
docker exec -it {{.Name}}.{{.ID}} bash"

docker exec -it c933edae1204 docker exec -it todo_mysql_master.1.myl4qv811w74n07w1r9tb68dz bash

```



# 02.초기 데이터 mysql에 입력

1. manager- master 컨테이너에 접속

   ``` power
   #docker exec -it <manager-node-id> \
   docker exec -it <todo_mysql_master.1.task-id>\
   init-data.sh
   ```

   

2. init-data.sh 스크립트를 실행-> 초기 데이터 생성

   ``` power
   #docker exec -it <manager-node-id> \
   docker exec -it <todo_mysql_master.1.task-id>\
   mysql -u gihyo -p gihyo tododb
   ```

   

3. mysql 접속 후 초기 데이터 확인

   ``` power
   mysql>select * from todo LIMIT 1\G
   ```



4. slave 에서도 데이터가 조회되는지 확인

   ``` power
   #docker exec -it <manager-node-id> \
   docker exec -it <todo_mysql_slave.1(or2).task-id>\
   mysql -u gihyo -p gihyo tododb
   
   mysql>select * from todo LIMIT 1\G
   ```



# 03.TODO API 구축 -todoapi

1. todoapi 폴더에 있는 dockerfile을 빌드해서 이미지를 생성

   ``` power
   //빌드할 이미지가 있는 폴더 (=dockerfile 위치)에서 이미지 생성
   #(todoapi)docker build -t ch04/todoapi:latest .
   ```

   

2. 이미지 태깅

   ``` power
   //이미지  태그
   #docker tag ch04/todoapi:latest localhost:5000/ch04/todoapi:latest
   ```

   

3. push 해서 이미지를 registry에 등록

   ``` power
   $#docker push localhost:5000/ch04/todoapi:latest
   ```

   

4. todoapi stack 배포

   ``` power
   #docker exec -it manager docker stack deploy -c /stack/todo-app.yml todo_app
   
   Creating service todo_app_api
   ```

   

# 04.Nginx 구축 -nginx 

1. nginx 폴더에 있는 dockerfile을 빌드해서 이미지를 생성

   ``` power
   //빌드할 이미지가 있는 폴더 (=dockerfile 위치)에서 이미지 생성
   #(nginx)docker build -t ch04/nginx:latest .
   ```

   

   ``` power
   //이미지  태그
   #docker tag ch04/nginx:latest localhost:5000/ch04/nginx:latest
   ```

   

2. push 해서 이미지를 registry에 등록

   ``` power
   #docker push localhost:5000/ch04/nginx:latest
   ```

   

4. nginx stack 배포

   //nginx stack 을 배포하면서 todo_app 의 스택을 다음과 같이 수정(updating & creating)

   ``` power
   #docker exec -it manager docker stack deploy -c /stack/todo-app.yml todo_app
   Updating service todo_app_api (id:kzsd0omdt7feb0aobkxb91jdb)
   Creating service todo_app_api
   ```

   

아래와 같은 그림을 참고하세요

현재 network 중 app stack, mysql stack 구축 완료인 상태이다

![todo_app 아키텍처](https://github.com/CeciliaJung/TIL/blob/master/images/architecture-todoapp.png)