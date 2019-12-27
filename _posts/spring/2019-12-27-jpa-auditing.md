---
title: "@JpaAuditing을 통한 Entity 중복제거"
date: 2019-12-24
categories:
  - JPA
tags:
  - "#JPA"
---

## @JpaAuditing이란

JPA를 사용할 때 모든 Entity들에 중복으로 사용되는 속성들이 있다. (등록시간, 수정시간)  
이 공통 부분들을 Auditing Entity를 사용하여 공통처리하여 제거할 수 있다.

## Auditing Entity

공통적인 속성들을 모아 추상클래스를 정의한다.

```java

@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {
    @Column(name = "REG_DATE")
    @CreatedDate
    private LocalDateTime regDate;

    @Column(name = "MOD_DATE")
    @LastModifiedDate
    private LocalDateTime modDate;
}


```

-이 클래스를 단독으로 사용할 일이 없기 때문에, 방어적으로 abstract로 선언하여 실수를 줄인다.

- `@MappedSuperclass` : 기본적으로 엔티티는 엔티티만 상속받을 수 있기 때문에 엔티티가 아닌 클래스를 상속받기 위해 사용한다.
- `@EntityListners` : DB적용 전 공통부분을 등록하기 위해 등록한다.

## @EnableJpaAuditing

JpaAuditing을 설정한다.

```java
@EnableJpaAuditing
@Configuration
public class JpaAuditingConfig {

}
```

## 활용

모든 설정 및 준비는 끝났다. 이제 공통속성을 정의한 추상클래스를 상속받아서  
Entity를 만들면 된다.

```java
public class Member extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long memberSeq;

    private String memberId;

    private String password;
}
```
