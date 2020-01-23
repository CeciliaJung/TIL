# Docker Image

## Docker - Image & Container

도커는 이미지와 컨테이너로 구성되어 있다.

- 이미지: 도커 컨테이너를 구성하는 **파일 시스템**과 실행할 **애플리케이션 설정**을 하나로 합친것으로, 컨테이너를 생성하는 **템플릿 역할**

  

- 컨테이너: 도커 이미지를 기반으로 생성되며, 파일 시스템과 애플리케이션이 **구체화돼 실행되는 상태**

  ## Ex. Docker Application & Image 

  

``` cmd
C:\Users\HPE\Work\docker\day01>docker build -t example/echo:latest .
Sending build context to Docker daemon  3.584kB
Step 1/4 : FROM ubuntu:16.04
 ---> c6a43cd4801e
Step 2/4 : COPY helloworld /usr/local/bin
 ---> f90a7df48d22
Step 3/4 : RUN chmod +x /usr/local/bin/helloworld
 ---> Running in 04b9f430ddda
Removing intermediate container 04b9f430ddda
 ---> 7b2cc99c0904
Step 4/4 : CMD ["helloworld"]
 ---> Running in 05536255ec96
Removing intermediate container 05536255ec96
 ---> 68bf3f61593e
Successfully built 68bf3f61593e
Successfully tagged example/echo:latest
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
```



<Dockerfile>

``` visual basic
FROM ubuntu:16.04

COPY helloworld /usr/local/bin
RUN chmod +x /usr/local/bin/helloworld

CMD ["helloworld"]
```



### FROM 인스트럭션



### RUN 인스트럭션



### COPY 인스트럭션



## Build Docker Image





## Entrypoint 인스트럭션



## Execute Docker Container



### Port Forwarding



### docker image build 



#### -f 옵션



#### --pull 옵션





### docker search - 이미지 검색



### docker image pull - 이미지 내려받기



### docker image ls - 보유한 도커 이미지 목록 보기



### docker image tag - 이미지에 태그 붙이기



### docker image push - 이미지를 외부에 공개하기



###