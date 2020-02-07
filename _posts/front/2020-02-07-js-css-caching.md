---
title: "js/css 파일 캐싱 방지"
date: 2020-02-07
categories:
  - web
tags:
  - "#caching"
---

## Browser Caching

브라우저는 기본적으로 네트워크를 통한 리소스요청은 기존의 리소스를 캐싱했다가 재사용한다.

개발 중 js/css 파일을 수정하여 확인하는 단계에서 수정한 내용이 반영이 되지않아  
브라우저 내의 캐시를 삭제하고 다시 요청하는 방법은 매우 번거롭게 느껴졌다.

수정 후 요청 리소스 뒤에 query string을 붙혀주면 브라우져에서 새로운 리소스로 인식하여 캐싱된 리소스가 아닌  
새롭게 요청한 리소스를 가져오게 된다.

```js
  //before
  <script scr="./src/common.js"></script>
  //after
  <script scr="./src/common.js?v=1.1"></script>
```
