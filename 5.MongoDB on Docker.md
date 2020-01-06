# MongoDB on Docker

## 데이터 볼륨 컨테이너

데이터 볼륨 컨테이너는 이름 그대로 데이터를 저장하는 것만이 목적인 컨테이너이다

도커 컨테이너를 파기(rm) 하지 않는 이상한 컨테이너의 내용은 디스크에 그대로 유지된다.



## ex. MongoDB 설치 on Docker

1. 이미지 받아오기

   //만약 태그명을 명시하지 않으면 자동으로 ` :latest` 가 다운된다

   ``` git
   HPE@DESKTOP-DFE1UPJ MINGW64 ~/Work/docker/day03
   $ docker pull mongo
   Using default tag: latest
   docklatest: Pulling from library/mongo
   2746a4a261c9: Pulling fs layer
   4c1d20cdee96: Pulling fs layer
   :
   :
   bca9e535ddb8: Verifying Checksum
   bca9e535ddb8: Download complete
   bca9e535ddb8: Pull complete
   9c3edad81b2a: Pull complete
   6dbcf78fe5ae: Pull complete
   Digest: sha256:7a1406bfc05547b33a3b7b112eda6346f42ea93ee06b74d30c4c47dfeca0d5f2
   Status: Downloaded newer image for mongo:latest
   docker.io/library/mongo:latest
   
   ```

   

2. 기동

   //container 기동완료 후, 반드시 `$docker ps` 로 container 가 기동되었는지 확인한다

   ``` git
   HPE@DESKTOP-DFE1UPJ MINGW64 ~/Work/docker/day03
   $ docker run --name mongodb_server -d -p 16010:27017 mongo
   f8c796ec677510aaee28f296b1a791bdc3947ab41e9c0546297f423c010b9cad
   
   HPE@DESKTOP-DFE1UPJ MINGW64 ~/Work/docker/day03
   $ docker ps
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
   00f91befe37f        mongo               "docker-entrypoint.s…"   3 seconds ago       Up 1 second         0.0.0.0:16010->27017/tcp   mongodb_server
   012b893f3dab        mysql:5.7           "docker-entrypoint.s…"   40 hours ago        Up 40 hours         3306/tcp, 33060/tcp        mysql
   ```

   

3. bash 접근, mongo 접속

   //mongodb_server : 우리가 지정한 mongo 베이스 이미지를 사용하는 컨테이너 이름

   ``` git
   docer exec -it mongodb_server bash
   ```

   

4. 관리자 계정 생성

   

5. 관리자 로그인, 일반 계정 생성

6. ReplicaSet 설정

   6.1 container 의 ip address 체크 // docker inspect + <containerID>

   ``` git
   #windows 에서는 작동하지 않을 수 있다
   #그럴땐 아래의 'docker inspect + <containerID> 활용
   ifconfig
   ```
   
   
   
   ``` GIT
   //1번쨰 container ip address 확인
   HPE@DESKTOP-DFE1UPJ MINGW64 ~/Work/docker/day03
   $ docker ps
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
   65f914741465        mongo               "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       0.0.0.0:36010->27017/tcp   mongodb_server_3
   0d0370b882a7        mongo               "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       0.0.0.0:26010->27017/tcp   mongodb_server_2
   a243945642b5        mongo               "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       0.0.0.0:16010->27017/tcp   mongodb_server_1
   012b893f3dab        mysql:5.7           "docker-entrypoint.s…"   41 hours ago        Up 41 hours         3306/tcp, 33060/tcp        mysql
   
   HPE@DESKTOP-DFE1UPJ MINGW64 ~/Work/docker/day03
   $ docker inspect a243945642b5
   :
   :
                      {"Gateway": "172.17.0.1",
                       "IPAddress": "172.17.0.3",
                       "IPPrefixLen": 16,
                       "IPv6Gateway": "",
                       "GlobalIPv6Address": "",
                       "GlobalIPv6PrefixLen": 0,
                       "MacAddress": "02:42:ac:11:00:03",
                       "DriverOpts": null}
   ```
   
   

 

``` git
//2번쨰 container ip address 확인
HPE@DESKTOP-DFE1UPJ MINGW64 ~/Work/docker/day03
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
65f914741465        mongo               "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       0.0.0.0:36010->27017/tcp   mongodb_server_3
0d0370b882a7        mongo               "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       0.0.0.0:26010->27017/tcp   mongodb_server_2
a243945642b5        mongo               "docker-entrypoint.s…"   11 minutes ago      Up 11 minutes       0.0.0.0:16010->27017/tcp   mongodb_server_1
012b893f3dab        mysql:5.7           "docker-entrypoint.s…"   41 hours ago        Up 41 hours         3306/tcp, 33060/tcp        mysql

HPE@DESKTOP-DFE1UPJ MINGW64 ~/Work/docker/day03
$ docker inspect 0d0370b882a7
:
:
                   {"Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.4",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null}
```

``` git
//3번쨰 container ip address 확인
$ docker inspect 65f914741465
:
:
                   {"Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.5",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null}
```

### Container name 과 IP address

1번째 container (=mongodb_server_1) -> 172.17.0.3

2번째 container (=mongodb_server_2) -> 172.17.0.4

3번째 container (=mongodb_server_3) -> 172.17.0.5

 6.2 Primary container 로 접속하여 rs.add



``` mongodb
myapp:PRIMARY> rs.initiate();
myapp:PRIMARY> rs.add("172.17.0.4:27017")

```

