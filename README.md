# Querydsl

## 7. 스프링 데이터 JPA가 제공하는 Querydsl 기능

여기서 소개하는 기능은 제약이 커서 복잡한 실무 환경에서 사용하기에는 많이 부족하다.     
그래도 스프링 데이터에서 제공하는 기능이므로 간단히 소개하고, 왜 부족한지 설명한다.

### 7-1. 인터페이스 지원 - QuerydslPredicateExecutor

* 공식 URL: https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.extensions.querydsl

#### QuerydslPredicateExecutor 인터페이스

```java
public interface QuerydslPredicateExecutor<T> {
    Optional<T> findById(Predicate predicate);

    Iterable<T> findAll(Predicate predicate);

    long count(Predicate predicate);

    boolean exists(Predicate predicate);
    // … more functionality omitted.
}
```

#### MemberRepository.java - 적용

```java
package study.querydsl.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.querydsl.QuerydslPredicateExecutor;
import study.querydsl.entity.Member;

import java.util.List;

public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom, QuerydslPredicateExecutor<Member> {

    List<Member> findByUsername(String username);
}

```

```
Iterable result = memberRepository.findAll(
        member.age.between(10, 40)
        .and(member.username.eq("member1"))
);
```

* **한계점**
    * 조인x (묵시적 조인은 가능하지만 left join이 불가능)
    * 클라이언트가 Querydsl에 의존해야 한다. 서비스 클래스가 Querydsl이라는 구현 기술에 의존해야 한다.
    * 복잡한 실무 환경에서 사용하기에는 한계가 명확하다.

> 참고: `QuerydslPredicateExecutor`는 Pagable, Sort를 모두 지원하고 정상 동작한다.

### 7-2. QuerydslWeb 지원

* 공식 URL: https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.web.type-safe
* **한계점**
    * 단순한 조건만 가능
    * 조건을 커스텀하는 기능이 복잡하고 명식적이지 않음
    * 컨트롤러가 Querydsl에 의존
    * 복잡한 실무환경에서 사용하기에는 한계가 명확

## Note