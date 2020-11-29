---
layout: post
title: '2장 테스트'
date: 2020-11-23 12:44:00
category: [toby-spring]
draft: false
comments: true
---

# 2장 테스트

-   스프링이 개발자에게 제공하는 가장 중요한 가치는?
    -   객체지향과 테스트
-   애플리케이션은 계속 변하고 복잡해져 감
    -   이에 대응하는 두 가지 전략
        1.  확장과 변화를 고려한 객체지향적 설계와 이를 담을 수 있는 IoC/DI
        2.  변화에 유연하게 대처할 수 있는 자신감을 주는 테스트 기술

## 2.1. UserDaoTest 다시 보기

-   테스트의 유용성
    -   테스트란 결국 내가 예상하고 의도했던 대로 코드가 정확히 동작하는지 확신할 수 있게 해주는 작업
-   작은 단위의 테스트
    -   테스트하고자 하는 명확한 대상에만 집중하는 것이 바람직함
    -   "관심사의 분리" 원리가 여기에도 적용
    -   단위 테스트(unit test) : 작은 단위의 코드에 대한 테스트
        -   일반적으로 단위는 작을수록 좋음
        -   테스트 중에 DB가 사용되면 단위 테스트가 아니라는 주장
            -   UserDaoTest는 매번 USER 테이블의 내용을 비우고 테스트를 진행했음
            -   이처럼 사용할 DB의 상태를 테스트가 관장하고 있다면 이는 단위 테스트라고 해도 됨
            -   통제할 수 없는 외부의 리소스에 의존하는 테스트는 단위 테스트가 아니라고 보기도 하는 것임
-   UserDaoTest의 문제점
    -   수동 확인 작업의 번거로움 (콘솔에 찍힌 값을 눈으로 확인)
    -   실행 작업의 번거로움 (매번 main 메소드 직접 실행)

## 2.2. UserDaoTest 개선

-   Junit 테스트로 전환
    -   메소드는 public, 메소드에 `@Test`어노테이션 붙여주기
-   검증 코드 전환

```java
assertThat(user2.getName(), is(user.getName()));
```

-   assertThat() 은 첫번째 파라미터의 값을 뒤에 나오는 matcher와 비교해서 일치하면 다음으로 넘어감
-   is()는 matcher의 일종으로 equals()로 비교해주는 기능을 가졌음
-   내 궁금증 : 그래서 누구의 equals인가? Doc에 따르면 `examined objects`, 즉 순서상 앞의 객체

https://junit.org/junit4/javadoc/latest/org/junit/Assert.html#assertThat(T,%20org.hamcrest.Matcher)
https://junit.org/junit4/javadoc/latest/org/hamcrest/CoreMatchers.html#is(T)
https://junit.org/junit4/javadoc/latest/org/hamcrest/CoreMatchers.html#equalTo(T)

-   junit assertThat (Deprecated) 대신 hamcrest assertThat을 쓰자.

## 2.3. 개발자를 위한 테스팅 프레임워크 Junit

-   스프링 프레임워크 자체도 JUnit 프레임워크를 이용해 테스트를 만들어가며 개발됐다. (!)

-   테스트 결과의 일관성

    -   코드에 변경사항이 없다면 테스트는 항상 동일한 결과를 내야 한다
        -   DB에 남아 있는 데이터와 같은 외부 환경에 영향을 받으면 안 됨
        -   테스트를 실행하는 순서를 바꿔도 동일한 결과가 보장되도록 만들어야 함

-   deleteAll()
    -   addAndGet()이 기존 DB 내용에 영향받지 않게 하기 위해 수동으로 데이터 삭제
    -   해당 기능을 deleteAll()을 활용하여 구현하고 테스트에 추가
-   getCount()

    -   deleteAll()이 제대로 작동했는지를 getCount()를 통해 검증
    -   그 후 하나씩 추가할 때마다 getCount()의 갯수가 늘어났는지 검증

-   get() 예외조건에 대한 테스트 (TDD)
-   파라미터로 전달된 id값에 대한 정보가 없을 때
    -   null 리턴 / 예외 던지기
    -   예외 던지기로 정하고, 이 상황에 대한 테스트 작성
    -   그리고 이 테스트가 성공하도록 코드를 수정

```java
@Test(expected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLException {
    ...
    dao.get("unknown_id");
}

public User get(String id) throws SQLException {
...

if (user == null) throw new EmptyResultDataAccessException(1);

}
```

-   포괄적인 테스트

    -   이렇게 DAO의 메소드에 대한 포괄적인 테스트를 만들어놓는 편이 안전하고 유용함
    -   개발자들이 자주 하는 실수 : 성공하는 테스트만 만드는 것
        -   로드 존슨 (스프링 창시자) "항상 네거티브 테스트를 먼저 만들라"

-   테스트 주도 개발

    -   앞선 사례 : 테스트 코드가 마치 잘 작성된 하나의 기능정의서처럼 보임
    -   기본 원칙 : "실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다"
    -   테스트 작성과 성공 코드를 만드는 작업의 주기를 가능한 한 짧게 가져갈 것
    -   "며칠 밤새다가 찾아낸 버그" -> "진작 테스트 했으면 찾아낼 수 있었던 것을 미루고 미루다 커다란 삽질로 만든 일" ㅠㅠ

-   테스트 코드 개선

    -   테스트 코드에도 리팩토링이 필요하다!

-   JUnit이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식

1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾기
2. 테스트 클래스의 오브젝트를 하나 마듬
3. @Before가 붙은 메소드가 있으면 실행
4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과 저장
5. @After가 붙은 메소드가 있으면 실행
6. 나머지 테스트 메소드에 대해 2-5번 반복
7. 모든 테스트의 결과를 종합해서 돌려줌

-   픽스처fixture : 테스트를 수행하는 데 필요한 정보나 오브젝트

## 2.4. 스프링 테스트 적용

-   테스트를 위한 애플리케이션 컨텍스트 관리
    -   앱이 커질수록 매번 만들었다 지우면 너무 오래 걸림
    -   테스트에선 Autowired를 쓰고, 모든 테스트에서 하나의 컨텍스트를 공유하게 하자

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {

    @Autowired
    private ApplicationContext context;
}
```

-   Autowired

    -   Autowired는 변수에 할당 가능한 타입을 가진 빈을 자동으로 찾음
    -   단, 같은 타입의 빈이 두 개 이상 있는 경우에는 타입만으로는 어떤 빈을 가져올지 결정할 수 없음
        -   이 경우 변수의 이름과 같은 빈을 먼저 가져옴

-   DI와 테스트

    -   "난 절대 DataSource의 구현을 안 바꿀 건데? 그래도 인터페이스로 사용해야 하나?"
        -   그렇다
        -   1. 소프트웨어 개발에서 절대 바뀌지 않는 것은 없다
        -   2. 클래스의 구현 방식은 바뀌지 않더라도 인터페이스를 두고 DI 적용시 다른 차원의 서비스 기능 도입 가능
        -   3. 테스트

-   테스트 코드에 의한 DI
    -   UserDao가 사용할 DataSource 오브젝트를 테스트 코드에서 바꿀 수 있다!

```java
@DirtiesContext
public class UserDaoTest {
    ...

    @Before
    public void setUp() {
        ...
        DataSource dataSource = new SingleConnectionDataSource("테스트용 db connection..");
        dao.setDataSource(dataSource);
    }
}
```

-   이 경우 DirtiesContext 어노테이션을 사용하자.
-   이 클래스의 테스트에서 애플리케이션 컨텍스트의 상태를 변경하고 있다는 것을 알려줌
    https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/annotation/DirtiesContext.html
-   하지만.. 뭔가 찜찜하다

    -   테스트용 test-applicationContext.xml을 만들어서 쓸 수도 있다

-   컨테이너 없는 DI 테스트
    -   사실 스프링 컨테이너에서 UserDao가 동작하는 것은 테스트의 기본적인 관심사가 아님
    -   그냥...
    ```java
      dao = new UserDao();
      dao.setDataSource(...);
      ...
    ```
    -   스프링은 비침투적(noninvansive) 기술의 대표적인 예 (애플리케이션 코드가 특정 기술에 종속 X)
-   테스트하기 불편하게 설계된 좋은 코드를 지금까지 본 적이 없다!

-   테스트 방법 선택하기
    -   1. 스프링 컨테이너 없이 테스트할 수 있는 방법을 우선적으로 고려
    -   2. 여러 오브젝트와 복잡한 의존관계 가진 경우 -> 스프링 설정을 이용한 DI 방식 테스트

## 2.5. 학습 테스트로 배우는 스프링

-   자신이 만들지 않은 프레임워크나 다른 개발팀에서 만들어서 제공한 라이브러리 등에 대해서 테스트를 작성하는 것
-   목적 : 자신이 사용할 API나 프레임워크의 기능을 테스트로 보면서 사용 방법 익히기
-   저자는 새로운 프레임워크 사용하거나 기술 사용시 항상 테스트코드를 먼저 만들어 봄

-   장점

    -   다양한 조건에 따른 기능을 손쉽게 확인 가능
    -   학습 테스트 코드를 개발 중에 참고할 수 있음
    -   프레임워크나 제품 업그레이드 시 호환성 검증을 도와줌
    -   테스트 작성에 대한 좋은 훈련이 됨
    -   새로운 기술을 공부하는 과정이 즐거워짐

-   스프링 학습 테스트를 만들 때 참고할 수 있는 가장 좋은 소스 : 스프링 자신에 대한 테스트 코드 (!)

    -   스프링 테스트 코드를 잘 읽어보면 레퍼런스 문서에서는 미처 설명되지 않은 중요한 정보도 많이 있음

-   학습 테스트 예제 : JUnit
-   정말 테스트 메소드를 수행할 때마다 새로운 오브젝트를 만드는지?
-   스프링 테스트 컨텍스트 프레임워크에서 만든 컨텍스트가 테스트마다 공유되는지?
-   그 외 직접 만들어볼 만한 것들 (\*)

    -   스프링이 싱글톤 방식으로 빈의 오브젝트 만드는 것 검증
    -   테스트 컨텍스트를 이용한 테스트에서 Autowired로 가져온 것이 애플리케이션 컨텍스트에서 직접 getBean()으로 가져온 것과 동일한지
    -   XML에서 스트링 타입의 프로퍼티 값을 설정한 것이 정말 빈에 잘 주입되는지
    -   getBean() 사용했는데 주어진 이름의 빈이 발견되지 않으면?

-   버그 테스트
    -   코드에 오류가 있을 때 그 오류를 가장 잘 드러내줄 수 있는 테스트
    -   일단 실패하도록 만들고 버그 테스트가 성공할 수 있도록 애플리케이션 코드 수정
