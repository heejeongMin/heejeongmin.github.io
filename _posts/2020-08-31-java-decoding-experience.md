---
layout: post
title: 디코딩 경험 (sun.misc.Base64Encoder와 java.util.BASE64 및 디코딩 한글깨짐)
categories: JAVA
tags: [JAVA]
---


## sun.misc.BASE64Encoder의 문제

첫 번째 작업은 회원 인증강화 작업이었는데, 인증파트에서 암호화된 값을 넘겨주면 나는 그것을 디코딩해서 사용하는 작업이었다. 암호화된 값은 {key:value, key:value} 로 되어 있었고, 이 값이 처음에는 base64 인코딩 후 Seed암호화를 한번 더 거친 값이었다. 따라서 나는 반대로 SEED로 복호화를 먼저 하고 BASE64 디코딩을 하면 되는 작업이었는데, 문제는 BASE64로 디코딩을 했을때였다. 

결론 부터 말하지면, BASE64로 인코딩을 하는 쪽에서 SUN 패키지를 사용했기 때문에 발생하였는데, 현상은 이랬다. 


SEED로 복호화를 하면 아래와 같이 나왔고, 

            {"키":"BASE64 암호화된 값", "키":"BASE64 암호화된 값"} 


다시 BASE64로 복호화를 하면 아래와 같이 비정상 복호화가되었고, 실제 로그를 끝에 찍으면 빈문자열이 있었다.

            {"키":"값", "키":"값 

혹은

            {"키":"값", "키":"값"

​

우선 환경적인 요소로는 나는 JDK7 을 사용하였고, 인증쪽에서는 JDK8을 사용하였다. 

BASE64의 표준 자바 라이브러리는 JAVA8부터 지원이 되기 때문에 나는 우선 JDK7에서 우회할 수 있는 방법을 인터넷에서 모두 찾아 시도를 해보았지만 결과는 비정상 출력이 동일하였다. 

운영 반영일자는 다가오고 인증파트와 이야기하여도 다른 방법이 나타나지 않자 당시 팀장님도 고민하다가 결국 나를 데리고 인증파트로 가서 실제 인코딩이 어떻게 되었는지를 확인하셨다. 

확인한 결과 BASE64 인코딩을 sun.misc.BASE64Encoder를 사용해서 작업이 되었는데 팀장님은 해당 패키지로 인코딩을 했을 때 이슈가 있는 것으로 알고 있으나 정확한 것은 찾아보기로 하였고, 인터넷 검색결과 아래 링크를 찾았고 설명은 다음과 같았다. 

간혹 암호화된 문자열의 마지막 라인에 정확히 76개의 문자열인 경우 개행이 발생 할 수 있다는 버그란다. JAVA8의 java.util.Base64를 만든 사람이 sun.misc.BASE64Encoder의 버그라고 확인했다고 한다. 

복호화는 해야하는데, JDK8에서 제공하는 표준 라이브러리 BASE64를 활용한 인증파트에서는 해당 암호화된 문자열을 복호화하면 잘 되는 것으로 보아, 내쪽에서도 같은 라이브러리를 사용한다면 해결이 되는 문제였다. 

현 프로젝트가 Java7이라 라이브러리를 import해서 사용할 수 없으니, Java8에 있는 Base64의 코드를 그대로 들고와서 우리 프로젝트 안에 넣어서 클래스로 활용을 하는 방법을 사용하였고 .. 풀리지 않았던 복호화가 제대로 되었다. 

![Java_decoding](/assets/images/java/Java_decoding.png)


https://stackoverflow.com/questions/35301409/is-java-8-java-util-base64-a-drop-in-replacement-for-sun-misc-base64

## default 캐릭터 셋으로 인한 복호화 오류​

복호화 관련하여 문제가 있었던 적이 있는데, 이때는 한글이 포함되어 있었고, 파트너 사에서 예약자의 한글 이름을 암호화해서 보내주었다. 
복호화하니 한글이 깨졌고, 이 값을 getByte해와서 모든 characterset 경우의 수를 다 컨버팅 해봤지만 소용이 없었다. 
문제는 파트너사에서는​ 해당값을 UTF-8로 암호화 하였고, 우리 쪽에서 그 값을 받는 서버의 캐릭터셋이 EUC-KR 이고, 
복호화를 할 시 캐릭터셋을 지정하지 않으니 우리의 Default 캐릭터 셋인 euc-kr로 복호화를 하면서 한글이 깨지는 현상이었다.​ 
그래서 복호화시 UTF-8을 지정하고 복호화를 하니 문제가 해결되었다.
