---
layout: post
title: Docker intro
subtitle: Kubernetes In Action
categories: kubernetes
tags: [KUBERNETES, DOCKER, CONTAINER, INFRASTRUCTURE]
---

### 도커 명령어와 동작방식
        docker run <image>
        docker run <image>:tag
        
   1. docker run 을 실행하였을 때 도커가 하는 일
      - 로컬 머신에 이미지가 저장되어 있는지 확인 
      - 로컬에 이미지가 없으면 레지스트리(도커허브)로 부터 이미지를 pull
      - 격리된 컨테이너에서 이미지를 실행
      - :tag가 없다면 최신본을 사용하는 것으로 간주된다. 
      
   2. 이미지를 위한 Dockerfile 생성  
      - 애플리케이션을 이미지로 패킹하기 하려면 Dockerfile 이라는 파일을 생성한다. 
      - 도커파일 예제  
        
            1   FROM adoptopenjdk/openjdk11:ubi 
            2   RUN mkdir /opt/app
            3   COPY build/libs/com-workbook-crane-0.0.1-SNAPSHOT.jar /opt/app
            4   CMD ["java", "-jar", "/opt/app/com-workbook-crane-0.0.1-SNAPSHOT.jar"]
 
      라인 1 : FROM <image>은 *이미지 생성의 기반이 되는 기본 이미지*로 사용할 컨테이너 이미지 정의  
      라인 2 : RUN <command>은 컨테이너 에서 실행할 명령어  
        - ADD 와 RUN 의 차이  
        https://stackoverflow.com/questions/31171851/docker-build-add-vs-run-curl
               
                ADD is executed in docker host.   
                The ADD instruction copies new files, directories or remote file URLs from <src> and adds them to the filesystem of the image at the path <dest>.
                    ex) ADD app.js /app.js
                 
                RUN is executed inside your container.  
                The RUN instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.
                    
      라인 3 : COPY <src> <dest> 호스트 시스템에서 컨테이너로 파일 복사  
        - ADD 와 COPY 의 차이 (tar를 추출하는 것이 아니라면, 좀 더 명시적인 COPY를 사용하길 권장)  
            https://nickjanetakis.com/blog/docker-tip-2-the-difference-between-copy-and-add-in-a-dockerile
                
                   COPY takes in a src and destination. 
                   It only lets you copy in a local file or directory from your host 
                   (the machine building the Docker image) into the Docker image itself.

                   ADD lets you do that too, but it also supports 2 other sources. 
                   First, you can use a URL instead of a local file / directory. 
                   Secondly, you can extract a tar file from the source directly into the destination.

   3. 컨테이너 이미지 생성  

            docker build -t <tagname> . 
            
         - Dockerfile이 있는 위치에서 명령어를 실행 
         - 디렉터리의 전체 컨텐츠가 도커 데몬에 업로드 되어 이미지가 빌드됨
            - 빌드 디렉토리의 모든 파일이 데몬에 업로드 됨으로 불필요한 파일을 디렉토리에 포함시키면 그만큼 느려지는데, 
            특히 원격 머신에 데몬이 있는 경우 성능 저하를 가지고 온다. 
            
            - 도커 클라이언트와 데몬이 같은 머신에 있을 필요는 없는데, 리눅스가 아닌 OS에서 도커를 사용하면
             도커 클라이언트는 호스트 OS에 위치하고, 데몬은 가상머신 내부에 위치한다.
             
         - image 의 이름은 소문자만 가능하다.
         ![repository_lowercase.png](/assets/images/kubernetes/repository_lowercase.png)

        
        //todo 78쪽 사진
        
  * 이미지 레이어  
       - 서로 다른 이미지가 여러개의 레이어를 공유할 수 있기 때문에 이미지 저장/전송에 효과적
       - 이미지를 가지고 올때도 각 레이어를 개별적으로 다운로드 한다. 
       - 이미지 프로세스가 완료되면 새로운 이미지가 로컬에 저장되고, 저장된 이미지 리스트를 도커에게 요청할 수 있다.
              
  * Dockerfile을 이용한 이미지 빌드와 수동 빌드 비교하기  
       - 도커 컨테이너 이미지를 생성하는 일반적인 방법으로 이미지 빌드를 자동화 할 수 있다.
       - Dockerfile로 빌드 하는 절차 (수동으로 할 시 아래와 같고 Dockerfile을 사용하면 변경사항만 적용하고 계속 재사용 가능) 
         1. 이미지에서 컨테이너를 실행
         2. 컨테이너 내부에서 명령어 수행
         3. 컨테이저 밖에서 최종 상태를 새로운 이미지로 커밋
         
  '4. 컨테이너 이미지 실행
  
        docker run --name crane_server -p 8080:8080 -d -t crane
        
   - "--name crane_app" 이라는 이름의 새로운 컨테이너를 실행하는 명령어이다. 
   - "-d" 백그라운드 실행을 의미
   - "-p 8080:8080" -> 로컬머신의 8080 포트가 컨테이너 내부의 8080포트와 매핑
        - 활용한 프로젝트에서는 docker-compose 파일을 사용하였고, 7070으로 포트를 띄워 매핑시켰다. 
          명령어는 "docker compose up"
          
            
      crane-app:
          container_name: crane-server
          build:
            context: .
            dockerfile: Dockerfile
          image: crane
          working_dir: /crane-app
          ports:
            - 7070:8080
          depends_on:
            crane-mysql:
              condition: service_healthy
    

 - 실행중인 모든 컨테이너 조회 
        
           
        docker ps     
    
   ![running_containers.png](/assets/images/kubernetes/running_containers.png)
        - CONTAINER ID : 16진수의 도커 컨테이너 ID  
        - NAMES : 컨테이너 이름  
        - "docker ps -a" 을 하면 현재 정지되어있는 컨테이너도 확인 할 수 있다. 
   
 - 컨테이너에 관한 추가 정보 얻기
    
    
        docker inspect [컨테이너아이디 | 컨테이너이름]
        
   
 - "docker ps" 는 컨텡너의 기본 정보만 출력하지만 docker inspect는 도커의 상세정보를 JSON 형식으로 출력한다. 
        
        [
            {
                "Id": "d314bf4b9316ea212ae8986032de75505d9411bbdcda14a3b01a3b96a60d0f9b",
                "Created": "2021-09-27T05:00:20.9438056Z",
                "Path": "java",
                "Args": [
                    "-jar",
                    "/opt/app/com-workbook-crane-0.0.1-SNAPSHOT.jar"
                ],
                "State": {
                    "Status": "running",
                    "Running": true,
                    "Paused": false,
                    "Restarting": false,
                    "OOMKilled": false,
                    "Dead": false,
                    "Pid": 64915,
                    "ExitCode": 0,
                    "Error": "",
                    "StartedAt": "2021-09-27T05:00:57.8374213Z",
                    "FinishedAt": "0001-01-01T00:00:00Z"
                },
                "Image": "sha256:9036c6f430105a2aa235728cea12700172de3bb7d061faa3ff5dd42863ff8b84",
                "ResolvConfPath": "/var/lib/docker/containers/d314bf4b9316ea212ae8986032de75505d9411bbdcda14a3b01a3b96a60d0f9b/resolv.conf",
                "HostnamePath": "/var/lib/docker/containers/d314bf4b9316ea212ae8986032de75505d9411bbdcda14a3b01a3b96a60d0f9b/hostname",
                "HostsPath": "/var/lib/docker/containers/d314bf4b9316ea212ae8986032de75505d9411bbdcda14a3b01a3b96a60d0f9b/hosts",
                "LogPath": "/var/lib/docker/containers/d314bf4b9316ea212ae8986032de75505d9411bbdcda14a3b01a3b96a60d0f9b/d314bf4b9316ea212ae8986032de75505d9411bbdcda14a3b01a3b96a60d0f9b-json.log",
                "Name": "/crane-server",
                "RestartCount": 0,
                "Driver": "overlay2",
                "Platform": "linux",
                "MountLabel": "",
                "ProcessLabel": "",
                "AppArmorProfile": "",
                "ExecIDs": null,
                "HostConfig": {
                    "Binds": [],
                    "ContainerIDFile": "",
                    "LogConfig": {
                        "Type": "json-file",
                        "Config": {}
                    },
                    "NetworkMode": "com-workbook-crane_default",
                    "PortBindings": {
                        "8080/tcp": [
                            {
                                "HostIp": "",
                                "HostPort": "7070"
                            }
                        ]
                    },
                    
           ...  
       
  

 - 실행 중인 컨테이너 내부 접근
    
        docker exec -it crane-server bash
        
   - 실행중인 crane-server 컨테이너 내부에 접근.
   - "-i" : 표준 입력으로 명령어를 입력하기 위해 필요하. 
   - "-t" : 터미널로, 없다면 명령어 prompt 가 화면에 나오지 않는다. 
   - 빠져나올 때에는 "exit" 명령어로 빠져나올 수 있다.
  
- 내부에서 컨테이너 탐색
    
        ps aux
  - 처음 위의 명령어를 쳤을때 command not found 였고, 시도하는 모든 명령어마다 command not found 였다.
    Dockerfile 을 수정하고 사용할 수 있었는데 수정한 부분은 다음과 같다. 
    
      **before**
      
            FROM adoptopenjdk/openjdk11:ubi
            RUN mkdir /opt/app
            COPY build/libs/com-workbook-crane-0.0.1-SNAPSHOT.jar /opt/app
            CMD ["java", "-jar", "/opt/app/com-workbook-crane-0.0.1-SNAPSHOT.jar"]
             
      **after**
            
            FROM openjdk:11
            RUN mkdir /opt/app && apt-get update && apt-get -y install netcat && apt-get clean
            COPY build/libs/com-workbook-crane-0.0.1-SNAPSHOT.jar /opt/app
            CMD ["java", "-jar", "/opt/app/com-workbook-crane-0.0.1-SNAPSHOT.jar"]
    
   - 조회 하면 bash, ps aux, 그리고 실행한 app밖에 보이지 않는다. 호스트의 머신의 다른 프로세스는 볼 수 없게 격리 되어있다. 
   - 컨테이너에서 조회한 pid와 호스트 머신에서 조회한 컨테이너 프로세스의 pid가 다른 것을 볼수 있다. (격리 격리). 아래 사진 참조.
   
   **컨테이너 안에서 조회한 프로세스들과 완전한 파일 시스템** 
   ![process_inside_container.png](/assets/images/kubernetes/process_inside_container.png)
   
   ![container_file_system.png](/assets/images/kubernetes/container_file_system.png)   
   **호스트 (로컬머신)에서 조회한 프로세스들과, 호스트에서 실행중인 컨테이너 프로세스** 
   ![process_host.png](/assets/images/kubernetes/process_host.png)
   
   ![container_process_on_host.png](/assets/images/kubernetes/container_process_on_host.png)
 
 
 - 컨테이너 중지와 삭제
 
        docker stop crane-server
        docker rm crane-server
      
     - stop은 컨테이너에 실행중인 메인 프로세스를 중지시키기 때문에, 컨테이너 내부에 실행되고 있는 다른 프로세스가 없어
       중지된다. 중지만 했 때문에 실행중이지 않은 전체 컨테이너를 조회하는 "docker ps -a" 명령어를 쳐보면 아직 살아 있는 것을 볼 수 있다. 기
     - 완전 삭제를 원하는 경우 rm 명령어를 사용하여 컨테너의 모든 내용을 삭제하게 된다. 
     
'5. 이미지 레지스트리에 이미지 푸시  

   - 이 기능은 로컬 컴퓨터에서만 빌드하여 사용 하던 이미지를 다른 컴퓨터에서도 실행할 수 있게 해준다. 
   - 공개된 레지스트리 중 많이 사용되는 것들은 다음과 같다. 
        - 도커 허브 (http://hub.docker.com)
        - Query.io
        - 구글 컨테이너 레지스트리 (Google Container Registry)
        
   - 이미지 푸시 순서
        1. 도커 허브 규칙에 따른 (도커허브를 선택했기 때문에) 이미지 태그 설정하는데 "도커허브아이디"/"이미지이름" 으로 정한다. 
            
                docker tag crane heejeong/crane
              
           태깅을 한 후, "docker images | head" 명령어를 통해 이미지를 확인 할 수 있는데, tag 이름만 다를 뿐 동일한 IMAGE ID를 가지고 있기 때문에 같은 이미지이다. 
           ![image_tag.png](/assets/images/kubernetes/image_tag.png)
          
        2. 도커 허브에 이미지를 푸시한다. 
            
                docker push heejeong/crane 
        - 만일 "requested access to the resource is denied" 메시지가 나오면서 푸시가 되지 않는다면, 로그인을 먼저 진행하고 재시도한다.  
            
                docker login -u heejeong 
            
            도커 허브에 올라간 모습
            ![image_push_docker_hub.png](/assets/images/kubernetes/image_push_docker_hub.png)
            
        3. 다른 머신에서 실행해 보기 
            
                docker run -p 6060:8080 heejeong/crane
            
            ![image_docker_hub_run.png](/assets/images/kubernetes/image_docker_hub_run.png)
  
             
        
-------
관련 서적

쿠버네티스 인 액션 [link]

[link]: https://www.google.com/search?q=%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EC%9D%B8%EC%95%A1%EC%85%98&oq=%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EC%9D%B8%EC%95%A1%EC%85%98&aqs=chrome..69i57.3723j0j7&sourceid=chrome&ie=UTF-8
