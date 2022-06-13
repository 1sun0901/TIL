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

    