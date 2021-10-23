---
layout: post
title: 레플리케이션과 그 밖의 컨트롤러 - 관리되는 파드 배포
subtitle: Kubernetes In Action
categories: kubernetes
tags: [KUBERNETES, CONTAINER, INFRASTRUCTURE]
---

Chapter4. 레플리케이션과 그 밖의 컨트롤러 : 관리되는 파드 배포
---

### 파드를 안정적으로 유지하기
파드가 노드에 스케줄링되는 즉시, 해당 노드의 Kubelet은 파드의 컨테이너를 실행하고 파드가 존재하는 한 컨테이너가 계속 실행되게 된다.

컨테이너의 주 프로세스가 크래시가 발생하면 Kubelet이 컨테이너를 다시 시작하는데 이때 쿠버네티스가 어플리케이션을 다시 실행해 준다. 
하지만 다시 시작되길 원치않는 크래시가 발생하는 경우 (ex. oome) 쿠버네티스가 다시 실행하지 않도록 알려줄 수도 있지만, 

이렇게 자동으로 말고, 외부에서 어플리케이션의 상태를 체크하면 좀더 유연하게 어플리케이션을 다시 시작해야하는지 아닌지를 구별할 수 있다. 

- 라이브니스 프로브 소개 (liveness probe)
    - 컨테이너가 살아 있는지 확인 할 수 있는 기능으로 파드의 스펙에 각 컨테이너의 라이브니스 프로브를 지정할 수 있다. 
    - 쿠버네티스가 주기적으로 프로브를 실행하고 프로브가 실패할 경우 컨테이너를 다시 시작할 수 있다.
    - 라이브니스 메커니즘
        - HTTP GET 프로브  
          > - 지정한 IP주소, 포트, 경로에 HTTP GET 요청을 수행  
          > - 정상 응답으로 (2xx, 3xx) 반환되는 경우 성공했다고 간주
          > - 비정상 응답 혹은 응답이 돌아오지 않으면 실패한 것으로 간주하고 재시작 
        - TCP 소켓 프로브 
          > - 컨테이너의 지정된 포트에 TCP 연결을 시도
          > - 연결 실패 시 프로브 실패로 간주하고 재시작
        - Exec 프로브
          > - 컨테이너 내의 임의의 명령어를 실행하고 종료 상태 확인
          > - 상태코드 0이 성공이며 그 외에 코드는 실패로 간주하고 재시작
                       
    - HTTP 기반 라이브니스 프로브
      ```yaml
        apiVersion: v1
        kind: pod
        metadata: 
          name: crane-liveness
        spec:
          containers:
          - image: heejeong/crane
            name: crane
            livenessProbe:
                httpGet:
                  path: /test
                  port: 8080
      ```
      - 위 yaml 설정에서 spec.containers.httpGet에 라이브니스 프로브를 설정하였는데, 이런 요청은 컨테이너가 실행되는 즉시 실행된다. 
      - 동작 중인 라이브니스 프로브 확인하려면 RESTART 열로 재시작된 횟수를 확인하거나 describe명령어를 사용하여 State를 확인한다. 
        - describe를 사용하면 추가 속성 중 프로브에 관한 속성을 얻을 수 있다.  
          ```
            Liveness : http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
          ```
          - delay : 컨테이너가 시작한 후 얼마만큼의 지연을 주고 시작하는지 의미이다. 초기지연을 주지 않으면 컨테이너가 실행되자 마자
            호출됨으로, 이때 애플리케이션이 요청을 받을 준비가 돼 있지 않을 수 있기 때문에 임곅밧을 초과하기 쉬우니, 초기 지연값을 주는 것이 좋다.
             ** 컨테이너 종료 코드 137, 143은 외부 시그널에 의해 프로세스가 종료되었음을 나타낸다. 
             ```yaml
             livenessProbe:
               httpGet:
                  path: /test
                  port: 8080
               initalDelaySeconds: 15
            ```
          - timeout : 주어진 시간 안에 컨테이너가 응답을 해야한다. 
          - period : 몇초 주기로 프로브를 실행하는지 의미
          - failure : 주어진 횟수만큼 연속 실패하면 컨테이너 재시작
      - 크래시된 컨테이너의 애플리케이션 로그를 얻어야 하는 경우, 이미 재시작이 되었다면 현재 컨테이너의 로그를 표시하기 때문에
        종료된 이유를 파악하려면 다음 명령어를 사용한다. 
           ```
              kubectl logs 파드이름 --previous
           ```    
      - 효과적인 라이브니스 프로브 생성
         - 더 나은 라이브니스 프로브를 위해 특정 URL 경로에 요청하도록 구성하여(ex. /health) 애플리케이션 내의 실행중인 모든 주요 구성 요소가 살아 있는지 응답이 없는지 확인하도록 구성가능하다.
           엔드포인트가  /health인 경우 인증이 필요하지 않은지 꼭 확인해야 한다. 그렇지 않으면 항상 재시작을 하게되는 상황이 올 수 있다. 
         - 라이브니스 프로브는 애플리케이션의 내부만 체크하고 외부 요인에 영향을 받지 않도록 해야한다. 예를 들어 데이터베이스를 연결할 수 없는 경우 
           실패를 반환한다면 근본적인 원인이 데이터베이스에 있는 경우 애플리케이션이 재시작을 해도 문제가 해결 되지 않는다. 
         - 너무 많은 연산 리소스를 사용해서는 안된다. (너무 오래걸리면 안됨) 주로 1초 내에 완료되어야한다. 너무 많은 일을 하는 
           프로브는 컨테이너의 속도를 상당히 느리게 한다. 
         - 프로브는 파드를 호스팅하는 노드의 Kubelet에서 실행된다. 마스터에서 실행중인 쿠버네티스 컨트롤 플레인 구성 요서는 관여하지 않는다. 
           하지만 노드 자체에 문제가 생겨 크래시가 발생한 경우 노드 크래시로 중단된 모드 파드를 생성하야 하는 것은 컨트롤 프레인이 한다. 다만, 
           파드들 중에 직접 수동으로 생성한 파드는 해당 사항이 없으니 이런 경우를 대비하여 레플리케이션 컨트롤러 또는 유사한 매커니즘으로 파드를 생성해야한다. 
           
     
### 레플리케이션 컨트롤러 소개
레플리케이션 컨트롤러는 쿠버네티스 리소스로서 파드가 항상 실행되도록 보장한다. 클러스터에서 노드가 사라지거나 노드에서 파드가 제거된 경우 모두 감지하여 교체생성한다. 
다만 이때도 레플리케이션 컨트롤러가 관리하는 파드만 보장한다. 

- 레플리케이션컨트롤러의 동작
파드가 의도하는 수보다 적은 경우 템플릿 에서 새 복제본을 만들고, 많은 경우 복제본이 제거된다. 
    - 복제본이 더 많이 생성된 경우는 다음과 같을 수 있다.
      1. 같은 유형의 파드를 수동으로 만든경우
      2. 기존 파드의 유형을 변경한 경우 (레이블 셀렉터와 일치하는 파드)
      3. 의도하는 파드 수를 줄인경우   
      
레플리케이션컨트롤러의 세가지 요소 이해  
    1. 레이블 셀렉터는 레플리케이션 컨트롤러의 범위에 있는 파드를 결정한다. 
    2. 레플리카 수는 실행할 파드의 의도하는 수를 지정한다. 
    3. 파드 템플릿은 새로운 파드 레플리카를 만들 때 사용한다. 
    

레플리케이션 컨트롤러의 범위 안팎으로 파드 이동하기
- 파드의 레이블을 변경하면 레플리케이션 컨트롤러의 범위에서 제거되거나 추가될 수 있다. 
- 레이블을 변경함으로써 한 레플리케이션 컨틀로러에서 다른 레플리케이션 컨트롤러로 옮겨 갈 수 있다. (overwrite을 꼭 써야한다.)
  ```yaml
    kubectl label pod crane-dmdck app=crane-server --overwrite
  ```
  
파드 템플릿 변경
- 레플리케이션컨트롤러의 파드 템플릿은 언제든지 수정할 수 있다. 현재 생성되어 있는 파드에는 영향을 주지 않으며 새로 생성되는 파드에만 영향을 준다. 
  기존 파드를 수정하려면 해당 파드를 삭제하고 레플리케이션컨트롤러가 새 템플릿을 기반으로 교체하도록 해야한다. 
  ```yaml
    kubectl edit rc crane
  ```
  - KUBE_EDITOR 환경변수를 설정해 원하는 텍스트 편집기를 사용하도록 kubectl에 지시할 수 있다. (ex. export KUBE_EDITOR = "/usr/bin/nano")
  
  
### 수평 파드 스케일링 
명령어 혹은 yml을 수정하는 두가지 방법이 있을 수 있다. 
yml로 실행하는 경우 두번째 명령어를 통해 sepc.replicas 수를 직접 변경한다. 
```yaml
kubectl scale rc crane --replicas=10

or

kubeclt edit rc crane
```

- 레플리케이션 컨트롤러 삭제  
    - kubectl delete를 통해 레플리케이션 컨트롤러를 삭제하면 파드도 삭제되는데, 이때 옵션을 주어서 
      레플리케이션 컨트롤러만 삭제하고 관리받던 파드들을 그냥 둘 수도 있다. 
      ```
        kubectl delete rc kubia --cascade=false
      ```
      이렇게 관리 받지 않게된 파드들도 원하면 언제든지 레이블 셀렉터를 통해 다른 레플리케이션 컨트롤러에서 관리받게 할 수 있다.
      
 
### 레플리케이션 컨트롤러 대신 레플리카셋 사용하기
초기에는 레플리케이션컨트롤러가 파드를 복제하고 노드 장애가 발생했을 때 재스케줄링하는 유일한 쿠버네티스 구성요였다. 
후에 레플리카셋이라는 유사한 리소스가 도입이되었는데 , 이후 레플리케이션 컨트롤러를 완전히 대체한다. 

일반적으로 리플리카셋을 직접 생성하지는 않고, 상위수준의 디플로이먼트 리소스를 생성할 때 자동으로 생성되게 된다. 

- 리플리카셋과 레플리케이션컨트롤러 비교  
    - 파드 셀렉터
        - 리플리카셋  
           > 리플리카셋의 셀렉터는 특정 레이블이 없는 파드나 레이블의 값과 상관없이 특정 레이블의 키를 갖는 파드를 매칭시킬 수 있다.
          
           > 레이블이 env=production인 파드와 env=dev1인 파드를 동시에 매칭시킬 수 도 있다.
                                                                                                                 
           > 레이블의 값은 상관없이 키만 동일하다면 매칭시키는것도 가능하다. (env=* 처럼 작동)
        - 리플리케이션컨트롤러  
           > 특정 레이블이 있는 파드만 매칭시킬 수 있고 레이블이 같아야한다. 
     
- 리플리카셋 정의하기 
```yaml
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
    name: crane
spec:
    replicas: 3
    selector:
      matchLabels: 
        app: crane
    template:
      metadata:
        labels:
          app: crane
      spec:
        containers:
        - name : crane
          image: heejeong/crane
```
* 레플리카셋은 v1 api의 일부가 아니라는 점이 다른데, API 버전 속성에 대해 알아보면, 
  API 그룹이 core그룹에 속하는 경우, apiVersion은 나열할 필요가 없다. 최신 쿠버네티스 버전에 도입된 다른 리소스는 여러 API 그룹으로 분리되기 때문에 
  API그룹/API버전 모두 지정한다. 
* 파드가 가져야하는 레이블은 selector 속성으로 나열되는 것이 아니라, selector.matchLabels 아래에 지정한다. 

- 레플리카셋 생성 및 검사 (rs : replicaset)
    ```yaml
      kubectl get rs 
    ```
  
- 리플리카셋의 더욱 표현적인 레이블 셀렉터 사용하기 
   ```
      selector:
        matchExpressions:
            - key: app
              operator: In
              values: 
                - crane
   ```
  사용할 수 있는 operator 
   1. In : 지정된 값 중 하나와 일치  
   2. NotIn : 레이블값이 지정된 값과 일치하지 않아야한다. 
   3. Exists : 파드는 지정된 키를 가진 레이블이 포함되어야하다. (값은 중요하지 않음으로 value를 명시하지 않아야한다.)
   4. DoesNotExists : 파드에 지정된 키를 가진 레이블이 포함돼 있지 않아야한다. (값은 중요하지 않음으로 value를 명시하지 않아야한다.)
   
- 리플리카셋 삭제하기
    ```yaml
       kubectl delete rs crane
    ```
  
### 데몬셋을 사용해 각 노드에 정확히 한개의 파드 실행하기
레플리케이션 컨트롤러와 레플리카셋은 쿠버네티스 클러스터 내 어딘가에 지정된 수만큼의 파드를 실행하는데 사용된다. 
그러나 클러스트이 모든 노드에 한개씩의 노드만 실행되어야 하는 경우가 있을 것이다. (인프라 관련 컨테이너인 경우)


- 데몬셋으로 모든 노드에 파드 실행하기
    - 모든 클러스터 노드마다 파드를 하나만 실행하려면 데몬셋 오브젝트를 생성해야 한다. 데몬셋에 의해 생성되는 파드는 
타깃 노드가 이미 지정되어 있고, 쿠버네티스 스케줄러를 건너뛰는 것을 제외하면 이 오브젝트는 레플리케이션 컨트롤러 또는 
레플리카셋과 매우 유사하다. 
    - 레플리카셋은 클러스터에 원하는 수의 파드 복제본이 존재하는지 확인하는 반면 데몬셋에는 원하는 복제본 수라는 개념이 없다. 
      파드 셀렉터와 일치하는 파드 하나가 각 노드에서 실행중인지 확인하는 것이 데몬셋이 수행해야하는 역할이기 때문에 복제본 개념이 필요하지 않다. 
      
    - 새로운 노드가 생성이되면 즉시 새로운 파드 인스턴스를 배포하고, 실수로 파드 중 하나를 삭제해 데몬셋의 파드가 없는 경우도 새로 생성한다. 
   
- 데몬셋으로 특정 노드에 파드 실행하기
```yaml
apiVersion: apps/v1beta2
kind: DemonSet
metadata:
    name: ssd-monitor
spec:
   selector:
      matchLabels: 
        app: ssd-monitor
    template:
      metadata:
        labels:
          app: ssd-monitor
      spec:
        nodeSelector:
          disk: ssd
        containers:
        - name : crane
          image: luksa/ssd-monitor
```
 - disk:ssd 레이블을 가지고 있는 노드에 luksa/ssd-monitor 컨테이너 이미지 기반으로 컨테이너 한 개를 갖는 데몬셋을 의미. 
 - 생성된 데몬셋을 보는 법
  ```yaml
    kubectl get ds
  ```
 - 필요한 레이블을 노드에 추가하기
 ```yaml
    kubectl label node minikube disk=ssd
 ```
 - 노드에서 레이블 제거하기
 ```yaml
  kubectl label node minikube disk=hdd --overwrite 
 ```
   데몬셋에 의해 노드에 이미 파드가 배포된후 노드의 레이블을 변경하게 되면, 파드가 종료되기 시작한다. (데몬셋을 삭제해도 파드가 삭제된다.)
   
### 완료 가능한 단일 태스크를 수행하는 파드 실행
- 잡 리소스 소개
쿠버네티스 잡 리소스를 사용하면 완료 가능한 태스크에서는 프로세스가 종료 된 후에 다시 시작되지 않는다. 프로세스 자체에 장애가 발생한 경우 
잡에서 컨테이너를 다시 시작할 것 인지 설정할 수 있다. 
```yaml
apiVersion: batch/v1
kind: Job
metadata:
    name: batch-job
spec:
    template:
      metadata:
        labels:
          app: batch-job
      spec:
        restartPolicy: OnFailure
      containers:
      - name: main
        image: luksa/batch-job
```
* 잡은 batch API 그룹과 v1 API 버전에 속한다. 
* restartPolicy는 default가 Always이지만, onFailure나 Never를 명시적으로 설정해야 종료될때 재시작되지 않도록 해야한다. 

- 파드를 실행한 잡 보기
    ```yaml
      kubectl get jobs
    ```
    해당 배치가 2분 후 종료되는 배치라고 하면, 2분후에는 더이상 파드가 조회되지 않고, -a 옵션 혹은 --show-all 을 사용하여 
    kubectl get po -a를 실행하면 STATUS가 Completed로 뜬다.
    
- 잡에서 여러 파드 인스턴스 실행하기
잡은 두개 이상의 파드 인스턴스를 생성해 병렬 또는 순차적으로 실행할 수 있도록 구성할 수 있다. 이는 스펙에 completions, parallelism 속성을 설정해 수행한다. 
    - 순차적으로 잡 파드 실행하기
    ```yaml     
      apiVersion: batch/v1
      kind: Job
      metadata:
          name: multi-completion-batch-job
      spec:
          completions: 5
          template:
            metadata:
              labels:
                app: batch-job
            spec:
              restartPolicy: OnFailure
            containers:
            - name: main
              image: luksa/batch-job
      
    ```
    completions 5라고 해놓으면 순차적으로 5개의 파드를 실행한다. 처음에 파들르 하나 만들고 완료하면 두번재 파드를 만들어 실행한다.   
    파드 중 하나가 실패하면 잡이 새 파드를 생성하므로 잡이 전체적으로 다섯개 이상의 파드를 생성할 수 있다.
    
    - 병렬로 잡 파드 실행하기
     ```yaml
          apiVersion: batch/v1
          kind: Job
          metadata:
              name: multi-completion-batch-job
          spec:
              completions: 5
              parallelism: 2
              template:
                metadata:
                  labels:
                    app: batch-job
                spec:
                  restartPolicy: OnFailure
                containers:
                - name: main
                  image: luksa/batch-job
          
    ```
    잡이 실행되는 동안 잡의 parallelism 속성을 변경할 수도 있다. 동일하게 scale명령어를 사용한다. 
    ```yaml
      kubectl scale job multi-completion-batch-job --replicas 3
    ```
  - 잡 파드가 완료되는데 걸리는 시간 제한하기
    파드가 특정 상태에 빠져서 완료되지 못하는 경우에는 activeDeadlineSeconds속성을 사용하여 파드의 실행 시간을 제한할 수 있다.
    잡의 매니페스트에서 spec.backoffLimit 필드를 지정해 실패한 것으로 표시되기전에 잡을 재시도 할 수 있는 횟수를 정할 수도 있는데, 
    정하지 않으면 default는 6이다. 
    
  - 잡을 주기적으로 또는 한번 실행되도록 스케줄링 하기 
    ```yaml
     apiVersion: batch/vbeta1
     kind: CronJob
     metadata:
         name: batch-job-every-fifteen-minutes
     spec:
         schedule: "0,15,30,45 * * * *"
         jobTemplate:
          spec:
             template:
               metadata:
                 labels:
                   app: periodic-batch-job
               spec:
                 restartPolicy: OnFailure
               containers:
               - name: main
                 image: luksa/batch-job
     
    ```
    스케줄은 분 시 일 월 요일로 이루어진다.  
    잡이나 파드가 상대적으로 늦게 생성되고 실행될 수 있는데, 이때 특정 시작을 지나서 시작되면 안된다는 엄격한 요구사항이 있는경우
    startingDeadlineSeconds 필드를 지정해 데드라인을 정할 수도 있다.  
    ```yaml
         apiVersion: batch/vbeta1
         kind: CronJob
         metadata:
             name: batch-job-every-fifteen-minutes
         spec:
             schedule: "0,15,30,45 * * * *"
             startingDeadlineSeconds: 15
    
    ```          
        
-------
관련 서적

쿠버네티스 인 액션 [link]

[link]: https://www.google.com/search?q=%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EC%9D%B8%EC%95%A1%EC%85%98&oq=%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EC%9D%B8%EC%95%A1%EC%85%98&aqs=chrome..69i57.3723j0j7&sourceid=chrome&ie=UTF-8
