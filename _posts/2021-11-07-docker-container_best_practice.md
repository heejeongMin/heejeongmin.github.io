---
layout: post
title: 컨테이너 실전 구축 및 배포
subtitle: 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문
categories: docker
tags: [DOCKER, CONTAINER, INFRASTRUCTURE]
---

Chapter03 컨테이너 실전 구축 및 배포
---

## 컨테이너 1개의 하나의 관심사
- 하나의 컨테이너는 한 가지 역할이나 문제 영역에 집중해야한다. 
    - 2장의 젠킨스 마스터와 슬레이브를 각각 컨테이너로 분리하였을때 슬레이브의 인스턴스가 부족한 경우 슬레이브만 늘리면된다. 
    - 웹 어플리케이션 스택에서도 프록시, 애플리케이션, 데이터베이스가 각자의 역할이 있고 이를 각자의 컨테이너로 나눈다고 생각하면 된다. 
    
## 컨테이너의 이식성
- 도커의 큰 장점은 이식성에 있지만, 단점 또한 존재한다. 
1. 커널 및 아키텍처의 차이
    - 도커에서 사용되는 컨테이너형 가상화 기술은 호스트 운영체제와 커널 리소스를 공유한다. => 도커 컨테이너를 실행하려면 호스트가 특정 CPU 아키텍쳐 혹은 운영체제를 사용해야 한다는 의미.
    ![docker_resource.png](/assets/images/docker/ch03/docker_resource.png)  
    - 모든 환경에서 동작하는 컨테이너는 없다. 
        - 라즈베리파이와 같은 ARM 계열의 armv7I 아키텍처를 채용한 플랫폼의 경우, 인텔의 x86_64 아키텍처에서 빌드한 도커 컨테이너를 실행할 수 없다. 
          우리는 보통 CentOS, Ubuntu 같은 알파인 리눅스를 기반으로 도커 이미지를 빌드하거나 사용하게 되는데, 이것 또한 x86_64 아키텍쳐에서 실행하는 것을 전제로 만들어진 것이며, 다른 환경에서 정상 작동을 보장하지 않는다. 
        - 최근 윈도우 플랫폼에서 도커를 실행하는 것을 전제로 한 microsoft/widnowsservercore 라는 윈도우 서버 기반 이미지도 제공하기 시작했는데, 이 도커 컨테이너는 반대로 리눅스, macOS환경에서 실행되지 않는다. 
   
2. 라이브러리와 동적 링크 문제
    - 어플리케이션이 어떤 라이브러리를 사용하는가에 따라 컨테이너의 이식성을 좌우하기도 한다. 
        - 라이브러리의 정적 링크 사용
            - 정적 링크는 애플리케이션에 사용되는 라이브러리를 내부에 포함하고 있다.
            - 애플리케이션의 크기가 비대해지는 경향이 있다. => 보완을 위해, 모든 빌드 프로세스를 실행하는 도커 컨테이너를 만들고 이 안에서 애플리케이션을 빌드한다. 
            - 이식성은 뛰어나다. 
            
        - 라이브러리의 동적 링크 사용
            - 애플리케이션의 크기가 작아진다.
            - CI와 컨테이너에서 사용하는 라이브러리가 충돌하는 경우 애플리케이션이 동작하지 않을 수 있다. 
            - CI에 맞추어 매번 컨테이너에서 사용하는 라이브러리를 수정할 수 없기 때문에 되도록 정적 링크를 사용하는 것이 추천된다.
            
3. 도커 친화적인 애플리케이션
도커 친화적인 애플리케이션이 되려면 환경 변수를 적극 활용한다. 재사용성과 유연성을 가질 수 있도록 옵션을 만들어 두고, 이 옵션에 따라 애플리케이션의 동작을 제어해야 한다. 특히 도커 환경에서는 외부에서 동작을 제어할 수 있어야 한다. 
애플리케이션의 동작을 제어하는 방법으로 다음과 같은 것들이 있다. 
    1. 실행 시 인자를 사용
        - CMD, ENTRYPOINT 와 같은 인스트럭션을 사용하여 컨테이너를 실행할때 사용할 명령을 정의해 둘 수 있고, 실행 시 전달을 받아 오버라이드도 가능하다. 
        - 단, 너무 많은 인자를 사용하는 경우 처리가 복잡해지고 관리가 어려울 수 있으니, 필요한 것만 잘 선정해야 한다.
    2. 설정 파일 사용 
        - 가장 많이 사용하는 방법으로, 실행할 어플리케이션에 development/production과 같이 환경 이름을 부여하고, 그에 따라 설정 파일을 변경해 가며 사용하는 방식이다. 
        - 도커의 경우, 설정 파일을 컨테이너 밖에서 실행 시 전달하는 형태로 사용한다면 실행환경을 추가해도, 이미지를 새로 빌드할 필요가 없다. 
        - 호스트에 위치한 환경별 설정 파일을 컨테이너에 마운트 해주는 것도 한가지 방법이지만, 호스트에 대한 의존성이 생기므로 관리 운영면에서는 좋지 않다. 
    3. 애플리케이션 동작을 환경 변수로 제어
        - 환경변수에 값을 설정하는 방식으로 매번 이미지를 다시 빌드하지 않아도 되는 장점이 있다. 
        - 외부에서 설정을 주입하는 방식임으로 애플리케이션과 별도의 리포지토리에 관리하는것이 일반적이다. 
        - 컴포즈를 사용하는 경우, docker-compose.yml 파일의 env 속성에 기술해 관리한다. 쿠버네티스도 비슷한 기능을 제공한다. 
        - 이 방법의 단점은 환경 변수의 특성상 key/value 형태이기 때문에, 계층 구조를 가지기 어렵다.
    4. 설정 파일에 환경 변수를 포함
        - 설정파일에 환경변수를 포함할 수 있다면 환경 변수의 장점과 설정 파일의 장점을 모두 취할 수 있다. 
            ```properties
              db.driverClass=${DB_DRIVER_CLASS:com.mysql.jdbc.Driver}
              db.jdbcUrl=${DB_JDBC_URL}
              db.user=${DB_USER}
              db.password=${DB_PASSWORD}
              db.initialSize=${DB_INITIAL_SIZE:10}
              db.maxActive=${DB_MAX_ACTIVE:50}
              db.maxIdle=${DB_MAX_IDLE:20}
              db.minIdle=${DB_MIN_IDLE:10}
            ```
          
## 퍼시스턴스 데이터를 다루는 방법
- 도커 컨테이너가 실행 중에 작성/수정된 파일은 호스트 파일 시스템에 마운트되지 않는 한 컨테이너가 파기될때 호스트에서도 함께 삭제된다. 
- 컨테이너를 사용해서 상태를 갖는 애플리케이션을 운영하려면 새로운 버전의 컨테이너가 배포되도 이전 버전의 컨테이너에서 사용하던 파일 및 디렉토리를 사용할 수 있어야한다. 

### 데이터 볼륨 
- 호스트와 컨테이너 사이의 디렉터리 공유 및 재사용 기능을 제공한다. 
- 이미지를 수정하고 컨테이너를 생성해도 같은 데이터 볼륨을 계속 사용할 수 있다. 데이터 볼륨은 컨테이너가 파기되어도 디스크에 그대로 남기때문
- 데이터볼륨을 생성하려면 ```docker container run``` 명령에 -v 옵션을 사용하면 된다. 
```docker
docker container run [option] -v 호스트_디렉터리:컨테이너_디렉터리 리포지토리명[:태그] [명령] [명령인자]
```
![data_volume.png](/assets/images/docker/ch03/data_volume.png)  

### 데이터 볼륨 컨테이너
- 위의 방법은 호스트와 컨테이너 사이의 디렉터리 공유이고, 데이터 볼륨 컨테이너 방법은 컨테이너 간에 디렉터리를 공유한다.
- 데이터 볼륨 컨테이너는 이름 그대로 데이터 저장만이 목적이다. 디스크에 저장된 컨테이너가 갖는 퍼시스턴스 데이터를 볼륨으로 만들어 다른 컨테이너에 공유하는 방식
- 호스트머신의 스토리지에 저장된다는 점에서 데이터볼륨과 데이터볼륨 컨테이너는 같다. 
- 데이터 볼륨 컨테이너의 볼륨은 호스트머신의 /var/lib/docker/volumes/ 아래에 위치한다. 
- 데이터 볼륨이 데이터 볼륨 컨테이너 안에 캡슐화됨으로 호스트에 대해 아는 것이 없어도 데이터 볼륨을 사용할 수 있다. 결합이 느슨해짐. 

![data_volume_container.png](/assets/images/docker/ch03/data_volume_container.png)  

#### 데이터 볼륨에 MySQL 데이터 저장하기
```dockerfile
FROM busybox

VOLUME /var/lib/mysql

CMD ["bin/true"]
```

- 빌드하고 실행한다. 실행 시, CMD 인스트럭션에서 셸을 실행하는 것이 전부이기 때문에 끝나면 바로 종료된다.
```docker
 docker image build -t example/mysql-data:latest .
 docker container run -d --name mysql-data example/mysql-data:latest
```

- mysql을 실행한다. --volumes-from 옵션을 사용해서 데이터볼륨컨테이너 mysql-data를 MySQL 컨테이너에 마운트한다. 
 => 이제 데이터는 /var/lib/mysql이 아닌, 데이터 볼륨 컨테이너에 저장된다.

```docker
docker container run -d --rm --name mysql \                
 -e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" \ 
 -e "MYSQL_DATABASE=volume_test" \
 -e "MYSQL_USER=example" \
 -e "MYSQL_PASSWORD=example" \
 --volumes-from mysql-data \
 mysql:5.7
```

- mysql에 접근하여, create, insert 쿼리를 실행하여 데이터가 어디에 저장되는지 확인한다.
```docker
docker container exec -it mysql mysql -u root -p volume_test
CREATE TABLE user(id int PRIMARY KEY AUTO_INCREMENT, name VARCHAR(255)) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_unicode_ci;
INSERT INTO user (name) VALUES ('gihyo'), ('docker'), ('Solomon Hykes');
```

- 컨테이너 파기 후 재접속 하여 데이터가 마운트 되는지 확인
```docker
docker container stop mysql
docker container exec -it mysql mysql -u root -p volume_test
select * from user;
```
![docker_volume_container_after_container_destroyed.png](/assets/images/docker/ch03/docker_volume_container_after_container_destroyed.png)  

- 데이터 볼륨은 충분히 좋은 기능이지만, 범위가 같은 도커 호스트 안으로 제한된다. 
- 다른 도커 호스트로 이전해야할 경우, 데이터를 파일로 익스포트한뒤 호스트로 꺼내면 된다. 
- 익스포트한 파일을 다른 도커 호스트에 옮기려면 새로운 데이터 볼륨 컨테이너를 만들고, 그 컨테이너 안에 익스포트 파일의 압축을 풀어주면된다.
```docker
docker container run -v ${PWD}:/tmp \
--volumes-from mysql-data \
busybox \
tar cvzf /tmp/mysql-backup.tar.gz /var/lib/mysql
```
![volume_export.png](/assets/images/docker/ch03/volume_export.png)  
![volume_export_result.png](/assets/images/docker/ch03/volume_export_result.png)  

> ```docker image save``` 라는 명령어도 있지만, 도커 이미지를 파일로 아카이빙하는 용도이기 때문에 데이터볼륨에는 해당되지 않는다.
> Netshare plugin 등 다양한 클라우드 스토리지를 지원하는 플러그인도 있긴하다. 



## 컨테이너 배치 전략
많은 트래픽을 처리할 수 있는 실용적인 시스템은 여러 컨테이너가 각기 다른 호스트에 배치되는 경우가 많다. 

### 도커 스웜 (Docker swarm)
- 도커 스웜은 여러 도커 호스트를 클러스터로 묶어주는 컨테이너 오케스트레이션 도구의 한 종류이다. 
- 스웜은 예전에는 독립적은 도구였으나, 현재는 도커에 내장되어 있어 스웜모드를 활성화 해야 사용할 수 있다.
- 어느 도커 호스트에 어떤 컨테이너를 배치해야 하는지, 서로 다른 호스트에 위치한 컨테이너 간의 통신은 어떻게 제어하는지 등의 조율을 가능하게 해준다. 

| 이름 | 역할 | 대응하는 명령어 |
| :-------------- | :-------------- | :-------------- |
| 컴포즈 | 여러 컨테이너로 구성된 도커 애플리케이션을 관리(주로 단일 호스트) | docker-compose |
| 스웜 | 클러스터 구축 및 관리(주로 멀티 호스트) | docker swarm |
| 서비스 | 스웜에서 클러스터 안의 서비스(컨테이너 하나 이상의 집합)를 관리 | docker service |
| 스택 | 스웜에서 여러 개의 서비스를 합한 전체 애플리케이션을 관리 | docker stack |

#### 여러 대의 도커 호스트로 스웜 클러스터 구성하기
- 여러대의 호스트를 가지려면 클라우드, 도커 머신즈 (맥은 virtualbox 드라이버, 윈도우인경우 hyperv 드라이버) 필요하지만, 
'도커 호스트 역할을 할 도커 컨테이너를 여러 개 실행하는 방법' 으로 도커 인 도커(Docker in Docker, dind) 라는 기능을 사용할 수 있다.

- 구성 
    ![dind.png](/assets/images/docker/ch03/dind.png)  
    
    1. registry : 도커 레지스트리 역할 컨테이너. manager, worker 컨테이너가 사용하게될 컨테이너이다. 왜냐하면, dind 환경에서 외부 도커 데몬에서 빌드된 도커 이미지를 사용할 수 없기 때문이다. 
    따라서, 외부 도커에 저장된 이미지를 registry컨테이너에 등록하였다가 manager/worker가 받아갈 수 있게 한다. 
    실제 업무에서는 도커 허브나 별도로 사전 구축한 인하우스 레지스트리를 사용하는 경우가 많다. 
    
    2. manager : 스웜 클러스터 전체를 제어하는 역할한다. 여러 대 실행되는 도커 호스트 (worker)에 서비스가 담긴 컨테이너를 적절히 배치한다.
    
    3. worker : 여러 대 실행되는 도커 호스트 

- docker-compose 작성하여, registry, manager, worker 생성 

```yaml
version: "3"
services:
  registry:
    container_name: registry
    image: registry:2.6
    ports:
      - 5000:5000
    volumes:
      - "./registry-data:/var/lib/registry"

  manager:
    container_name: manager
    image: docker:18.05.0-ce-dind
    privileged: true
    tty: true
    ports:
      - 8000:80
      - 9000:9000
    depends_on:
      - registry 
    expose:
      - 3375
    command: "--insecure-registry registry:5000"
    volumes:
      - "./stack:/stack"

  worker01:
    container_name: worker01
    image: docker:18.05.0-ce-dind
    privileged: true
    tty: true
    depends_on:
      - manager
      - registry 
    expose:
      - 7946
      - 7946/udp
      - 4789/udp
    command: "--insecure-registry registry:5000"

  worker02:
    container_name: worker02
    image: docker:18.05.0-ce-dind
    privileged: true
    tty: true
    depends_on:
      - manager
      - registry 
    expose:
      - 7946
      - 7946/udp
      - 4789/udp
    command: "--insecure-registry registry:5000"

  worker03:
    container_name: worker03
    image: docker:18.05.0-ce-dind
    privileged: true
    tty: true
    depends_on:
      - manager
      - registry 
    expose:
      - 7946
      - 7946/udp
      - 4789/udp
    command: "--insecure-registry registry:5000"
```
![dind_compose_up.png](/assets/images/docker/ch03/dind_compose_up.png)  

- dind 컨테이너를 여러대 띄우기만 한 것이라 manager에게 스웜모드를 활성화 시켜준다. d
```docker
docker container exec -it manager docker swarm init
```
![swarm_init.png](/assets/images/docker/ch03/swarm_init.png)  

- 생성된 join token을 사용하여 3대의 노드를 스웜 클러스터에 워커로 등록한다. 
- manager 및 모든 worker 컨테이너는 컴포즈로 생성한 기본 네트워크 위에설 실행되기 때문에 서로 컨테이너명으로 식별할 수 있다. 
```docker
docker container exec -it worker01 docker swarm join \
--token SWMTKN-1-50g592tsmld4ueqs8mc1fhqawk4bymcx6vkkwklurgpq2syqph-2ad3eyhr23rtaqz0p8eolsq1j manager:2377
```
![docker_cluster.png](/assets/images/docker/ch03/docker_cluster.png)  

#### 도커 레지스트리에 이미지 등록하기
- 도커 레지스트리 역할을 할 registry 컨테이너가 실행 중임으로 호스트에서 registry 컨테이너에 도커 이미지를 등록해야한다. (2장에서 사용한 echo 사용)
```docker
docker image tag example/echo:latest localhost:5000/example/echo:latest
```
- 태그이름은 [레지스트리_호스트/]리포지토리명[:태그] 형태인데, 여기서 레지스트리_호스트는 이미지를 등록하거나 내려받는 레지스트리를 의미한다. 
- 현재 registry 컨테이너는 호스트에서 localhost:5000과 같이 접근할 수 있음으로 주소를 붙여 태그를 완성하였고, ```docker image push```를 하면 registry컨테이너에 이미지가 등록한다.
- 이 방법이 비공개 레지스트리를 사용하는 기본 방법이다. 


- registry로 이미지 푸시하고, worker에서 pull받기
```docker
docker image push localhost:5000/example/echo:latest
docker container exec -it worker01 docker image pull registry:5000/example/echo:latest
docker container exec -it worker01 docker image ls
```
![worker_image.png](/assets/images/docker/ch03/worker_image.png)  


#### 서비스
- 애플리케이션을 구성하는 일부 컨테이너를 제어하기 위한 단위
- manager 컨테이너에서 서비스 생성 후 생성된 서비스 조회
```docker
docker container exec -it manager docker service create --replicas 1 --publish 8080:8080 --name echo registry:5000/example/echo:latest
docker container exec -it manager docker service ls
```
![docker_service.png](/assets/images/docker/ch03/docker_service.png)  

- 서비스 scale out 후 스웜 클러스터의 노드에 분산 배치됨을 확인 
```docker
docker container exec -it manager docker service ps echo
```
![docker_service_scale_out.png](/assets/images/docker/ch03/docker_service_scale_out.png)  
![docker_service_scale_out_distribute.png](/assets/images/docker/ch03/docker_service_scale_out_distribute.png)  

- 서비스를 삭제하면 복제되었던 컨테이너들도 함께 삭제
```docker
docker exec -it manager docker service rm echo 
docker container exec -it manager docker service ls
```

#### 스택
- 하나 이상의 서비스를 그룹으로 묶은 단위로 애플리케이션 전체 구성을 정의한다. 
- 스택은 스웜에서 동작하는 스케일 인, 아웃, 제약조건 부역 가능한 컴포즈 이다. 
- ```docker stack``` 하위 명령으로 조작한다. 
- 스택을 사용해 배포된 서비스 그룹은 overlay 네트워크에 속한다. 
    - overlay 네트워크란 여러 도커 호스트에 걸쳐 배포된 컨테이너 그룹을 같은 네트워크에 배치하기 위한 기술로, 서로 다른 호스트에 위치한 컨테이너 끼리 통신 할 수 있다.
    - 어떤 overlay를 사용할지 설정하지 않으면 스택마다 각자 overlay 네트워크를 생성하고 그 안에 서비스 그룹이 속하게 되고, 통신이 불가능하다. 따라서 한 overlay네트워크에 각 서비스를 소속시켜야한다.  
        ![wo_an_overlay.png](/assets/images/docker/ch03/wo_an_overlay.png)  
 
- overlay network 생성하기 
```shell script
docker container exec -it manager docker network create --driver=overlay --attachable ch03
```    
  
- 스택 배포하기 (```docker stack deploy [options] 스택명```)

| 스택 하위 명령 | 내용 | 
| :-------------- | :-------------- | 
| deploy | 스택을 새로 배포, 혹은 업데이트함 |
| ls | 배포된 스택의 목록을 출력 |
| ps | 스택에 의해 배포된 컨테이너의 목록을 출력 |
| rm | 배포된 스택을 삭제 |
| services | 스택에 포함된 서비스 목록을 출력 |


스택에 사용될 컴포즈 파일을 stack 경로에 생성한다. 이때 stack 경로는 docker-comopse.yml 레벨에 만들어야하는데, 
이유는 manager volume에 ./stack을 호스트로 지정하였기 때문이다. 
```yaml
version: "3"
services:
  nginx:
    image: gihyodocker/nginx-proxy:latest
    deploy:
      replicas: 3
      placement:
        constraints: [node.role != manager]
    environment:
      SERVICE_PORTS: 80
      BACKEND_HOST: echo_api:8080
    depends_on:
      - api
    networks:
      - ch03
  api:
    image: registry:5000/example/echo:latest
    deploy:
      replicas: 3
      placement:
        constraints: [node.role != manager]
    networks:
      - ch03

networks:
  ch03:
    external: true
```

배포 명령어
```shell script
docker container exec -it manager docker stack deploy -c /stack/ch03-webapi.yml echo
```
![docker_stack_deploy.png](/assets/images/docker/ch03/docker_stack_deploy.png)  

- 스택에 배포된 컨테이너 확인하기
```shell script
docker container exec -it manager docker stack ps echo
```
![docker_stack_ps_echo.png](/assets/images/docker/ch03/docker_stack_ps_echo.png)  

- visualizer를 사용해 컨테이너 배치 시각화
    - 스웜클러스터에 컨테이너 그룹이 어떤 노드에 어떻게 배치되었는지 시각화해주는 visualizer 라는 애플리케이션이 있다. 
    - 동일한 stack 디렉토리에 yml 파일을 작성한다. 
    ```shell script
    version: "3"
    
    services:
      app:
        image: dockersamples/visualizer
        ports:
          - "9000:8080"
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
        deploy:
          mode: global
          placement:
            constraints: [node.role == manager]
    ```   
        - deploy.mode.global : 모든 노드에 배치
        - deploy.mode.placement.constraints : manager 노드 빼고
        - ports : 호스트와 manager 노드 사이의 포트가 9000 이기 때문에 9000으로 받아서 8080으로 포워드 시켜준다. 
    
    ```shell script
      docker container exec -it manager docker stack deploy -c /stack/visualizer.yml visualizer
    ```
    ![visualizer.png](/assets/images/docker/ch03/visualizer.png)  

- 스택 삭제
```docker stack rm 스택명``` 명령을 사용하면 배포된 서비스가 스택째로 삭제된다. 
```shell script
docker container exec -it manager docker stack rm echo
```

#### 스웜 클러스터 외부에서 서비스 사용하기
- visualizer가 외부에서 접근이 가능했던 이유는 manager에 배치되도록 하였기 때문이다. 
- 만일 워커노드에 있는 ehco_nginx에 접근하려면 어떻게 해야할까 ? -> 외부에서 오는 트래픽을 복적하는 서비스로 보내주는 프록시 서버가 있어야한다. 
    - HAProxy를 프록시 서버로 사용해 스웜 클러스터 외부에서 ehco_nginx에 접근할 수 있게 할 수 있다. 
      HAProxy는 다기능 프록시 서버로, HTTP, TCP load balancer 역할을 한다. 
      
- HAProxy yml
```yaml
version: "3"
services:
  haproxy:
    image: dockercloud/haproxy
    networks: 
      - ch03 
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
    ports:
      - 80:80
      - 1936:1936 # for stats page (basic auth. stats:stats)

networks:
  ch03:
    external: true
```    
  
- HAProxy가 서비스를 찾을 수 있게 nginx 환경변수에 SERVICE_PORTS 추가후 배포 (이미 되어있음)
```shell script
docker exec -it manager docker stack deploy -c /stack/ch03-webapi.yml echo
docker exec -it manager docker stack deploy -c /stack/ch03-ingress.yml ingress
```
![service_ls_curl.png](/assets/images/docker/ch03/service_ls_curl.png)  


-------
관련 서적

도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문 [link]

[link]: http://www.yes24.com/Product/Goods/70893433
