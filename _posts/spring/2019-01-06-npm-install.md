---
title: "npm install 개발용 라이브러리와 배포용 라이브러리"
date: 2019-01-06
categories:
  - Node.js
tags:
  - "#node"
  - "#npm"
---

## npm install

- NPM 전역 설치

  NPM 전역 설치는 위와 같이 프로젝트에서 사용할 라이브러리를 불러올 때 사용하는 것이 아니라  
  시스템 레벨에서 사용할 자바스크립트 라이브러리를 설치할 때 사용합니다.

  ```sh
  npm install gulp --global
  ```

- NPM 전역 설치 경로

  이렇게 설치된 자바스크립트 라이브러리는 어느 위치에서 해당 명령어를 실행했던지 간에 OS별로 아래와 같은 폴더 경로에 설치됩니다.

  ```sh
  # window
  %USERPROFILE%\AppData\Roaming\npm\node_modules

  # mac
  /usr/local/lib/node_modules
  ```

- NPM 지역 설치

  NPM 지역 설치에 자주 사용되는 2가지 옵션은 다음과 같습니다.

  ```sh
  npm install jquery (--save-prod)
  npm install jquery -D(--save-dev)
  ```

  여기서 설치 옵션에 아무것도 넣지 않은 `npm i jquery`는 package.json의 dependencies에 등록됩니다.

  ```json
  // package.json
  {
    "dependencies": {
      "jquery": "^3.4.1"
    }
  }
  ```

  설치 옵션으로 `-D`를 넣은 경우에는 해당 라이브러리가 package.json의 devDependencies에 등록됩니다.

  ```json
  // package.json
  {
    "devDependencies": {
      "jquery": "^3.4.1"
    }
  }
  ```

- 개발용 라이브러리와 배포용 라이브러리

  설치된 배포용 라이브러리는 npm run build로 빌드를 하면 최종 애플리케이션 코드 안에 포함됩니다.

  그런데 만약 반대로 설치 옵션에 -D를 주었다면 해당 라이브러리는 빌드하고 배포할 때 애플리케이션 코드에서 빠지게 됩니다.  
  따라서, 최종 애플리케이션에 포함되어야 하는 라이브러리는 -D로 설치하면 안 됩니다.  
  개발할 때만 사용하고 배포할 때는 빠져도 좋은 라이브러리의 예시는 다음과 같습니다.

  - 빌드도구( `webpack` )
  - 코드 문법 검사 도구( `eslint` )
  - 이미지 압축 도구( `imagemin` )
