---
title: "Gadle 과 Maven"
date: 2019-01-13
categories:
  - Build
tags:
  - "#build"
  - "#gradle"
---

## Build 도구

- Maven vs Gradle

  Gradle이 시기적으로 늦게 나온만큼 사용성, 성능 등 비교적 뛰어난 스펙을 가지고있다. ( 최대 100배 속도 )

  - Build라는 동적인 요소를 정의하기엔 Maven의 pom.xml로는 어려운 점이 있다.

    - 설정 내용이 길어진다.
    - 의존관계가 복잡할수록 가독성이 매우 떨어진다.

  - Gradle은 Groovy 기반이기 때문에, 동적인 빌드는 Groovy Script로 직접 해결할 수 있다.

    - Configuration Injection 방식을 사용해서 공통 모듈을 상속해서 사용하는 단점을 커버했다.
    - 설정 주입 시 프로젝트의 조건을 체크할 수 있어서 프로젝트별로 주입되는 설정을 다르게 할 수 있다.
