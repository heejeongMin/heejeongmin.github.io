---
layout: post
title: Kubernetes intro
subtitle: Kubernetes In Action
categories: kubernetes
tags: [KUBERNETES, DOCKER, CONTAINER, INFRASTRUCTURE]
---

              
### 쿠버네티스 클러스터 설치

- 이 책에서는 로컬머신에 단일 노드 쿠버네티스 실행하는 방법과, 구글 쿠버네티스 엔진에 실행중인 클러스터에 접근하는 방법을 다룬다.

#### Minikube를 활용한 단일 노드 쿠버네티스 클러스터 실행
 - Minikube는 로컬에서 쿠버네티스를 테스트하고 애플리케이션을 개발하는 목적으로 설치를 도와주는 도구이다. 
 
 1. Minikube  설치  
   a. http://github.com/kubernetes/minikube 의 문서를 따라 설치
      나는 homebrew 로 설치하는 방식을 선택함. 
   b. 설치 후 쿠버네티스 클러스터를 바로 시작할 수 있다. 
        
            minikube start
 
   c. kubectl은 설치가 되어 있어서 스킵 (언제했는지 기억이 안남) 쿠버네티스를 다루기 위한 CLI 클라이언트임. 
      
   d. 시작한 cluster가 잘 작동하는지 다음 명령어로 확인할 수 있다. 
   
         kubectl cluster-info
         
   ![kubectl_cluster_info.png](/assets/images/kubernetes/ch02/kubectl_cluster_info.png) 
   
 '2. 클러스의 개념 이해하기
     - 각 노드는 도커, Kubelete, kube-proxy를 실행한다. 
     - Kubectl 클라이언트 명령어는 마스터 노드에서 실행중인 쿠버네티스 API 서버로 REST 요청을 보내 클러스트와 상호작용한다. 
   
   **노드가 세개인 쿠버네티스 클러스터와의 상호작용**  
   ![kubernetes_cluster.png](/assets/images/kubernetes/ch02/kubernetes_cluster.png) 
   
   | 명령어 | 용도 | 추가설명 |
   | :-------------- | :----------- | :-------------- |
   | kubectl get nodes | 클러스터안의 노드 조회 | 오브젝트의 기본 정보만 표시 (Name, Status, Age, Version) |
   | kubectl describe node [노드이름] | 오브젝트의 상세 정보 (Cpu, 메모리, 시스템정보 ...) | node의 이름을 명시하지 않으면 전체 조회된다. |
   | kubectl describe pod [파이름] | 오브젝트의 상세 정보 (Cpu, 메모리, 시스템정보 ...) | node의 이름을 명시하지 않으면 전체 조회된다. |

  - kubectl alias와 명령줄 자동완성
    1. 나는 맥에서 zsh를 사용하니 ~/.zshrc 를 열어 다음 alias를 추가하고 저장한 후 "source ~/.zshrc" 를 실행하여 준다.  
       적용하고 나면, 앞으로 "k describe nodes" 이런식으로 사용 가능하다. 
            
            alias k=kubectl
     
            
'3. 쿠버네티스에서 애플리케이션 구동하기  
    1. kubectl run : JSON, YAML을 사용하지 않고 필요한 모든 구성 요소를 생성하는 방법
            
           kubectl run crane --image=heejeong/crane --port=7070
       
   ![kubectl_run_app.png](/assets/images/kubernetes/ch02/kubectl_run_app.png) 
        - "--generator=run/v1"은 deprecated되었기 때문에, 향후 kubectl run --generator=run-pod/v1 또는 kubectl create를 사용하는 것이 좋다.  
        - 책에서는 repcliationController 라고 생성되는데 나는 pod 로 생성된다. 왜?
        
   ![kubectl_get_pods_ImagePullBackOff.png](/assets/images/kubernetes/ch02/kubectl_get_pods_ImagePullBackOff.png)
        - 파드의 상태가 ImagePullBackOff인 경우는 이미지를 가져오지 못하여서 간격을 두고 계속 pull을 시도하는 상태이다. 
          처음 생성시 이미지에 "crane"을 사용하였더니, (로컬에서 docker build -t crane . 으로 생성한 이미지) 이미지를 가져오지 못하였고, 
          도커 허브에 올라간 이미지로 하니 정상 생성되었다. ("heejeong/crane")
          -> 이유는, 로컬에서 빌드한 이미지는 로컬에서만 사용할 수 있기 때문에 도커 데몬이 실행중인 다른 워크 노드에서 컨테이너 이미지를 접근하게 하고자하여 도커 허브에 올라간 이미지를 사용한다.
        - CrashLoopBackOff               
   - 파드  
       - 쿠버네티스는 개별 컨테이너를 직접 다루지 않는다. 대신 함께 배치된 다수의 컨테이너라는 개념을 사용하고, 이 컨테이너 그룹을 파드(Pod)라고 한다.
       - 파드는 하나 이상의 밀접하게 연관된 컨테이너의 그룹으로 같은 워커 노드에서 같은 리눅스 네임스페이스로 함께 실행된다. 
       - 각 파드는 자체 IP, 호스트 이름, 프로세스 등이 있는 논리적을 분리된 머신이며, 애플리케이션 프로세스를 실행하는 하나 이상의 컨테이너를 갖는다.
       - 파드에서 실행 중인 모든 컨테이너는 동일한 논리적인 머신에서 실행하는 것처럼 보이지만, 다른 파드에서 실행중인 컨테이너는 같은 워크노드에서 실행중이라도 다른 머신에서 실행중인 것으로 나타난다. 
       - 파드 라이프 사이클 https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
          ![pod_lifecycle.png](/assets/images/kubernetes/ch02/pod_lifecycle.png)

  - 위의 파드까지 띄우는 과정은 다음과 같다. 
        ![process_to_run_pod.png](/assets/images/kubernetes/ch02/process_to_run_pod.png)
        
'4. 실행중인 파드에 어떻게 접근할 수 있을까?   
   - 각 파드는 자체 IP주소를 가지고 있지만 이 주소는 클러스터 내부에 있으며 외부에서 접근이 불가능하다. 
      외부에서 접근이 가능하게 하려면 서비스 오브젝트를 통해 노출해야 한다. 파드 뿐만 아니라 다른 서비스들도 마찬가지인데
      외부에 노출 시키기 위해서는 LoadBalancer 유형의 특별한 서비스를 생성해야 한다.   
   - LoadBalancer 유형의 서비스를 생성하면 외부 로드 밸런서가 생성되므로 로드 밸런서의 퍼블릭 IP를 통해 파드에 연결할 수 있다.     
   - 서비스 오브젝트 생성하기  
            
            kubectl expose po crane --type=LoadBalancer --name crane-http

        - po는 pod를 의미하는 약어인데, 대부분의 리소스 유형은 약어를 가지고 있다고 한다.  
                - service : svc
                - replicationcontroller : rc
            ![kubectl_expose.png](/assets/images/kubernetes/ch02/kubectl_expose.png)  
       
       - 서비스 조회하기               
                
                kubectl get services  
            위의 스크린샷에서 외부 서비스 IP가 아직 pending인건 로드 밸런서를 생성하는데 시간이 걸리기 때문이라서 기다렸다 조회하면 생성된다고 한다. 
            단, minikube로 하는 경우 터널링을 켜주어야 external ip가 생성된다.  
                           
                           
                minikube tunnel     
   <<LoadBalancer를 생성하면서 만난 오류 : **Recv failure: Connection reset by peer**>>      
       
   - 처음 로드밸런서를 생성할때 포트를 따로 지정해 주지 않았는데, 그럼 pod 설정을 따라가는지 7070으로 생성되었다. 
         생성된 external-ip 로 접근시도하였지만 "Connection reset by peer" 메세지가 나왔다. 
   - 도움 받은 글 : https://github.com/kubernetes/minikube/issues/9482  
     로드밸런서의 포트를 8080으로 생성하지 않으면 이런 현상이 있다고 한다. 생성된 서비스를 "kubectl delete service crane-http" 명령어로 
     삭제하고 8080으로 다시 띄우니 파드내 어플리케이션 접속 성공     
                        
           kubectl expose po crane --type=LoadBalancer --port=8080 --name crane-http
    
    
- 파드와 컨테이너의 이해  
    - 시스템의 가장 중요한 구성 요소
    - 실습에서는 파드가 하나의 컨테이너를 가지고 있지만 보통 파드는 원하는 만큼의 컨테이너를 포함할 수 있다. 
    - 컨테이너 내부에는 crane 서비스의 프로세스가 있고, 포트 7070에 바인딩되어 요청을 기다리고 있다. 
    - 자체의 고유 사설 IP주소와 호스트 이름을 같는다. 
    
- 서비스가 필요한 이유
    - 서비스가 필요한 이유를 알려면 파드의 주요 특성을 알아야한다. 파드는 일시적이기 때문에 언제든지 사라질 수 있다.  
          새로운 파드는 다른 IP주소를 할당 받는다. 이렇게 파드의 IP가 변경되는 문제와, 여러개의 파드를 단일 IP와 포트의 쌍으로 
          연결하기 위해 서비스가 필요하다.   
    - 서비스는 어느 파드가 어디에 존재하는지, 어떤 IP를 갖는지 상관없이 파드 중 하나로 연결하여 요청을 처리한다. 
          서비스는 동일한 서비스를 제공하는 하나 이상의 파드 그룹의 정적 위치를 나타낸다. 
            
- 레플리케이션 컨트롤러 역할의 이해 
    - 위의 실습처럼 생성하면 replication controller가 생성되는 것이 아니라 바로 pod가 생성된다. 
          replication controller를 생성하려면 yml 혹은 json 파일을 만들어야 한다. 
          
        1. json 파일 (파일명은 내가 임의로 replication-controller.json으로 생성하였다.)  
        (https://jamesdefabia.github.io/docs/user-guide/replication-controller/operations/)  
        
            ```$json
            {
              "kind": "ReplicationController",
              "apiVersion": "v1",
              "metadata": {
                "name": "crane-controller"
              },
              "spec": {
                "replicas": 1,
                "template": {
                  "metadata": {
                    "lables": {
                      "app": "crane"
                    }
                  },
                  "spec": {
                    "volumes": null,
                    "containers": [
                      {
                        "name": "crane",
                        "image": "heejeong/crane",
                        "imagePullPolicy": "Always",
                        "ports": [
                          {
                            "containerPort": 7070
                          }
                        ]
                      }
                    ]
                  }
                }
              }
            }        
            ``` 
            
        2. kubernetes에게 json 파일을 전달하여 replication controller를 생성한다.  
        
            ``` 
             kubectl create -f 
            ``` 
                
            ![kubectl_create_rc.png](/assets/images/kubernetes/ch02/kubectl_create_rc.png) 
    
        3. replication controller가 생성되었는지 확인한다.  
            - rc는 replication controller의 약자
            - 생성된 replication controller는 metadata.name에서 정의한 이름을 따라간다.
                ``` 
                 kubectl get rc  
                ```             
        4. replication controller가 관리하는 파드는 자동으로 생성된다.  
            이름은 metadata.name을 따라가지만 식별 '-' 후 다섯자리가 더 붙는다자.
             ``` 
              kubectl get pods  
             ```
            ![kubectl_get_pods.png](/assets/images/kubernetes/ch02/kubectl_get_pods.png) 
     
        5. 이제 replication controller에 대해서 loadbalancer를 생성한다. (minikube tunnel 도 열기)
            
            ``` 
              minikube tunnel
           
              kubectl expose rc crane --type=LoadBalancer --name crane-http  
            ```
       
       - 구성과 레플리케이션 컨트롤러의 역할
          ![rc_po_svc.png](/assets/images/kubernetes/ch02/rc_po_svc.png) 
                - 항상 정확히 하나의 파드 인스턴스를 실행하도록 지정되어 있기 때문에 (replication-controller.json) 
                  파드를 강제로 삭제하면 다시 자동으로 하나 더 만든다. 즉, 파드를 복제하고 항상 실행 상태로 만드는 역학을 한다. 
                  
      
- 애플리케이션 수평 확장
   - 쿠버네티스를 사용하는 주요 이점 중 하나는 간단하게 배포를 확장할 수 있따는 점이다. 다음은 실행중인 인스턴스를 세개로 증가시키는 것이다.  
        - DESIRED : 레플리케이션 컨트롤러가 유지해야하는 파드의 개수
        - CURRENT : 현재 실행 중인 파드의 개수
    
    
        kubectl get rc  
        
      
   - 의도하는 레플리카 수 늘리기 
        - 파드의 레클리카 수를 늘리려면 레플리카 컨트롤러에서 의도하는 레플리카 수를 변경해야 한다. (DESIRED 값 변경)
         
         
                   kubectl scale rc crane --replicas=3

   ![kubectl_scale.png](/assets/images/kubernetes/ch02/kubectl_scale.png)  
   
   ![kubectl_scale_result.png](/assets/images/kubernetes/ch02/kubectl_scale_result.png) 
    
   - 쿠버네티스에게 파드 세개를 항상 유지 해야한다는 것을 알려준다. 어떻게 변경해야하는 지는 알려주지 않는데, 이것이 쿠버네티스의 
          기본 원칙 중 하나이다. 쿠버네티스가 어떤 액션을 수행해야하는지 정확하게 알려주는 대신 시스템의 의도하는 상태 (desired state)를 
          선언적으로 변경하면 쿠버네티스가 실제 현재 상태를 (current state)를 검사해 의도한 상태로 조정한다. (reconcile)
      
   - 서비스 호출 시 모든 파드가 요청을 받는지 확인
        ![kubectl_all_pod_requests.png](/assets/images/kubernetes/ch02/kubectl_all_pod_requests.png) 
         
      - 요청을 보낼때 마다 다른 pod에서 요청이 처리되는데, 하나 이상의 파드가 서비스 뒤에 존재할때 서비스가 로드밸런서 역할을 한다. 
          
   - 애플리케이션이 실행중인 노드 검사하기 
        - 어떤 노드에 파드가 실행중인지 중요하지 않지만 어떤 노드에 스케줄링이 되었는지 궁금할 수 있다. 
        - 파드가 스케줄링된 노드와 상관없이 컨테이너 내부에 실행 중인 모든 애플리케이션은 동일한 유형의 운영체제 환경을 갖는다. 
        - 각 파드는 요청된 만큼의 컴퓨팅 리소스를 제공받는다. 
        - 파드를 조회할 때 파드 IP와 실행중인 노드 표시하기
          ```
            kubectl get pods -o wide
          ```      
        ![kubectl-get-pods-wide.png](/assets/images/kubernetes/ch02/kubectl-get-pods-wide.png) 
          
        - 파드 세부 정보 살펴보기
            ```
              kubectl describe pod crane-9b2q7
            ```         
    
        ```
            Name:         crane-9b2q7
            Namespace:    default
            Priority:     0
            Node:         minikube/192.168.49.2
            Start Time:   Mon, 04 Oct 2021 21:21:49 +0900
            Labels:       app=crane
            Annotations:  <none>
            Status:       Running
            IP:           172.17.0.4
            IPs:
              IP:           172.17.0.4
            Controlled By:  ReplicationController/crane
            Containers:
              crane:
                Container ID:   docker://4068e2e21db3ee048aa9f1c18e2e3f6067a2523e06f0bfd995dcc94896891323
                Image:          heejeong/crane
                Image ID:       docker-pullable://heejeong/crane@sha256:90536c360799a5097b84f685fa7bbb9d837e658cb54404684abd1143bbd0deca
                Port:           7070/TCP
                Host Port:      0/TCP
                State:          Running
                  Started:      Mon, 04 Oct 2021 21:21:56 +0900
                Ready:          True
                Restart Count:  0
                Environment:    <none>
                Mounts:
                  /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hdfrp (ro)
            Conditions:
              Type              Status
              Initialized       True 
              Ready             True 
              ContainersReady   True 
              PodScheduled      True 
            Volumes:
              kube-api-access-hdfrp:
                Type:                    Projected (a volume that contains injected data from multiple sources)
                TokenExpirationSeconds:  3607
                ConfigMapName:           kube-root-ca.crt
                ConfigMapOptional:       <nil>
                DownwardAPI:             true
            QoS Class:                   BestEffort
            Node-Selectors:              <none>
            Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                         node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
            Events:
              Type    Reason     Age    From               Message
              ----    ------     ----   ----               -------
              Normal  Scheduled  7m11s  default-scheduler  Successfully assigned default/crane-9b2q7 to minikube
              Normal  Pulling    7m10s  kubelet            Pulling image "heejeong/crane"
              Normal  Pulled     7m5s   kubelet            Successfully pulled image "heejeong/crane" in 5.117939s
              Normal  Created    7m5s   kubelet            Created container crane
              Normal  Started    7m4s   kubelet            Started container crane
        ```
    
   - 미니쿠베 대시보드
      ```
        minkube dashboard
      ```    
   ![minikube_dashboard.png](/assets/images/kubernetes/ch02/minikube_dashboard.png) 
   ![minikube_dashboard_web.png](/assets/images/kubernetes/ch02/minikube_dashboard_web.png) 
       
             
        
-------
관련 서적

쿠버네티스 인 액션 [link]

[link]: https://www.google.com/search?q=%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EC%9D%B8%EC%95%A1%EC%85%98&oq=%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EC%9D%B8%EC%95%A1%EC%85%98&aqs=chrome..69i57.3723j0j7&sourceid=chrome&ie=UTF-8
