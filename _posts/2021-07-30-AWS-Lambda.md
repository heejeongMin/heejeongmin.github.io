---
layout: post
title: Lambda
categories: cloud
tags: [AWS, CLOUD, SERVERLESS]
---

## 람다란 ?

---

1. 서버를 프로비저닝하거나 관리할 필요 없이 코드를 실행할 수 있는 서버리스 컴퓨팅 서비스
2. 다른 AWS 서비스에서 코드를 자동으로 트리거하도록 설정하거나, 웹 또는 모바일 앱에서 직접 코드를     호출해서 사용할 수 도 있음. 
3. 사용한 컴퓨팅 시간에 대해서만 비용을 지불함으로, 코드가 실행되지 않을 때에는 요금이 부과되지 않음
  - 람다의 모든 호출의 실행시간은 15분으로 제한된다.
  - 비용은 호출 건 마다 100ms 단위로 측정하여 부과된다.
4. 단순한 리소스 모델로 128MB ~ 10GB의 용량(메모리) 선택이 가능하며, 메모리량에 따라 나머지 설정을 비례하게 된다. 
5. 서비스 리밋을(동시성 처리) 변경하지 않으면 default 1,000으로, 람다의 요청이 밀린다고 하면, 서비스 리밋을 확인해야합니다. 증가는 대략 최대 999,999 까지 가능하다고 합니다.

![Lambda_what_is_Lamda](/assets/images/cloud/Lambda_what_is_Lamda.png)

## 람다의 작동방식

---

### 람다 함수의 구성

람다의 호출 포맷은 어떤 언어로 개발하여도 아래 형식을 따른다. 
``` java
handler(event, context)
```

- handler  
    실행 진입점
    
- event         
    호출한 event 및 처리에 필요한 정보 포함
    
- context       
    람다의 실행 컨텍스트로, 람다 호출 이후 Req ID가 발생하고, 실행 런타임 정보를 가지고 있다. 
    

### 람다 실행 시점

---

1. 직접 호출 (요청 - 응답 모델)
2. 이벤트 소스 (푸시 모델)
3. 스트림 기반 (풀 모델)

* 람다를 트리거 할 수 있는 서비스들
https://docs.aws.amazon.com/lambda/latest/dg/lambda-services.html

#### 1. 직접 호출 (요청 - 응답 모델)

- 직접 호출 = 요청-응답 모델 = 동기식 실행
- HTTP 요청인 경우 응답을 받아가기 때문에 Amazon API Gateway 가 람다를 invoke 해줄 때 실행이 되게 됩니다.
- 실행 순서  
    1. 클라이언트가 Amazon API Gateway를 통해 URL을 호출  
    2. 해당 URL이 실행되면서 Lamda를 트리거함     
    3. IAM에서 API Gateway가 람다를 호출 할 수 있는 권한이 있는지 확인
    4. 권한이 있다면 Lambda 로직 실행 후 응답 반환  
   
![Lambda_sync_call](/assets/images/cloud/Lambda_sync_call.png)    
       
#### 2. 이벤트 소스 (푸시 모델)

- 이벤트 소스 = 푸시 모델 = 비동기식 실행 
- 자기 자신의 로직을 처리하는 기능이 없는 SNS, S3의 경우 응답을 받아가지 않기때문에 비동기로 실행이 가능 합니다.
- 실행 순서
    1. AWS 서비스에서 이벤트를 Lambda로 푸시
    2. IAM에서 해당 서비스가 람다를 트리거 할 수 있는 권한이 있는지 확인
    3. Lambda가 이벤트 처리
    
![Lambda_async_call](/assets/images/cloud/Lambda_async_call.png)    


#### 3. 스트림 기반 (풀 모델)

- 스트림 기반 = 풀 모델
- DynamoDB 자체에는 트리거 기능이 없지만, DynamoDB Stream이 변경 내역을 담고 있고, Lambda가 Stream에게 변경사항이 있는지 물어보는  형태의 풀 이벤트 모델 입니다. 
- 실행순서 
    1. Lambda가 이벤트 소스를 폴링
    2. 이벤트가 감지되면 가져와서 처리
    
![Lambda_stream_call](/assets/images/cloud/Lambda_stream_call.png)    


#### IAM에서 권한확인 

![Lambda_iam](/assets/images/cloud/Lambda_iam.png)    

1. AWS 서비스에서 이벤트를 Lambda로 푸시
2. IAM에서 해당 AWS 서비스가 Lambda에  접근 가능한지 확인
3. Lambda가 이벤트 처리   
    a. 처리하면서 람다가 타 서비스에 접근시 (그림상 S3)에도 람다가 그러한 권한이 있는지 확인
    b. 람다의 로그는 클라우드 워치에 남기때문에 로깅할 수 있는 권한 있는지 basic role확인
   
   
#### 4. 람다의 버저닝 
- 하나의 리소스에 버전/별칭을 붙여서 실행하고 싶은 것을 실행할 수 있다. 따로 붙이지 않으면 latest로 붙습니다.  
- 각 버전 또는 별칭에 대해 각각 Amazon 리소스 이름이 지정됩니다. 
- 게시되지 않은 버전의 함수에서만 함수 코드와 설정을 변경할 수 있습니다.  

![Lambda_versioning](/assets/images/cloud/Lambda_versioning.png)    


## 람다의 로깅

---

- EC2를 사용하는 경우 로그를 보고 싶으면 서버 안으로 들어가서 보면 되는데, Lambda의 경우 서버가 없기 때문에 할당되어 있는 저장 공간이 기본적으로 존재하지 않습니다.
    - 512mb의 /tmp 라는 공간이 존재하긴 하지만, Lambda 자체는 어떤 상태를 계속 가지고 가지 않기 때문에, /tmp에 일시적으로 저장은 가능하지만, 다음 호출 때 그 정보가 /tmp 안에 있다는 보장이 없습니다.
    - Lambda는 전용 인스턴스가 N개가 발생하는데, 호출이 줄어들면 인스턴스의 개수 또한 줄어든다. 발생된 N개의 인스턴스 중 호출 시 어떤 인스턴스에서 실행 될 지 모르게 때문에, /tmp의 사용이 필요한 경우 일시적인 로직을 처리하기 위해 사용하고 그 외 상태를 저장하는 용도로 사용하지 않습니다.
- Lambda의 로그는 CloudWatch에 쌓이게 되고, Cloudwatch Metric을 통해 확인 가능합니다. 

