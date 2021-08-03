# Querydsl

## 6. 실무 활용 - 스프링 데이터 JPA와 Querydsl

### 6-1. 스프링 데이터 JPA 리포지토리로 변경

#### MemberRepository.java - 스프링 데이터 JPA

* `src/main/java/study/querydsl/repository/MemberRepository.java`

```java
package study.querydsl.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import study.querydsl.entity.Member;

import java.util.List;

public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(String username);
}

```

#### MemberRepositoryTest.java - 스프링 데이터 JPA 테스트

* `src/test/java/study/querydsl/repository/MemberRepositoryTest.java`

```java
package study.querydsl.repository;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import study.querydsl.entity.Member;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@Transactional
class MemberRepositoryTest {
    @Autowired EntityManager em;
    @Autowired MemberJpaRepository memberRepository;

    @Test
    @DisplayName("Spring Data JPA 적용 테스트")
    public void basicTest() {
        Member member = new Member("member1", 10);
        memberRepository.save(member);

        Member findMember = memberRepository.findById(member.getId()).orElseThrow();
        assertThat(member).isEqualTo(findMember);

        List<Member> result1 = memberRepository.findAll();
        assertThat(result1).containsExactly(member);

        List<Member> result2 = memberRepository.findByUsername("member1");
        assertThat(result2).containsExactly(member);
    }

}

```

## Note