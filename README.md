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

## Note