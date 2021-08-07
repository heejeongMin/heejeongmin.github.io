---
layout: post
title: API Gateway
categories: cloud
tags: [AWS, CLOUD, SERVERLESS]
---

## Amazon API Gateway 란 ?

---

1. Amazon API Gateway를 이용하면, 개발자가 AWS Lambda 함수와 같은 API를 생성, 게시, 유지 관리, 모니터링을 보호할 수 있다. (외부에서 다이렉트로 리소스에 접근하지 않기 때문)
    - 리소스 정책은 지정된 보안 주체 (보통 IAM 사용자 또는 역할)가 API를 호출할 수 있는지 여부를 제어하기 위해 API에 연결하는 JSON 정책 문서이다. 이런 정책을 API 메서드와 연결항ㄹ 수 있고 다음 두가지 유형의 정책을 지원하여 보호할 수 있다. 
    - API 생성, 배포 및 관리
    -  API의 메서드 호출 및 캐시 새로 고침
    - 백엔드 리소스에 액세스 하기 전에 Amazon Cognito 또는 Lambda 권한 부여자와 통합하여 클라이언트 인증 및 권한 부여 수행할 수 있다. Amazon CloudFront를 보안 계층으로 상용하여 DDos공격으로 부터 보호도 가능하다.

    ![APIGateway_intro](/assets/images/cloud/APIGateway_intro.png)

2. 트래픽 관리, 권한 부여 및 액세스 제어, 모니터링 및 API 버전 관리 등 최대 수십만건의 동시 API호출을 수신 및 처리하는데 관계된 모든 작업을 처리한다. 
- 허용된 요청 속도와 버스트 한도를 바탕으로 다양한 사용량 계획을 생성할 수 있다.
- 사용량 계획은 배포된 하나 이상의 API 단계에 누가 액세스 할 수 있는지 지정하고, 호출자가 API에 얼마나 많이, 도 얼마나 빨리 액세스 할 수 있는지도 지정한다. API 키를 이용하여 API 클라이언트를 식별하고, 개별 클라이언트 API키에 적용되는 구성 가능한 조절 및 할당량 한도를 사용하여 API 단계에 해단 액세스를 측정한다.
- 메시지 변환 및 검증 기능 제공한다. 모델을 생성하여 요청/응답 메시지에 대한 스키마를 정의할 수 있다.

    ![APIGateway_message_convert](/assets/images/cloud/APIGateway_message_convert.png)

 - API Gateway는 서버가 없다. 그러나 서버가 있는 것처럼 보이게 한다.!
      
        @PostMapping("/users") → lamda
        
        @DeleteMapping("/users") → ec2 api

        @PutMapping("/users") → mock
       
      —> 이런 호출이 들어오면 —> 어떤 애를 호출해 줄수 있게 해주는 것이 API Gateway이다. 따라서 로직이 실행되는 곳이 lamda일 수도 있고, Ec2일수도 있고 .. 중요한것은 클라이언트의 입장에서 API Gateway는 응답까지 만들어서 주는 서버처럼 보인다. 

```java
@PostMapping("/users")
public ResponseEntity userRegister(){
	//회원가입
}

@DeleteMapping("/users")
public ResponseEntity userDelete(){
	//탈티
}

@PutMapping("/users")
public ResponseEntity userRevival(){
	//휴면해제
}
```

- 작성/테스트/배포 모두 가능하다. swagger문서처럼 생겼는데 직접 swagger문서를 올릴 수도 있다고 한다.  API gateway도 로그는 cloudwatch에 생성된다.

    ![APIGateway_example](/assets/images/cloud/APIGateway_example.png)

## Amazon Cloud Watch로 로깅

---

- CloudWatch로 로그 메시지 및 세부 지표를 전송할 수 있고 상세 지표는 다음과 같다.
    - API 호출 건수
    - 지연 시간
    - 통합 지연 시간
    - HTTP 400 및 500 오류

    ![APIGateway_logging](/assets/images/cloud/APIGateway_logging.png)

# API Gateway로 개발

---

**순서**  
API 생성 → RESTful 리소스 경로 및 메서드 정의 → API 메서드 테스트 → API 배포 및 구성 → 클라이언트에서 API 호출

### API 생성

- New API(새 API)를 선택하면 API를 처음부터 생성할 수 있다. API가 이미 존재하고, Swagger 정의 파일이 있는 경우, Import from Swagger(Swagger에서 가져오기)를 선택하여 임포트 할 수 있다.
- API 엔드포인트를 클라이언트가 배포하고 액세스 하는 방법은 엔드포인트 유형별로 결정된다.
    1. **리전별** :  현재 리전에 배포됨. 일반적으로 연결 지연시간을 줄임으로 default 설정이다.
    2. **엣지 최적화** :  CloundFront 네트워크에 배포됨. 지리적으로 클라이언트와 더 가까운 CloundFront 배포된다. 
    3. **프라이빗** : VPC 엔드포인트를 사용하여  Amazon Virtual Private Cloud에서만 액세스 할 수 있다.

    ![APIGateway_create](/assets/images/cloud/APIGateway_create.png)

### API의 RESTful 리소스 정의

- 백엔드에 있는 세 가지 유형의 엔드포인트에 액세스 할 수 있다.
    1. AWS Lambda 함수
    2. AWS 서비스
    3. HTTP 엔드포인트, 웹사이트 또는 웹페이지
- 모델은 페이로드의 데이터 구조를 정의하며 JSON 스키마 형식을 사용하여 설며오딘다. 매핑 템플릿을 사용하여, 한 모델에서 다른 모델로 데이터를 변환할 수 있다.
- 권한 부여의 유효값은 다음과 같다.
    1. NONE :  클라이언트 인증 수행하지 않음. 모든 사용자에게 개방
    2. AWS_IAM : IAM권한을 사용하여 메서드에 대한 클라이언트 액세스를 제어한다. 메서드에 액세스할 권한이 있는 IAM역할의 사용자만 메서드를 호출하도록 허용함. 이렇게 하려면,  IAM에서 프로비저닝한대로, Access Key 및 secretKey를 사용하여 서명해야 한다. 

    ![APIGateway_resource_define](/assets/images/cloud/APIGateway_resource_define.png)

    ![APIGateway_method_auth](/assets/images/cloud/APIGateway_method_auth.png)

### API 테스트

 - Amazon 리쇄스 이름(ARN)은 호출되는 통합 엔드포인트의 내부 ID로 엔드포인트 유형에 따라 형식이 다름.

    ![APIGateway_api_test](/assets/images/cloud/APIGateway_api_test.png)

### API 배포

- 기가 바이트 단위로 크기를 지정함으로써  API호출에 캐싱을 추가할 수 있다.
    - 캐싱 용량의 최소값과 최대값은 각각 0.5GB, 237GB이다.
    - 기본 TTL 값은 300초이고, 최대는 3600초이다. 0으로 설정시 캐싱 비활성화된다.

    ![APIGateway_api_deploy](/assets/images/cloud/APIGateway_api_deploy.png)

    ![APIGateway_caching](/assets/images/cloud/APIGateway_caching.png)