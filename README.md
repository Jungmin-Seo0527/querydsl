# Querydsl

## 1. 프로젝트 환경 설정

### 1-1. 프로젝트 생성

#### build.gradle

```groovy
plugins {
    id 'org.springframework.boot' version '2.5.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}

```

### 1-2. Querydsl 환경설정 검증

#### Hello.java - 검증용 엔티티 생성

* `src/main/java/study/querydsl/entity/Hello.java`

```java
package study.querydsl.entity;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
@Getter @Setter
public class Hello {

    @Id @GeneratedValue
    private Long id;
}

```

#### 검증용 Q타입 생성

* **Gradle IntelliJ 사용법**
    * Gradle -> Tasks -> build -> clean
    * Gradle -> Tasks -> other -> compileQuerydsl

* **Gradle 콘솔 사용법**
    * `./gradlew clean compileQuerydsl`

* Q타입 생성 확인
    * build -> generated -> querydsl
        * study.querydsl.entity.QHello.java 파일이 생성되어 있어야 함

> 참고: Q타입은 컴파일 시점에 자동 생성되므로 버전관리(GIT)에 포함하지 않는 것이 좋다. 앞서 설정에서 생성 위치를 gradle build 폴더 아래 생성되도록 했기 때문에 이 부분도 자연스럽게 해결 된다. (대부분 gradle build 폴더를 git에 포함하지 않는다.)

#### 테스트 케이스로 실행 검증

```java
package study.querydsl;

import com.querydsl.jpa.impl.JPAQueryFactory;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;
import study.querydsl.entity.Hello;
import study.querydsl.entity.QHello;

import javax.persistence.EntityManager;

import static org.assertj.core.api.Assertions.*;

@SpringBootTest
@Transactional
class QuerydslApplicationTests {

    @Autowired EntityManager em;

    @Test
    void contextLoads() {
        Hello hello = new Hello();
        em.persist(hello);

        JPAQueryFactory query = new JPAQueryFactory(em);
        QHello qHello = QHello.hello;

        Hello result = query
                .selectFrom(qHello)
                .fetchOne();

        assertThat(result).isEqualTo(hello);
        assertThat(result.getId()).isEqualTo(hello.getId());
    }

}

```

### 1-3. 라이브러리 살펴보기

* gradle 의존관계
    * `./gradlew dependencies --configuration compileClasspath`

* Querydsl 라이브러리
    * querydsl-apt: Querydsl 관련 코드 생성 기능 제공
    * querydsl-jpa: querydsl 라이브러리

* 스프링 부트 라이브러리 살펴보기
    * spring-boot-starter-web
        * spring-boot-starter-tomcat: 톰캣 (웹서버)
        * spring-webmvc: 스프링 웹 MVC
    * spring-boot-starter-data-jpa
        * spring-boot-start-aop
        * spring-boot-starter-jdbc
            * HikariCP 커넥션 풀 (부트 2.0 기본)
        * hibernate + JPA: 하이버네이트 + JPA
        * spring-data-jpa: 스프링 데이터 JPA
    * spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
        * spring-boot
            * spring-core
        * spring-boot-starter-logging
            * logback, slf4j

* 테스트 라이브러리
    * spring-boot-starter-test
        * junit: 테스트 프레임웤, 스프링 부트 2.2부터 junit5(`Jupiter`) 사용
            * 과거 버전은 `vintage`
        * mockito: 목 라이브러리
        * assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
            * https://joel-costigliola.github.io/assertj/index.html
        * spring-test: 스프링 통합 테스트 지원
    * 핵심 라이브러리
        * 스프링 MVC
        * JPA, 하이버네이트
        * 스프링 데이터 JPA
        * Querydsl
    * 기타 라이브러리
        * H2 데이터베이스 클라이언트
        * 커넥션 풀: 부트 기본은 HikariCP
        * 로깅 SLF4J & LogBack
        * 테스트

### 1-4. H2 데이터베이스 설치

개발이나 테스트 용도로 가볍고 편리한 DB, 웹 화면 제공

* https://www.h2database.com/html/main.html
* 다운로드 및 설치
* h2 데이터베이스 버전은 스프링 부트 버전에 맞춘다.
* 권한 주기: `chmod 755 h2.sh`
* 데이터베이스 파일 생성 방법
    * `jdbc:h2:~/querydsl`(최소 한번)
    * `~/querydsl.mv.db` 파일 생성 확인
    * 이후 부터는 `jdbc:h2:tcp://localhost/~/querydsl`

> 참고: H2 데이터베이스의 MVC 옵션은 H2 1.4.198 버전부터 제거

## Note