# Docker Container

## Docker Container Lifecycle

- 실행 중 상태

- 정지 상태

- 파기 상태

  > docker container run // 컨테이너 생성 및 실행

``` git
docker (container) run [options] +<이미지명>[:태그] [명령] [명령인자...]
```



> -i 옵션 

``` git
docker (container) 
```



> --rm 옵션

``` git
docker
```



> -v 옵션

``` git
docker
```



> docker container ls // 도커 컨테이너 목록보기

**container ID** 가 가장 중요!



> --filer 옵션 // 컨테이너 목록 필터링하기

``` git
docker (container) ls --filter "필터명=값"
```



> -a 옵션 // 종료된 컨테이너 목록 보기

 

> stop 명령 // 컨테이너 정지하기

``` git
docker (container) stop 컨테이너ID_또는_컨테이너명
```



> restart 명령 // 컨테이너 재시작하기



> rm 명령 // 컨테이너 파기하기



> run --rm 결합 명령 // 동시에 정지 및 삭제



> logs 명령  // 표준 출력 연결하기



> exec 명령 // 실행 중인 컨테이너에서 명령 실행하기



### prune 명령 // 컨테이너 및 이미지 파기

``` git
docker (container/image) prune [option]
```



> stats 명령 // 사용 현황 확인



## 도커 컨테이너 = 단일 애플리케이션

도커는 애플리케이션 배포에 특화된 컨테이너이다.

**도커 컨테이너 = 단일 애플리케이션**

도커 컨테이너로 시스템을 구축하면 하나 이상의 컨테이너가 서로 통신하며, 

그 사이에 **의존관계** (dependency)가 생긴다.