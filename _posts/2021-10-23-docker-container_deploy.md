---
layout: post
title: 도커 컨테이너 배포
subtitle: 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문
categories: docker
tags: [DOCKER, CONTAINER, INFRASTRUCTURE]
---

Chapter02 도커 컨테이너 배포
---
### 01 컨테이너로 애플리케이션 실행하기
1. 도커 이미지 vs 도커 컨테이너  
- 도커 이미지 = 도커 컨테이너를 구성하는 파일 시스템 설정 + 실행할 애플리케이션 설정  
- 도커 컨테이너 = 도커 이미지가 구체화돼 실행되는 상태

![docker_image_vs_container.png](/assets/images/docker/ch02/docker_image_vs_container.png)

- 도커 이미지 다운로드 
```docker
docker image pull gihyodocker/echo:latest
```
![docker_image_pull.png](/assets/images/docker/ch02/docker_image_pull.png)  
    * 처음 받으면 모든 레이어 이미지를 받는 것을 볼수 있다.

#### 도커 명령어 모음

| 도커 명령어 | 용도 | remark |
| :-------------- | :-------------- | :-------------- |
| docker image pull | 도커 허브에서 이미지 다운로드 | =docker pull|
| docker image build | 도커 이미지 빌드 | =docker build, -t 태그 |
| docker image ls | 도커 이미지 리스트 | -q : 컨테이너 아이디만 조회 가능 |
| docker container run | 도커 컨테이너 실행  | = docker run -p : 포트 |
| ^^ | ^^ | -d=true or -d : 컨테이너 실행시 로그 올라오는데 안봐도 되면 백그라운드 모드로 실행|
| ^^ | ^^ | -t : Allocate a pseudo-tty (foreground mode에서 컨테이너에 터미널로 접속할 수 있게 해준다. [-t explanation])|
| docker container ls | 도커 컨테이너 리스트 조회 | "docker ps" 와 같고, "docker ps -a"인 경우 실행중이지 않고, 삭제되지 않은 컨테이너를 볼 수 있다. |
| docker container stop [컨테이너 아이디 혹은 이름] | 도커 컨테이너 정지  |(docker stop)|
| docker rm [컨테이너 아이디 혹은 이름] | 도커 컨테이너 삭제  ||

[-t explanation]: https://stackoverflow.com/questions/30137135/confused-about-docker-t-option-to-allocate-a-pseudo-tty

#### 간단한 애플리케이션과 도커 이미지 만들기
**순서**
![docker_use_seq.png](/assets/images/docker/ch02/docker_use_seq.png)

1. 애플리케이션을 만든다. -> main.go

```text
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		log.Println("received request")
		fmt.Fprintf(w, "Hello Docker!!")
	})

	log.Println("start server")
	server := &http.Server{Addr: ":8080"}
	if err := server.ListenAndServe(); err != nil {
		log.Println(err)
	}
}
```

2. 애플리케이션을 기반으로 도커 이미지를 만든다. -> Dockerfile  

```dockerfile
FROM golang:1.9

RUN mkdir /echo
COPY main.go /echo

CMD ["go", "run", "/echo/main.go"]
```  

 * 도커파일 인스트럭션  
  
| 인스트럭션(명령) | 용도 | remark |
| :-------------- | :-------------- | :-------------- |
| FROM | 도커 이미지의 바탕이 될 베이스 | 도커 허브에서 받아오며 ':' 뒤는 태그라고 하며 이미지 버전을 구별하는 식별자 |
||| 도커 이미지는 각자 고유의 해시값을 갖는데, 태그가 없으면 해시만으로 구별해야 하여 쉽게 파악이 안되니 태그를 붙이는것이 좋다.|
| RUN | 도커 이미지를 실행할 때 컨테이너 안에서 실행할 명령을 정의 | 예제에서는 애플리케이션을 배치하기 위한 /echo 디렉토리를 만든다. |
||| 이미지를 빌드할 때 실행된다. |
| COPY | 도커가 동작중인 호스트 머신의 파일/디렉토리 -> 도커 컨테이너 안으로 복사 | 비슷한 기능으로 ADD가 있다.|
| CMD | 도커 컨테이너를 실행할 때 안에서 실행할 프로세스 | 명령을 공백으로 나눈 배열로 구성되며, 컨테이너를 시작할 때 1회 실행된다. |
||| CMD["실행파일", "인자1", "인자2"] -> 실행파일에 인자를 전달. 권장 사용방식 |
||| CMD "명령인자" "인자1" "인자2" -> 명령과 인자 지정으로 셸에서 실행되므로 셸에 정의된 변수를 참조할 수 있다. |
||| CMD ["인자1", "인자2"] -> ENTRYPOINT에 지정된 명령어에 사용할 인자를 전달한다. |
| LABEL | 이미지를 만든 사람의 이름 등을 적을 수 있다. | |
| ENV | 컨테이너 안에서 활용할 수 있는 환경변수 지정 | |
| ARG | 이미지 빌드 시 정보를 함께 넣기 위한 용도로 빌드할때만 사용된다. | |

* LABEL, ENV, ARG 포함 dockerfile 예시

```dockerfile
FROM alpine:3.7
LABEL maintainer="user@example.com"

ARG builddate
ENV BUILDDATE=${builddate}

ENV BUILDFROM="from Alpine"

ENTRYPOINT ["bin/bash", "-c"]

CMD ["env"]
```

```docker
  docker image build --build-arg builddate=today -t example/others
```

3. 도커 이미지를 빌드 -> (docker image build -t [이미지명(:태그명)] [도커파일경로] )  

```docker
  docker image build -t example/echo .
```

이미지명의 '/' 앞은 네임스페이스로 이미지명이 겹치는 것을 방지하기 위함이다. 빌드된 이미지는 다음 명령어로 조회할 수 있다.

```docker
docker image ls
```

![docker_image_build_1.png](/assets/images/docker/ch02/docker_image_build_1.png)

![docker_image_build_2.png](/assets/images/docker/ch02/docker_image_build_2.png)

4. 도커 이미지를 기반으로 도커 컨테이너를 실행한다. -> (docker container run [이미지명:태그])

```docker
    docker container run example/echo:latest
```
![docker_container_run_foreground.png](/assets/images/docker/ch02/docker_container_run_foreground.png)

```docker
    docker container run -d example/echo:latest
```
![docker_container_run_background.png](/assets/images/docker/ch02/docker_container_run_background.png)
    
   * -d 옵션을 주고 실행하였을시 나오는 해시값처럼 생긴 문자열은 도커 컨테이너의 ID로 유일 식별자이다. 

#### 포트포워딩 
main.go 코드를 보면 8080을 리스닝하고 있지만 ```curl http://localhost:8080``` 으로 요청을 보내면 Connection refused 메세지를 받는다. 
- 도커 컨테이너는 가상 환경이지만 외부에서 봤을 때 독립된 하나의 머신처럼 다룰 수 있다. 
- 애플리케이션은 8080을 리스닝하지만, 이 포트는 컨테이너 포트라고 하여 컨테이너에 한정된 포트이다. 따라서, 컨테이너 안에서 curl로 8080 접근하면 가능하지만 host os에서는 접근이 안된다.
- 도커 포워딩을 사용하여 컨테이너 밖에서 온 요청을 컨테이너 안에 있는 애플리케이션에 전달해야한다.  
    1. 현재 실행중인 컨테이너 정지
        ```docker
           docker container stop $(docker container ls --filter "ancestor=example/echo" -q)
        ```
    2. 포트포워딩을 포함하여 실행한다. (-p 호스트포트:컨테이너포트)
        ```docker
           docker container run -d -p 9000:8080 example/echo:latest
        ```
    3. 호스트포트로 요청한다. (curl http://localhost:9000)
        ![curl_host_port.png](/assets/images/docker/ch02/curl_host_port.png)
    
- 호스트 포트를 생략하고 포트포워딩을 사용하면 도커가 자동으로 할당함으로 부여받은 포트로 접근하면 된다. 
        ![ephemeral_port.png](/assets/images/docker/ch02/ephemeral_port.png)

![port_forwarding.png](/assets/images/docker/ch02/port_forwarding.png)

### 02 도커 이미지 다루기 

#### docker image build  - 이미지 빌드
- 옵션 -f : 그냥 docker image build를 사용하게 되면 기본적으로 Dockerfile 이라는 이름을 가진 Dockerfile을 찾아 빌드한다. 
 그외 다른 이름으로 된 Dockerfile을 사용하고 싶다면 -f 옵션을 붙여 주어야 한다. 
 ```docker
    docker image build -f Dockerfile-test -t example/echo:latest .
 ```
- 옵션 --pull : Dockerfile 의 FROM에 기재된 베이스 이미지를 받게 되면 호스트 운영체제에 저장되기 때문에 이미 저장된 레이어 이미지는 다시 받아오지 않는다. 
만일 강제로 이미지를 새로 받아 오고 싶다면 옵션 -pull=true를 사용한다. 
작동 원리는 도커 허브에서 최신 버전이 있는지 확인하고 빌드를 하기 때문에 빌드 속도 면에서는 조금 불리하다. 
```docker
    docker image build --pull=true -t example/echo:latest .
```

#### docker search  - 이미지 검색
- 도커 허브에 등록된 리포지토리를 검색할 수 있다. (docker search [options] 검색키워드)
```docker
    docker search --limit 5 mysql
```

   ![docker_search.png](/assets/images/docker/ch02/docker_search.png)
    
   * 목록 중 네임스페이스가 생략되어 있는 mysql을 볼 수 있는데 이 리포지토리가 공식 mysql 리포지토리이다. 
   공식 리포지토리의 네임스페이스는 모두 library이며 생략할 수 있다.
   
- 리포지토리 검색은 가능하지만, 리포지토리가 관리하는 도커 이미지의 태그까지는 검색할 수 없으니 다음 API를 사용한다.
 
```text
  curl -s 'https://hub.docker.com/v2/repositories/library/golang/tags/?page_size=10' | jq -r '.results[].name'
```
   ![tag_api.png](/assets/images/docker/ch02/tag_api.png)


#### docker image tag  - 이미지 태그 붙이기
- 어플리케이션 파일이나 이미지 빌드에 사용되는 Dockerfile을 변경하고 빌드하면 새로운 ID를 갖는 이미지가 생성되며, 이전 이미지가 latest였어도, 
새로운 이미지가 생성되면 이전 이미지는 tag가 <none>이 된다. (latest 태그는 한개밖에 사용못함)
   ![tag_latest.png](/assets/images/docker/ch02/tag_latest.png)
- 도커의 버전은 이미지 ID 로 관리가 되며, tag는 별칭인데, none으로 변경되면서 어떤 이미지였는지 구분이 쉽지 않으니 tag를 따로 붙여줄 수도 있다. 
```docker
    docker image tag example/echo:latest example/echo:0.1.0
```
   ![diff_tag_same_image.png](/assets/images/docker/ch02/diff_tag_same_image.png)

- 그럼 이제 이미지가 한개가 더 생기는데 id가 동일함을 확인할 수 있어 동일한 이미지를 가리킨다는 것을 알 수 있다. 태그만 다름. 

#### docker image push - 이미지 외부에 공개
- 도커 이미지를 도커 허브 등의 레지스트리에 등록하기 위해 사용한다. 이때 네임스페이스는 자신, 혹은 소속 기관이 소유한 레포지토리에만 등록할 수 있어, 네임스페이스를 변경한다.
   ![change_tag_for_push.png](/assets/images/docker/ch02/change_tag_for_push.png)
- 도커 허브에 푸시하면 터미널에 프로그레스바가 생기고, 등록이 완료되면 sha256 해시값이 출력된다. 올라가고 나면 누구나 (docker image pull) 명령어로 받을 수 있다.
   ![docker_image_push.png](/assets/images/docker/ch02/docker_image_push.png)
   ![docker_image_in_hub.png](/assets/images/docker/ch02/docker_image_in_hub.png)

   
### 03 도커 컨테이너 다루기
#### 도커 컨테이너 생명주기

![container_life_cycle.png](/assets/images/docker/ch02/container_life_cycle.png)
- 실행,정지,파기의 상태를 가진다. 
- 각 컨테이너는 같은 이미지로 생성했다고 해도 별개의 상태를 갖는데, 상태를 가지지 않는 이미지와의 가장 큰 차이이다. 
- 컨테이너가 정지되고 파기되지 않는다면, 디스크에 그대로 남아있게 되고 디스크를 차지하는 용량이 많이 질 것을 대비해 불필요한 컨테이너는 완전히 삭제하는것이 좋다.
- 파기가 된 컨테이너는 다시 실행할 수 없다. 

#### docker container run - 컨테이너 생성 및 실행
- 컨테이너에 이름 붙이기 
    - 컨테이너를 실행하고 나서 ```docker container ls``` 명령으로 컨테이너 목록을 보면 NAMES 목록에 무작위 단어로 이름이 지어진 것을 볼 수 있다. 
     옵션으로 --name을 사용하여 이름을 붙여줄 수 있는데 개발단계에서는 용이하나 실제 운영에서는 같은 이름을 또 생성하려면 삭제하고 진행하여야 하여 잘 사용하지 않는다.

    ```docker
      docker container run -d --name echo example/echo        
    ```
    ![container_name.png](/assets/images/docker/ch02/container_name.png)
    
| 항목 | 내용 |
| :-------------- | :-------------- |
| CONTAINER ID | 컨테이너를 식별하기 위한 유일 식별자 | 
| IMAGE | 컨테이너를 만드는데 사용된 도커 이미지 | 
| COMMAND | 컨테이너에서 실행되는 애플리케이션 프로세스 | 
| CREATED | 컨테이너 생성 후 경과된 시간 | 
| STATUS | Up(실행중), Exited(종료) 등 컨테이너의 실행 상태 | 
| PORTS | 호스트 포트와 컨테이너 포트의 연결관계 (포트 포워딩) | 
| NAMES | 컨테이너의 이름 | 

- 컨테이너 ID만 추출 하기
```docker
docker container ls -q
```

- 컨테이너 목록 필터링 하기 (docker container ls --filter "필터명=값")
    - 아래 예제는 name컬럼으로 필터한다.
        ```docker
        docker container ls --filter "name=echo"
        ```

    - 컨테이너를 생성한 이미지를 기준으로 하면 ancestor라는 필드명을 사용한다. 
        ```docker
        docker container ls --filter "ancestor=example/echo"
        ```
        ![container_filter.png](/assets/images/docker/ch02/container_filter.png)
        
    - 종료된 컨테이너 목록 조회 
        ```docker
        docker container ls -a
        ```

#### docker container stop - 컨테이너 정지 
- 컨테이너 정지 
    ```docker
      docker container stop 컨테이너 ID 또는 컨테이너명
    ```
- 컨테이너 재시작 
    ```docker
    docker container restart 컨테이너 ID 또는 컨테이너명
    ```
    ![container_stop_restart.png](/assets/images/docker/ch02/container_stop_restart.png)


#### docker container rm - 컨테이너 삭제 
개발중이면, 컨테이너를 여러번 실행/정지 할텐데, 그럼 정지된 컨테이너들이 쌓일 수 있고, 이것들을 조회하여 컨테이너 아이디로 삭제할수 있다. 
1. 정지된 컨테이너만 조회 
```docker
 docker container ls --filter "status=exited"  
```
2. ID 혹은 이름을 삭제 
 ```docker
  docker container rm c9ba48fa6962
 ``` 
![container_rm.png](/assets/images/docker/ch02/container_rm.png)
  
- 혹은 ㅈ어지된 컨테이너는 항상 삭제하고 싶은 경우가 있을 수 있따. 이때는
 ```docker container run --rm ```을 사용해 컨테이너를 정지할 때 함께 삭제하게 할 수 있다. 

#### docker container logs - 표준 출력 연결하기
현재 실행 중인 특정 도커 컨테이너의 표준 출력 내용을 확인할 수 있는 명령어로, 컨테이너의 출력 내용중 표준 출력만 확인할 수 있으며 파일에 출력한 것은 볼 수 없다. 

-f 옵션을 사용하면 라이브로 출력이 가능하다. (docker container logs -f 컨테이너ID또는이름)
```docker
docker container logs echo   
```
![container_logs.png](/assets/images/docker/ch02/container_logs.png)

#### docker container exec - 실행 중인 컨테이너에서 명령어 실행하기
실행중인 컨테이너에 대해 원하는 명령어를 실행할 수 있는데, 마치 컨테이너에 ssh로 로그인 한 것처럼 컨테이너 내부를 조작할 수 있다. 
표준 입력 연결을 유지하는 -i와 유사 터미널을 할당하는 -t옵션을 좋바하면 컨테이너를 셸을 통해 다룰 수 있다. 
![container_exec.png](/assets/images/docker/ch02/container_exec.png)

#### docker container cp - 파일 복사하기
실행중인 컨테이너<>컨테이너, 컨테이너<>호스트 간에 파일을 복사하기 위한 명령어이다. 
1. 컨테이너의 파일을 호스트로 복사하려면 (docker container cp 컨테이너ID 혹은 명:원본파일 대상파일)
```docker
docker container cp echo:/echo/main.go .
```
![container_cp_to_host.png](/assets/images/docker/ch02/container_cp_to_host.png)

2. 호스트에서 컨테이너로 파일을 복사하려면 (docker container cp 호스트원본파일 컨테이너ID 또는 명:대상파일)
```docker
 docker container cp ./cp/dummy.txt echo:/tmp 
```
![container_cp_to_container.png](/assets/images/docker/ch02/container_cp_to_container.png)


### 04 운영과 관리를 위한 명령
#### prune - 컨테이너 및 이미지 파기
1. docker container prune : 실행중이 아닌 모든 컨테이너 삭제
2. docker image prune : 태그가 붙지 않은 모든 이미지 삭제
3. docker system prune : 사용하지 않는 도커 이미지 및 컨테이너, 볼륨, 네트워크 등 모든 도커 리소스 일괄적으로 삭제

#### docker container stats - 사용 현황 확인
유닉스 계열 운영체제의 top명령과 같은 역할을 한다고 보면 된다.
```docker
 docker container stats
```
![docker container stats.png](/assets/images/docker/ch02/docker container stats.png)


### 05 도커 컴포즈로 여러 컨테이너 실행하기
- 도커 컨테이너 = 단일 애플리케이션이라고 봐도 무방하며, 애플리케이션 간의 연동 없이는 실용적 수준의 시스템을 구축할 수 없다.
- 도커 컨테이너로 시스템을 구축하면 하나 이상의 컨테이너가 서로 통신하며 그 사이의 의존관계가 생기게된다. 
- 컨테이너간의 의존관계를 고려한 포트포워딩, 컨테이너의 동작을 제어하기 위한 설정파일 및 환경변수 전달 등의 고려가 필요하다. 

#### docker-compose 명령으로 컨테이너 실행하기
- yaml 포맷으로 기술된 설정 파일로, 어려 컨테이너의 실행을 한번에 관리할 수 있게 해준다. 
    1. version : docker-compose 파일을 내용을 해석하기 위한 문법 버전
    2. services
        - echo : 컨테이너 이름
        - image : 도커 이미지
        - ports : 포트 포워딩 설정
        
```yaml
version: "3"
services:
  echo:
    image: example/echo:latest
    ports:
      - 9000:8080
```

- docker-compose.yml에 기술된 컨테이너를 모두 실행하려면 ```docker compose up```을 활용하면 된다. 
![docker_compose_up.png](/assets/images/docker/ch02/docker_compose_up.png)

- docker-compose.yml에 기술된 컨테이너를 모두 정지하려면 ```docker compose down```을 활용하면 된다. 
- 이미지를 함께 빌드해서 새로 생성한 이미지를 실행할 수 도 있다. image속성 대신, build 속성을 사용하면 된다. build 속성에는 Dockerfile의 경로를 지정해준다.
docker-compse 가 이미 이미지를 빌드한 적이 있다면, 생략하고 컨테이너가 실행되지만 --build 옵션을 사용해서 강제로 다시 빌드하게 할 수 있다.
```yaml
version: "3"
services:
  echo:
#    image: example/echo:latest
    build: .
    ports:
      - 9000:8080
```
```yaml
docker compose up -d --build
```
![docker_compose_up_build.png](/assets/images/docker/ch02/docker_compose_up_build.png)

### 06 컴포즈로 여러 컨테이너 실행하기
docker-compose 는 여러 컨테이너를 실행할때 매우 유용하다. 예로 젠킨스를 사용해보자. 

#### 젠킨스 컨테이너 실행하기
```yaml
version: "3"
services:
  master:
    container_name: master
    image: jenkins/jenkins:lts
    ports:
      - 8080:8080
    volumes:
      - ./jenkins_home:/var/jenkins_home
```
- volumes : 호스트와 컨테이너 사이에 파일을 공유할 수 있는 매커니즘이다. 젠킨스 컨테이너를 호스트쪽에서 편리하게 다룰 수 있도록 volumes를 정의해 
호스트쪽 현재 작업 디렉터리 바로 아래에 jenkins_home 디렉터리를 젠킨스 컨테이너의 /var/jenkins_home에 마운트한다. 
- 컨테이너를 실행하면 secret 정보가 /var/jekins_home에 저장되고 접근하여 확인할 수 있다. 


#### 마스터 젠킨스 용 SSH 키 생성
- 실제로 젠킨스를 운영할 때 단일 서버로 운영하는 경우는 흔치않다. 
- 관리/작업실행 지시는 마스터 인스턴스가 진행하고, 작업을 실제로 진행하는 것은 슬레이브 인스턴스가 담당한다. 
1. 마스터가 슬레이브에 접근할 수 있도록 마스터 컨테이너에서 SSH키를 생성한다. 
```docker
 docker container exec -it master ssh-keygen -t rsa -C "" 
```
2. /var/jenkins_home/.ssh/id_sa.pub 파일이 마스터 젠킨스가 슬레이브 젠킨스에 접속할 때 사용할 키이다. 
![jenkins_ssh.png](/assets/images/docker/ch02/jenkins_ssh.png)

3. 슬레이브 젠킨스 컨테이너 생성
    - 슬레이브 컨테이너는 SSH로 접속하는 슬레이브용도로 구성된 도커 이미지 jenkins/ssh-slave를 사용한다. 
    - 환경변수로 JENKINS_SLAVE_SSH_PUBKEY를 설정하는데, SSH로 접속하는 상대가 이 키를 보고 마스터 젠킨스임을 식별한다. 
        - 슬레이브 컨테이너의 ~/.ssh/authorized_keys 파일에 마스터 컨테이너의 SSH공개키가 추가된다. 

4. ssh 접속 대상 설정
   - 마스터 컨테이너가 어떻게 슬레이브 컨테이너를 찾아 추가할 것이가는, 마스터 컨테이너 설정에 links 속성을 사용하여 services 그룹에 해당하는 다른 컨테이너와 통신할 수 있다. 
 
5. 최종 yml

```yaml
version: "3"
services:
  master:
    container_name: master
    image: jenkins/jenkins:lts
    ports:
      - 8080:8080
    volumes:
    - ./jenkins_home:/var/jenkins_home
    links:
      - slave01

  slave01:
    container_name: slave01
    image: jenkins/ssh-agent
    environment:
      - JENKINS_AGENT_SSH_PUBKEY=
```  

![jenkins_master_slave.png](/assets/images/docker/ch02/jenkins_master_slave.png)

- docker compose up으로 띄우고 난뒤 jenkins에 접속하여 노드관리에서 slave node추가가 필요. 
![jenkins_master_slave_node.png](/assets/images/docker/ch02/jenkins_master_slave_node.png)

- 에러1. 책에있는 jenkins 이미지를 사용하면 플러그인 다운로드가 모두 실패하여 실습을 진행할 수 없다. 도커허브의 젠킨스 공식 이미지를 받아온다. (ssh 도 마찬가지)
    - https://hub.docker.com/_/jenkins
    - https://hub.docker.com/r/jenkins/ssh-agent
- 에러2. 젠킨스를 실행하고 slave를 연결하려고하면 java를 찾지 못해서 에러가 난다. launch log에 보면 JAVA_HOME 경로가 나오는데 이 경로를 복사하여, 
설정에서 고급에 자바 경로에 넣어준다. 
![jenkins_java_path_config.png](/assets/images/docker/ch02/jenkins_java_path_config.png)
![jenkins_complete.png](/assets/images/docker/ch02/jenkins_complete.png)


-------
관련 서적

도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문 [link]

[link]: http://www.yes24.com/Product/Goods/70893433
