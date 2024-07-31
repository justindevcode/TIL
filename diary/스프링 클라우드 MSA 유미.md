# MSA란

## 참고자료
https://www.youtube.com/watch?v=VoonWkCJxcQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=2

## MSA : Micro Service Architecture
MSA는 마이크로 서비스 아키텍처의 약자로 특정 서비스를 구축하는 방식을 의미한다.  

## 서비스 구축 방법
* 모놀로식
과거 부터 현재까지 주로 이용되는 서비스 구축 방식으로 하나의 프로젝트에 모든 비즈니스 로직과 설정 데이터들을 넣어 개발하는 방식이다

* MSA
모놀리식 서비스에서 각각의 비즈니스 로직을 분리하여 개별 프로젝트로 생성한 뒤 가장 앞단에서 API Gateway와 같은 분배기를 통해 각 서비스 서버에 요청을 분산하여 관리한다.

## 모놀리식과 MSA
![1](https://github.com/user-attachments/assets/90d1f340-ea58-4e3d-9946-4f75ddb4c181)  

## MSA 장단점
* 장점
서비스별 스케일링 가능  
서비스별 다른 프레임워크 사용 가능  
하나의 서비스가 off 되더라도 나머지 동작 가능  
부분적으로 로직 업데이트 가능

* 단점
초기 구성의 난이도  
시스템이 돌아가지만 특정 서비스가 off되면 재기능을 못할 수 있음  
서버간 호출 비용  
분산 관리

## MSA의 여러 요소

서비스 로직  
게이트웨이  
모니터링 서버  
변수관리 서버(여러 서버에서 DB관련정보 한번에 바꾸려면 변수주입서버 필요)  

## 스프링 프레임워크에서의 MSA
스프링 프레임워크도 MSA를 지원하기 위해 MSA 관련 의존성들을 제공한다.  

## 추가참고 영상
https://www.youtube.com/watch?v=CM47-1UpgOc  

---

# 스프링에서 msa

## 참조
https://www.youtube.com/watch?v=VoonWkCJxcQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=2  

## 스프링에서 MSA 모식도
![1](https://github.com/user-attachments/assets/52db5ec9-1607-48c0-bebf-96111bdf7b58)  

## 스프링 부트
비지니스 로직 처리를 담당하기 위한 어플리케이션 서버  
즉 특정 경로에 대한 컨트롤러를 통해 요청 처리를 진행한다.  

## Spring Cloud Gateway : SCG
URL 주소에 대해서 아래 세부 경로에 따라 각각의 스프링 부트 어플리케이션에 분배하는 분배기 역할을 수행  

## Spring Cloud Eureka Server
모니터링 서버로 Eureka Client 설정을 해둔 서버를 Eureka Server에 띄워 줌  
모니터링 기능과 함께 추가적으로 Spring Cloud Gateway에 목록을 전달하여 Gateway가 로드밸런싱 대상을 설정하도록 작업  

## Spring Cloud Eureka Client
Eureka Server에 등록되는 요소로 스프링 부트 어플리케이션과 같은 여러 스프링 프레임워크 서버에 설정이 가능하다.  

## Spring Config Server
변수 값들을 제공하는 서버로 특정 경로로 접근하면 미리 사전에 설정해둔 변수 값들을 제공 받을 수 있다.  
MSA를 구축하면 각각의 스프링 부트 어플리케이션에 application.properties에 값을 명시하는 것이 아닌 config server로 부터 데이터를 받아서 사용한다.  

## Config Repository
Config Server는 단순하게 데이터를 전달하는 매개체로 실제 데이터는 Config Server 뒷단에 깃허브 리포지토리와 같은 저장소를 물려서 사용한다.  

## Spring Config Client
Config Server로 부터 변수 데이터를 받기 위한 Client 서버 설정  
