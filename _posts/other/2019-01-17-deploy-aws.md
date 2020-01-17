---
title: "AWS EC2 서버 배포환경 구성하기"
date: 2019-01-17
categories:
  - Other
tags:
  - "#CI"
  - "#AWS"
  - "#deploy"
---

## 과정

1. AWS EC2 인스턴스에 접근한다.

   ```sh
    sudo ssh -i (key) ec2-user@(인스턴스 ip)
   ```

2. git를 설치하고 git clone을 통해 소스를 받아온다.

3. 배포를 위한 스크립트를 작성한다.

   `vi deploy.sh`

   로컬에서 작업 후 저장소에 푸쉬를 하면, ec2서버에서 pull을 한 후 빌드를 다시 하고 run 중인 서버를 새로 빌드된 파일로 재부팅한다.

   ```sh
   #!/bin/bash

    REPOSITORY=/home/ec2-user/app/git

    cd $REPOSITORY/springboot-webservice/

    echo "> Git Pull"

    git pull

    echo "> 프로젝트 Build 시작"

    ./gradlew build

    echo "> Build 파일 복사"

    cp ./build/libs/*.jar $REPOSITORY/

    echo "> 현재 구동중인 애플리케이션 pid 확인"

    CURRENT_PID=$(pgrep -f springboot-webservice)

    echo "$CURRENT_PID"

    if [ -z $CURRENT_PID ]; then
        echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
    else
        echo "> kill -2 $CURRENT_PID"
        kill -2 $CURRENT_PID
        sleep 5
    fi

    echo "> 새 어플리케이션 배포"

    JAR_NAME=$(ls $REPOSITORY/ |grep 'springboot-webservice' | tail -n 1)

    echo "> JAR Name: $JAR_NAME"

    nohup java -jar $REPOSITORY/$JAR_NAME &
   ```

4. 이제 작업을 하고 저장소에 push 후 ec2 서버에서 deploy.sh 스크립트를 실행시켜주기만 하면 배포가 완료된다.

- TravisCI 적용 예정
