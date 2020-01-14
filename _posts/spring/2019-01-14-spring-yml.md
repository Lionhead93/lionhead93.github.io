---
title: "@ConfigurationProperties, @PropertySource 그리고 yml"
date: 2019-01-06
categories:
  - Spring
tags:
  - "#Spring"
  - "#Yaml"
---

## @PropertySource @ConfigurationProperties

하나의 프로젝트를 시작할 때, 항상 고민하게 되는 부분 프로젝트를 구성하는 설정들을 어떻게 관리하냐는 점이다.

- .yml 사용

  환경( `profile` )에 따라 다른 설정 값을 하나의 파일에서 구분지어 관리할 수 있다는 점에서 좋다.

  ```yml
  server:
  port: 9092

  spring:
  profiles: dev
  datasource:
    driver-class-name: jdbcdriver
    url: url
    username: name
    password: password
  ---
  server:
  port: 9093

  spring:
  profiles: test
  datasource:
    driver-class-name: jdbcdriver
    url: url
    username: name
    password: password
  ```

- @ConfigurationProperties

  스프링 부트는 application.properties 파일을 이용해서 설정을 제공한다. 이 파일에는 부트가 제공하는 프로퍼티뿐만 아니라 커스텀 프로퍼티를 추가할 수 있다. 커스텀 프로퍼티를 사용하는 몇 가지 방법이 있는데 그 중에서 설정 프로퍼티 클래스를 사용하면 관련 설정을 한 클래스에서 관리할 수 있어 편리하다.

  설정 프로퍼티 클래스를 사용하는 방법은 간단하다. 먼저, 설정 프로퍼티 클래스로 사용할 클래스를 작성한다. 이 클래스는 다음과 같이 작성한다.

  - @ConfigurationProperties 애노테이션을 클래스에 적용한다. yml 파일에서 사용할 접두어를 지정한다.
  - yml 파일에 설정한 프로퍼티 값을 전달받을 setter를 추가한다.
  - 프로퍼티 값을 참조할 때 사용할 get 메서드를 추가한다.

  ```java
  @Component
  @ConfigurationProperties(prefix = "test.config")
  @Getter @Setter
  public class TestConfig {
    private String test1;
    private String test2;
  }
  ```

  ```yml
  test:
  config:
    test1: test111
    test2: test222
  ```

  테스트 코드를 작성하여 확인해보면 잘 로드 되는 것을 확인할 수 있다.

  ```java
    @Autowired
    TestConfig testConfig;

    @Test
    void test() {
        System.out.println(testConfig.getTest1());
        System.out.println(testConfig.getTest2());
    }
  ```

- @PropertySource

  `@PropertySource`의 사용법은 아래와 같다.

  ```java
  @Component
  @PropertySource(value = {"classpath:/config/test-config.yml"})
  @Getter @Setter
  public class TestConfig {
    @Value(value = "${test1}")
    private String test1;
    @Value(value = "${test2}")
    private String test2;
  }
  ```
