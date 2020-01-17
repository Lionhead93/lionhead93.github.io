---
title: "Travis CI로 지속적인 배포환경 구성"
date: 2019-01-17
categories:
  - Other
tags:
  - "#CI"
  - "#travis"
  - "#deploy"
---

## 테스트 빌드 자동화 과정

1. [Travis CI](https://travis-ci.org/)에 접속하여 Github 계정으로 로그인한다.

2. Github 저장소를 검색하여 활성화시킨다.

3. 프로젝트 설정

```yml
language: java
jdk:
  - openjdk8

branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - "$HOME/.m2/repository"
    - "$HOME/.gradle"

script: "./gradlew clean build"

# CI 실행 완료시 메일로 알람
notifications:
  email:
    recipients:
      - xxx@gmail.com
```

- 권한 문제로 에러 발생 시 추가

  ```yml
  before_install:
    - chmod +x gradlew
  ```

4. commit 후 push 하면 build history/Email에서 성공여부를 확인 할 수 있다.

5. Travis CI 라벨 추가

빌드를 확인했던 화면에서 우측 상단의 build/(unknown) 라벨을 클릭하면 모달이 나온다.  
모달에서 markdown을 선택하고 아래의 코드를 복사하여 라벨이 나오길 원하는 위치(README.md)에 추가한다.

- 테스트와 빌드까지의 자동화를 마쳤지만 아직 배포까지 자동화를 하지못했다.

## AWS Code Deploy 연동

- TravisCI로 Build된 파일을 EC2 인스턴스로 전달하기 위해 AWS CodeDeploy를 사용한다.

- 과정

1. IAM 추가

- AWS IAM 서비스에 사용자를 추가(프로그래밍 방식 엑세스)
- 사용 정책은 기존 정책의 CodeDeploy와 S3 권한만 할당
- 완성되면 엑세스키와 secret키가 생성되고 .csv 파일로 저장한다.

2. S3 버킷 생성

- Build된 jar파일을 보관할 S3버킷을 생성한다.

3. IAM Role 추가

- key를 사용해 기능을 진행 할 IAM Role을 추가한다.(EC2, CodeDeploy)
- IAM - 역할 - 역할 만들기

##EC2

- 사용사례선택 -> EC2
- 권한정책 연결 -> AmazonEC2RoleforAWSCodeDeploy

##CodeDeploy

- 사용사례선택 -> CodeDeploy
- 권한정책 연결 -> AWSCodeDeployRole

4. EC2에 Code Deploy Role 추가

- EC2 인스턴스에서 IAM 역할 연결/바꾸기

5. EC2에 CodeDeploy Agent 설치

- EC2에 ssh 접속 후 aws-cli 설치

```sh

sudo yum -y update
sudo yum install -y aws-cli

```

- 설치 후 인스턴스에서 아래 과정 진행

```sh

cd /home/ec2-user
sudo aws configure

```

region : ap-northeast-2 / output : json 을 입력한다.

- AWS Code Deploy CLI 설치

`aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2`  
실행권한 부여
`chmod +x ./install`
설치 & 실행  
`sudo ./install auto`
Agent 실행 확인
`sudo service codedeploy-agent status`

- EC2 인스턴스가 부팅되면 자동으로 AWS CodeDeploy Agent가 실행될 수 있도록 /etc/init.d/에 쉘 스크립트 파일 생성

`sudo vim /etc/init.d/codedeploy-startup.sh` (리눅스에서 /etc/init.d/ 에 스크립트가 있으면 부팅시 자동실행)

```vim

#!/bin/bash

echo 'Starting codedeploy-agent'
sudo service codedeploy-agent start

```

6. .travis.yml & appspec.yml 설정

AWS S3 : Travis CI가 Build 한 결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간

- travis.yml에 아래 내용 추가

  ```yml
  deploy:
    - provider: s3
      access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
      secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
      bucket: springboot-webservice-deploy # 6-3-3에서 생성한 S3 버킷
      region: ap-northeast-2
      skip_cleanup: true
      acl: public_read
      wait-until-deployed: true
      on:
        repo: (name)/springboot-webservice #Github 주소
        branch: master
  ```

- $AWS_ACCESS_KEY , $AWS_SECRET_KEY 추가

Travis CI의 저장소 Settings - Environment Variables 에 받은 키 값 등록

7. 연동 확인

-현재 까지의 내용을 commit 후 push  
(S3 Bucket에 접근할 수 없어 deploy가 실패 되었다 - Bucket 권한에서 새 ACL에 대한 접근을 허용하니 성공!)

8. 프로젝트 압축하여 Bucket에 올리기

- 아래 설정을 추가

```yml

        ...
before_deploy:
  - zip -r springboot-webservice *
  - mkdir -p deploy
  - mv springboot-webservice.zip deploy/springboot-webservice.zip

deploy:
  - provider: s3
    ...
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    ...

```

9. Travis CI & S3 & CodeDeploy 연동

- AWS CodeDeploy 이동 - 애플리케이션 생성 (컴퓨팅 플랫폼 - EC2/온프레미스 / 배포유형 - 현재위치)

- 모든 설정 후 EC2에 ssh 접속후 zip을 받아올 디렉토리 생성

  ```sh
      mkdir /home/ec2-user/app/travis
      mkdir /home/ec2-user/app/travis/build
  ```

- 프로젝트에 CodeDeploy 설정 파일 작성 (appspec.yml)

  ```yml
  version: 0.0 ##CodeDeploy Version
  os: linux
  files:
    - source: / ##Bucket의 복사할 파일 위치
      destination: /home/ec2-user/app/travis/build/ ##Bucket zip 파일의 압축을 풀 위치
  ```

- .travis.yml 설정 추가

  ```yml
  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: springboot-webservice-deploy # S3 버킷
    key: springboot-webservice.zip # 빌드 파일을 압축해서 전달
    bundle_type: zip
    application: springboot-webservice # 웹 콘솔에서 등록한 CodeDeploy 어플리케이션
    deployment_group: springboot-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹
    region: ap-northeast-2
    wait-until-deployed: true
    on:
      repo: name/springboot-webservice
      branch: master
  ```

- commit & push

10. CodeDeploy로 스크립트 실행

- 스크립트로 배포된 jar파일을 실행까지 진행

- jar파일을 모아둘 디렉토리 생성

  `mkdir /home/ec2-user/app/travis/jar`

- jar를 실행할 deploy.sh 작성

  `vim /home/ec2-user/app/travis/deploy.sh`

  ```sh
  #!/bin/bash

  REPOSITORY=/home/ec2-user/app/travis

  echo "> 현재 구동중인 애플리케이션 pid 확인"

  CURRENT_PID=$(pgrep -f springboot-webservice)

  echo "$CURRENT_PID"

  if [ -z $CURRENT_PID ]; then
      echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
  else
      echo "> kill -15 $CURRENT_PID"
      kill -15 $CURRENT_PID
      sleep 5
  fi

  echo "> 새 어플리케이션 배포"

  echo "> Build 파일 복사"

  cp $REPOSITORY/build/build/libs/*.jar $REPOSITORY/jar/

  JAR_NAME=$(ls $REPOSITORY/jar/ |grep 'spring-webservice' | tail -n 1)

  echo "> JAR Name: $JAR_NAME"

  nohup java -jar $REPOSITORY/jar/$JAR_NAME &

  ```

- deploy.sh 스크립트 확인

  ```sh
    /home/ec2-user/app/travis/deploy.sh
  ```

- appspec.yml 에 설정 추가

  ```yml
  hooks:
    AfterInstall: # 배포가 끝나면 아래 명령어를 실행
      - location: execute-deploy.sh
        timeout: 180
  ```

- CodeDeploy에서 접근할 excute-delpoy.sh 작성

  /프로젝트 루트

  ```sh
   #!/bin/bash
  /home/ec2-user/app/travis/deploy.sh > /dev/null 2> /dev/null < /dev/null &
  ```

- commit & push

- 정리

  local push to Github - test/build(Travis) - deploy to Bucket(Travis) - deploy to EC2(CodeDeploy) - jar 구동 script 실행(CodeDeploy)

* 참고 - 스프링부트로 웹 서비스 출시하기 https://jojoldu.tistory.com/
