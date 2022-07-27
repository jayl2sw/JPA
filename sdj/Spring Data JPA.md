# Spring Data JPA

### 메소드 이름으로 쿼리 생성 (쿼리 메소드)

메소드 이름을 분석해서 JPQL 쿼리 실행

```java
package com.study.datajpa.repository;

import com.study.datajpa.entity.Member;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface MemberRepository extends JpaRepository<Member, Long> {
    
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}

```

생성 조건은 Spring Data JPA 메뉴얼 확인

* https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation

* `find...By`, `read...By`, `query...By`, `get...by`

* `count...By`, `exist...by`,

* 삭제 : `delete...By`, `remove...By`

* DISTINCT: `findDistinct`, `findMemberDistinctBy`

* LIMIT: `findFirst3`, `findFirst`, `findTop3`

  

**장점**

* 변수명을 하나 바꾸면 컴파일러가 에러를 잡아준다.



### JPA NamedQuery

잘 안씀



### 레포지토리 메서드에 쿼리를 바로 정의



