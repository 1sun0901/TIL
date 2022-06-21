# TIL

2022.6.13

Today I Learned 시작

TIL잘한것: http://github.com/cheese10yun/TIL

1. CQRS패턴

CQRS는 Command Query Responsibility Segregation 의 약자로 단어 그대로 해석하면 "명령 조회 책임 분리"입니다. 이는 애플리케이션들을 구성하는 아키텍처에 대한 하나의 패턴입니다. 즉, 애플리케이션을 구현함에 있어 명령과 조회에 대한 책임을 분리하는 것이 CQRS입니다. 
출처: https://always-kimkim.tistory.com/entry/cqrs-pattern [언제나 김김:티스토리]

낮은 수준의 CQRS는 Read하는 모델을 따로 만들거나, Read Replica를 만든다거나, Read DB에 Cache를 적용하는 등 다양하게 또 간단하게 적용해볼 수 있습니다.
허나 단순 모델의 분리로는 Single DB의 성능 문제를 해결하지 못했고, 읽기 전용 DB 분리는 동기화 방식의 고민이 남아있었습니다.



때문에 더 높은 수준의 CQRS 패턴은 Event Sourcing과 함께, Queue(AWS SQS, RabbitMQ, Kafka와 같은 Message Queue가 일반적)를 이용하여 비동기적으로 데이터를 쓰고 읽어오는 형태를 취합니다.

결론 : 읽기위한 DB 쿼리가 훨씬 많아 부하가 많고, 성능측면에서 읽기 DB를 하나 더 두는 것 같음.
그럼 별도로 읽기위한 Target DB를 하나 두어 Source DB에서 Target DB로 동기화하는 걸 Kafka로 적용해서 둔다는 말

2. SAGA패턴

SAGA 패턴이란 마이크로서비스들끼리 이벤트를 주고 받아 특정 마이크로서비스에서의 작업이 실패하면 이전까지의 작업이 완료된 마이크서비스들에게 보상 (complemetary) 이벤트를 소싱함으로써 분산 환경에서 원자성(atomicity)을 보장하는 패턴입니다.


3. JPQL

예제)

@Test
	public void testJPQL() {
		String qlString = "select m from Member m " +
							"where m.userName = :userName";

		Member findMember = em.createQuery(qlString, Member.class)
				.setParameter("userName", "kang")
				.getSingleResult();

		assertThat(findMember.getUserName()).isEqualTo("kang");
	}

4. queryDsl : 동적인 쿼리의 경우 사용하면 됨.

예제)

@Test
	public void testQuerydsl() {
		queryFactory = new JPAQueryFactory(em);
		QMember m = new QMember("m");

		Member findMember = queryFactory
				.select(m)
				.from(m)
				.where(m.userName.eq("kang"))
				.fetchOne();

		assertThat(findMember.getUserName()).isEqualTo("kang");

	}

    5. Feign Client란?
Feign은 Netflix에서 만든 선언적 Http 클라이언트이다. 여기서 선언적이란 말은 Spring에서 선언적 트랜잭션이라는 용어와 비슷한데, 소스코드가 아닌 어노테이션 선언만으로 트랜잭션을 적용하게 하는 기술이다. Feign에서의 선언적 Http클라이언트 역시 어노테이션 선언만으로 Http 클라이언트를 만들 수 있고 이를 통해서 Http Api 호출이 가능한 것을 의미한다.
    
Controller에 정의되어 있는 API를 호출하면
그 메서드에 정의되어 있는 Interface class feignclient의 메서드를 호출할 수 있게 되고
거기 feignclent에 정의되어 있는 api를 또 호출해서 자동으로
쉽게 api를 호출할 수 있다.

localhost:8080/v1/github/OpenFeign/feign
--> https://api.github.com/repos/OpenFeign/feign/contributors

2022.6.14

1. @GetMapping(produces = "application/json")

GetMapping해서 조회하는 데이터를 json형태로 바꿔준다.

2. Circuit breaker 패턴이란

Circuit breaker 패턴은 그림 1에서 server A와 server B사이에 circuit breaker를 두는 것이다.

그림 ) client <-> ServerA <-> Circuit Breaker <-> Server B
server A가 server B을 호출할 때 circuit breaker가 모든 트래픽을 bypass 한다.
이때 server B에 응답이 없다면 circuit breaker가 server B로의 호출을 강제로 끊고 server A에게 에러 메시지를 날린다.
이를 더 발전시켜 server A에게 에러 메시지를 보내는 것이 아니라 다른 의미 있는 메시지를 보내줄 수 있다. (Fall-back 메시징)

Circuit breaker 패턴 사용법
넷플릭스의 자바 라이브러리 - Hystrix (소프트웨어)
장점 : 추가 서버 관리 비용이 들지 않는다.
단점 : 코드 수정 필요, 자바 언어에 종속적
envoy.io 프록시 서버 (하드웨어)
장점 : 코드 수정 필요 x, 언어에 종속적 x
단점 : 프록시 서버를 하나 더 운영해야 하는 부담 

3. 분산 트랜잭션 - 2단계 커밋
첫 번째
첫 번째 단계에서는 노드들에게 트랜잭션 생성과 함께 명령을 수행한다. 여러 노드에서 실행되는 트랜잭션 관리를 위해 이 트랜잭션 id는 coordinator가 관리한다.

 
두 번째
모든 노드들에게 commit이 가능한지 물어본다. 이 요청에 각 노드들은 현재 트랜잭션을 commit 할 수 있는지 응답한다.

 
세 번째
두 번째 단계에서 모든 노드들에게 받은 응답을 취합한다. 하나라도 NO가 있다면 전체 노드들에게 abort를 보내고, 모두가 OK라면 commit을 보낸다.

4. MSA란?

통합되어 있던 Application과 Database를 "상품", "배송", "회원관리", "주문"의 각각의 서비스별로 작은 단위로 분산하여 구성하며 상호간에 약속된 규격으로 통신합니다. 이로서 만약 하나의 서비스 혹은 DB에서 장애가 발생하더라도 다른 서비스로의 영향은 아주 제한적이게 됩니다.

 

대표적인 통신 방식으로는 REST기반의 API 통신이 있으며, 그외에도 Message Queue 등 다양한 통신 프로토콜을  이용 합니다.   
API Gateway는 외부에의 유일한 진입로 입니다. 자세한 사항은 다음장에서 다룰 예정입니다.
 

또한 개발과 운영 관점에서도 다양한 장점이 존재 합니다.

각 서비스들은 Rest API 등으로 상호간에 규칙만 정의하면 서로의 서비스에 영향을 끼치지 않습니다.

 

하여, 개인 혹은 팀 단위의 서비스들은 오로지 자신의 서비스 개발&운영에 집중할 수 있습니다.

만약 프로젝트에 신규 인력이 투입된다면 모놀리식 프로젝트에서는 해당 신규 인력의 당담업무를 분석하는데 있어 다른 업무와의 연관관계가 높아 를 파악이 어렵지만, MSA 프로젝트에서는 외부와의 Interface만 알면 자신의 업무 파악에

집중할 수 있기에 효율성이 증가됩니다.

 

또한 운영중에도 신규 기능 등의 개발 요구사항이 계속 반영되어야 하기 때문에 작은 서비스 단위의 개발, 테스트, 운영 배포가 용이해집니다.

 

또 다른 장점으로는 모놀리식 아키텍처와 달리 MSA 기반에서는 새로운 언어나 기술의 도입이 쉽습니다.

기존의 운영 서비스들과 Interface만 약속하면 새로 만들어질 서비스에서 원하는 언어와 기술로 개발하면 되니까요.

물론 이러한 MSA가 완벽한 것만은 아니며, 다양한 단점이 존재합니다.

 

일단 아키텍처 적인 측면에서 단조로운 모놀로식보다 복잡합니다.

다양하게 분산된 서비스들을 관리하고 통신하여 하하며, 장애가 발생하였을 경우 대처하는 방법도 까다롭다 보니 모놀리식 아키텍처에서 신경 쓰지 않아도 될 다양한 이슈들이 발생합니다

 

Dababase의 트랜잭션을 유지하기 어렵다는 것이 큰 이슈 중 하나 입니다.

각 서비스 별로 Database가 분리되어 있다는 것이 단점이 될 수 있습니다.

예를 들어 모놀리식 아키텍처에서는 단 한번의 서비스 요청이 하나의 트랜젝션으로 묶일수가 있어(DB가 하나이기에)

행여 처리중에 오류가 발생하더라도 전체 Rollback이 쉽습니다.

 

하지만, MSA에서는 각 서비스마다 DB가 분리되어 있기 때문에 여러 서비스를 거치다 특정 서비스에서

오류가 발생했을 경우 이미 완료처리된 서비스를 Rollback 하는 것이 쉽지 않습니다.

 

(하지만 Neflix OSS 및 Spring Cloud 프레임워크가 있기에 그나마 MSA를 편리하게 구성할 수 있어 정말 다행이지 않을 수 없습니다. 우리는 이러한 프레임워크를 앞으로 배워나가볼 예정입니다.)


5. MSA를 위한 핵심 Framework

Spring Cloud는 Spring Boot 기반으로 제작되었으며, 클라우드 환경을 보다 편리하게 구축하기 위해 설계된 라이브러리 입니다.

이러한 Spring Cloud에서는 MSA 구축에 널리 쓰인 Netflix OSS 프레임워크의 핵심적인 라이브러리리를 포함하고 있습니다.

Netflix OSS의 대표적인 핵심 라이브러리인 Eureka, Ribbon, Hystrix, Zuul과 Spring Cloud의 Config의 이론과 실습을 진행할 계획입니다.

Eureka, Ribbon,Hystrix,Zuul, Config의 5가지의 요소를 충분히 이해하고 구현해 보는 것이 개발 관점에서 MSA를 개념을 이해하는 시작이자 필수적인 요소이라고 생각합니다.

 

어려운 개발 없이도 단지 이러한 1) 라이브러리를 추가하고 2) 간단한 properties 설정하고 3) 특정 Annotation을 등록함으로써 손쉽게 Micro Services Architeture를 구현할 수 있습니다. 

MSA 환경에서는 다양한 서비스들이 존재하며 각 서비스들은 고유한 IP와 PORT, 그리고 기본 정보를 가지고 있습니다. 그리고 서로 통신을 하기 위해서는 이 IP와 PORT와 같은 정보를 서로 알고 있어야 하겠지요. 

다행히도 우리는 이런 걱정을 덜어낼수 있습니다. 바로 서비스 디스커버리 패턴이라는 좋은 방안이 있습니다.
디스커버리 서버는 이러한 가변적인 모든 서비스의 정보들은 각 서비스의 고유ID(거의 변하지 않는)에 매핑하여 관리합니다.

 

그리고 각 서비스들은 지속적으로 1) 자신의 정보를 디스커버리 서버에 등록 하며 2) 다른 서비스들의 정보를 조회 합니다. 그럼 각 서비스들은 다른 서비스의 IP와 PORT를 몰라도 서비스의 고유ID만 가지고 연계가 가능해집니다.

 

이러한 서비스 디스커버리의 구현을 쉽게 도와주는 것이 Spring Cloud의 Eureka 입니다.

 

Eureka는 아래의 2가지로 구성 됩니다.

Eureka Server (서버 라이브러리) : 서비스들의 정보를 관리, 제공

Eureka Client (클라이언트 라이브러리) : 서버에 자신의 정보를 제공, 다른 서비스의 정보를 조회
말그대로 Eureka Server는 디스커버리 서버에 탑재되며, Eureka Client는 각각의 서비스들에 탑재됩니다.

위의 그림과 같이 각 서비스들이 기동될때 혹은 가동 중에도 지속적으로 자신의 정보를 디스커버리 서버에 등록(갱신)하며, 서버는 이 정보들을 캐싱하여 관리합니다.

만약 상품서비스에서 주문서비스로 HTTP 통신을 한다고 하는 경우를 예를 들어 봅시다.

대다수의 서비스들은 단일 인스턴스로 구성되지 않고 고가용성을 위해 여러개의 인스턴스로 이중화되어 구성되어 집니다.

만약 주문서비스가 2개 이상의 인스턴스로 구성되어 있다면, 상품서비스는 어떤 인스턴스로 요청할지 어떻게 결정해야 할까요?

Spring Cloud 환경에서는 더이상 물리적인 L4를 사용하지 않아도 됩니다. 바로 Ribbon이라고 하는 라이브러리가 도와주니까요

만약 상품서비스가 주문서비스로 고유ID인 order로 요청하면 Ribbon이 Eureka로 부터 얻은 정보를 이용하여 정의된 알고리즘에 따라 자동으로 부하 분산을 해줍니다. (상품서비스는 주문서비스의 인스턴스를 알 필요가 없습니다. 오직 주문서비스의 고유ID만 알면 되지요.)

이를 클라이언트측 부하분산 혹은 소프트웨어적 로드밸런싱이라고 부르기도 합니다.

여기서 한가지 의문점이 있습니다.

내부에서는 디스커버리 서버를 통해 서로의 정보를 공유하고 통신하는데 문제가 없지만, 외부 클라이언트에서는 각 서비스들의 정보는 어떻게 알아야 할까요? 디스커버리 서버에 접근을 할수가 없는데 모든 서비스의 IP와 PORT를 알고 있어야 할까요?

여기의 해답에는 API Gateway가 있습니다.

그리고 이러한 API Gateway를 쉽게 구현할 수 있게 도와주는 도구로 Spring Cloud의 Zuul1, Zuul2, Spring Cloud gateway 등 다양한 라이브러리가 있습니다.

외부에서는 내부의 서비스들을 몰라도 됩니다. 단지 공개된 규격의 URL를 이용하여 API Gateway 서버로 요청하면 API Gateway가 알아서 해당 서비스로 호출하고 응답을 받아서 외부 클라이언트에게 답해줍니다.

외부 클라이언트가 유효한 사용자인지 인증(Authentication)을 하거나 해당 사용자가 원하는 자원에 접근할 권한이 있는지 인가(Authorization)하는 등의 작업을 각 서비스가 신경 쓸 필요없이 모두 API Gateway가 처리하면 됩니다.


Config Server

Config Server에서 중앙집중식으로 환경설정(프로퍼티)을 관리할 수 있습니다. 즉 코드와 환경설정을 분리하는 것이 Micro Services Architecture 사상에 적합니다. 


2022.6.17

Zuul의 기본 구성

Zuul은 요청이 들어 오면 Pre Filter > Route Filter > Post Filter의 순으로 동작 하며 각 필터들의 역할은 다음과 같습니다.

1. Pre Filter --> 2. Route Filter --> 3. Post Filter
						| ^
						| |
						v |
					   서비스
 

Pre Filter

대상 서비스로 요청이 Routing 되기 전에 실행되는 필터입니다.

대상 서비스에 요청하기 전에 유효한 요청인지 인증(Authentication)이나 인가(Authorization)를 하거나, 요청 로깅, 요청의 유효성 검사, 
공통 초기화 설정 등의 기능을 처리하기에 좋은 위치 입니다.

 
Route Filter

따로 구현하지 않아도 Zuul 내부적으로 기본 Route Filter(RibbonRoutingFilter)가 작동하여 대상 서비스에 라우팅을 수행합니다.

Custom Route Filter를 구현하면 기본 라우팅이 수행되기 전에 호출을 가로채어 동적으로 원하는 Routing 기능을 구현 할 수 있습니다.

예를 들어 원본 대상 라우팅전에 반드시 거쳐야 하는 특정 서비스를 호출해야 한다거나, 카나리아 테스트를 위해 라우팅 대상 서비스를 동적으로 변환하여 라우팅하는 등의 기능을 구현할 수 있습니다.


Post Filter

대상 서비스를 호출하고 최종 응답을 클라이언트에 반환한 뒤에 호출되는 메서드 입니다.

응답 로깅을 하거나, 완료 후 처리되는 공통 로직 등의 기능을 넣기에 적절한 위치 입니다.

 
2022.6.21

1. Service Mesh란?

Kubernetes에서 동작하는 다양한 어플리케이션이 서로 데이터를 공유하거나, 인증 및 라우팅하는 것을 지원하는 기술이다. 특히 Microservice 환경에서 많이 활용되는 네트워크 구성이고, 네트워크의 형태가 mesh형식으로 구성되는 경우가 많아 service mesh라고 명명되었다. 
실제 서비스(microservice)는 사용자가 구축한 어플리케이션이고, 내부 통신은 sidecar를 통해서 이루어진다. Service mesh는 이 side car에 통신과 관련된 많은 기능들을 제공하여, 어플리케이션 코드의 변경없이 다양한 네트워크 구성(인증, 라우팅, 보안 등)을 적용할 수 있도록 한다. 
API Gateway 구성

API Gateway는 외부의 단일접점을 제공하고, 내부의 복잡한 구성/연결을 추상화하여 외부에 제공한다.  API Gateway는 비즈니스 로직을 포함한 어플리케이션에 직접 연결하게 되므로, 네트워크의 변경(보안, 인증, 프로토콜 등)이 있을 경우 어플리케이션에 영향을 미치게 된다. (어플리케이션 수정 등)
Service Mesh 구성

Service Mesh는 API Gateway를 통해 들어온 내부 네트워크를 관리하는데 초점을 맞추고있다. 특히 기존 어플리케이션 영역(Business Logic)과 네트워크 영역을 분리(Sidecar 추가)하여 네트워크의 변경 및 적용을 분리된 영역에서 처리하게 되므로, 네트워크의 다양한 변경에도 어플리케이션에 영향이 없이 적용 가능하다.  
사이드카(Sidecar) 패턴은 모든 응용 프로그램 컨테이너에 사이드카 컨테이너를 추가하여 배포합니다. 사이드카는 서비스에 들어오거나 나가는 모든 네트워크 트래픽을 처리합니다. 가장 큰 특징은 비즈니스 로직이 포함된 실제 서비스와 사이드카가 병렬로 배치되기 때문에 서비스가 타 서비스를 직접 호출하는 것이 아니라 프록시(proxy)를 통해서 호출한다는 점입니다. 따라서 대규모의 마이크로서비스 환경이더라도 별도 작업 없이 서비스 연결뿐만 아니라 로깅, 모니터링, 보안, 트래픽 제어가 가능하다는 장점이 있습니다.


2. 도커란?

컨테이너를 하나만 띄워서 사용해야지! => 도커

0월 0시에, 100개의 컨테이너를 자동으로 생성해야지! => 쿠버네티스

즉, 도커는 ’이미지를, 컨테이너에 띄우고 실행하는 기술’이고
쿠버네티스는 '도커를 관리하는 툴'이라고 생각하시면 됩니다.
따라서, 도커는 '한 개의 컨테이너를 관리’하는 데 최적화 되어있고,
쿠버네티스는 '여러 개의 컨테이너를, 서비스 단위로 관리’하는 데 최적화 되어있습니다.

도커는 '컨테이너 기반의 오픈소스 가상화 플랫폼' 입니다.
그렇다면, 컨테이너란 무엇일까요?
컨테이너는, ‘애플리케이션’과 ‘애플리케이션을 구동하는 환경’을, ‘Host OS’ 로부터 격리한 공간을 의미합니다.

기존의 가상머신(VM) 서버
[Server → Host OS → Hypervisor → 각각의 Guest OS 가 설치된 VM 구동] (그림에는 VM 의 Host OS 가 빠져있습니다.)

장점

Host OS 가 Window 여도, Guest OS 로 Linux 를 사용할 수 있습니다.
보안적으로, Guest OS 가 뚫렸을 경우, 다른 Guest OS 와 Host OS 가 완벽하게 분리되어 있기 때문에, 각각의 VM 에 피해가 가지 않습니다.

단점

VM 마다 무거운 Guest OS 를 띄우기 때문에, Container 에 비해 속도가 느립니다.

컨테이너 서버
[Server → Host OS → Docker Engine → Container 구동]

장점
하나의 Host OS 를 공유하기 때문에, Container 별로 무거운 OS 를 띄우지 않아, Container 의 속도가 훨씬 빠릅니다.

단점
Host OS 가 Window 라면, Guest OS 로 Linux 를 사용할 수 없습니다.
보안적으로, Container 가 뚫렸을 경우, 다른 Container 와 Host OS 가 위험해질 수 있습니다.


3. Keycloak?


