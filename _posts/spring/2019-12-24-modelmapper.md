---
title: "ModelMapper 활용하기"
date: 2019-12-24
categories:
  - Spring
tags:
  - "#Spring"
  - "#ModelMapper"
---

## Entity와 DTO매핑의 문제점

기본적으로 (Client) <-> Controller부터 Service까지 사용하는 DTO,  
Service부터 Repository <-> (DB) 까지 사용하는 Entity간의 변환 매핑 시  
매핑관계 당 하나의 매소드를 만들어 주어야 하고, 데이터에 변경사항이 있으면  
매소드까지 수정해주어야 하는 불편한 상황에 직면하게 됩니다.  

## ModelMapper란

DTO와 Entity의 필드명을 일치시켜주면 매핑을 수행해주는 라이브러리이다.  
(매핑할 class에는 setter가 있어야하며 매핑될 class에는 getter가 있어야한다.)  

`modelMapper.map(student,StudentDto.class)`와 같이 사용하면  
첫번째 인자 Entity가 DTO로 매핑된 return을 받을 수 있다.  

필드명 매핑전략 외에도 여러가지 매핑전략이 있다.