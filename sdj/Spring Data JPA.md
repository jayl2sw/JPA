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

```java
@Query("select m from Member m where m.username = :username and m.age > 15")
List<Member> findUser(@Param("username") String username, @Param("age") int age);
```

* 오타를 치면 application 정의 시점에 문법 에러를 알려준다.



동적 SQL은 어떻게 하나?	=> QueryDSL





### @Query, 값, DTO 조회하기

```java
@Query("select new com.study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
List<MemberDto> findMemberDto();
```



### 파라미터 바인딩

```java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```



### 반환 타입

* 유연한 반환타입 지원

* 단건
* 컬렉션
* 단건(Optional)





### Pagination and Sort

#### By Natual JPA

```java
public List<Member> findByPage(int age, int offset, int limit) {
    return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
            .setParameter("age", age)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
    
}

public long totalCount(int age) {
    return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
            .setParameter("age", age)
            .getSingleResult();
}
```

​	

#### By Spring Data JPA

* `org.springframework.data.domain.Sort` : 정렬 기능
* `org.springframework.data.domain.Pageable`: 페이징 기능(내부에 Sort 포함)

특별한 반환 타입

* `org.springframework.data.domain.Page` : 추가 count 쿼리 결과를 포함하는 페이지
* `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 다음페이지만 확인가능 
  * 모바일에서 더보기 가능한지 확인할 때 사용 가능
* `List`

```java
// MemberRepository
Page<Member> findByAge(int age, Pageable pageable); // 위에서 method 이름으로 query문 만드는 것 과 같음
// 사용할 때 PageRequest pageRequest = PageRequest.of(page, size, Sort.by(Sort.Direction.DESC, properties))
// PageRequest 의 부모 interface 가 Pageable
```

```java
// test
@Test
public void paging() {

    //given
    memberRepository.save(new Member("member1", 10));
    memberRepository.save(new Member("member2", 10));
    memberRepository.save(new Member("member3", 10));
    memberRepository.save(new Member("member4", 10));
    memberRepository.save(new Member("member5", 10));

    int age = 10;
    PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

    //when
    //자동으로 total count 요청을 해준다.
    Page<Member> page = memberRepository.findByAge(age, pageRequest);

    //then
    List<Member> content = page.getContent();
    long totalElements = page.getTotalElements();

    assertThat(content.size()).isEqualTo(3);
    assertThat(page.getTotalElements()).isEqualTo(5);
    assertThat(page.getNumber()).isEqualTo(0);
    assertThat(page.getTotalPages()).isEqualTo(2);
    assertThat(page.isFirst()).isTrue();
    assertThat(page.hasNext()).isTrue();



}

```





### 벌크성 수정 쿼리

* ex) 모든 회원의 나이 += 1
* 