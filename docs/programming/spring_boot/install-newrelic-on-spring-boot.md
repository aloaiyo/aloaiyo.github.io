---
layout: minimal
title: Java Spring Boot 서버에 Newrelic (APM) 설치하기
grand_parent: programming
parent: spring_boot
nav_order: 1
keywords:
    - runbook
    - newrelic
slug: java-spring-boot-서버에-newrelic-apm-설치하기
---

# Java Spring Boot 서버에 Newrelic (APM) 설치하기

## 목표

Java Spring Boot 서버에 Newrelic을 설치하여 APM을 통해 transaction 과 error 를 모니터링한다.

## 배포 상황

여러 repository가 있으며 아래의 두 방식을 혼용하여 쓴다고 가정한다

- Docker on EC2
- ECS & Code Deploy

## 설치

회원 가입 후 newrelic에서 제공하는 guide를 순서대로 따른다.

- APM 선택 후 Java 선택

![언어 선택](/assets/images/install-newrelic-on-spring-boot.jpeg)

- Begin installation 버튼 클릭

![Begin installation 버튼 클릭](/assets/images/install-newrelic-on-spring-boot1.jpeg)

- 우측 하단 Other Java installation options 클릭

![우측 하단 Other Java installation options 클릭](/assets/images/install-newrelic-on-spring-boot2.jpeg)

- 1번에 앱 이름을 적는다.
- 2번 Download 버튼을 눌러 newrelic.yml 파일을 다운로드 받는다.

![2번 Download 버튼을 눌러 newrelic.yml 파일을 다운로드 받는다.](/assets/images/install-newrelic-on-spring-boot3.jpeg)

- 3번 Copy to clipboard 를 눌러 curl 명령어를 copy 하고 shell 에서 paste 하여 newrelic-java.zip 을 다운로드 받는다.

![3번 Copy to clipboard 를 눌러 curl 명령어를 copy 하고 shell 에서 paste 하여 newrelic-java.zip 을 다운로드 받는다.](/assets/images/install-newrelic-on-spring-boot4.jpeg)

- 4번 Spring Boot 를 선택한다
- 이후 단계는 무시해도 된다. 6번의 경우 Docker의 condition을 check 하고 싶다면 AWS console 에 접속하여 명령어를 입력한다. 하지만, newrelic은 월 무료 100G가 넘으면 1G당 $0.25을 내야하기 때문에 다른 모니터링으로 확인하는 것을 추천한다.

![4번 Spring Boot 를 선택한다](/assets/images/install-newrelic-on-spring-boot5.jpeg)

![무시해도 된다](/assets/images/install-newrelic-on-spring-boot6.jpeg)

- 받은 newrelic-java.zip을 압축 푼 뒤 Spring project 의 root 바로 아래에 둔다. 이 때, newrelic 폴더 아래에 jar들이 배치되도록 폴더명 변경등을 한다.

![newrelic 폴더 아래에 jar들이 배치되도록 폴더명 변경등을 한다.](/assets/images/install-newrelic-on-spring-boot7.jpeg)

- git push 하여 repository에 반영한다. 대부분의 개발자는 jar 파일을 git ignore로 두므로 force add 한다.

```bash
git add <blar>.jar -f
```

- 배포를 한 번 진행하여 서버에 newrelic 파일이 올라가도록 한다.

## Docker on EC2 일 경우

- 본인의 시스템에 있는 docker-compose.yml을 열고 아래와 같이 수정한다.
  - entrypoint 에 -javaagent:/newrelic/newrelic.jar 를 추가하였다. -javaagent는 -jar보다 반드시 명령어가 앞에 있어야 한다.
  - volumes에 ./newrelic:/newrelic/ 를 추가하였다. 위의 entrypoint 에서 javaagent를 실행하는 부분에서 -javaagent:/app/newrelic/newrelic.jar 를 하면 mount 할 필요없다.

```yaml
maven:
    build:
      context: ./
      dockerfile: Dockerfile
      args:
        PROCESS: test
    environment:
      - PROCESS=test
    container_name: maven
    expose: [ 8080 ]
    ports:
      - 8080:8080
    entrypoint: java -javaagent:/newrelic/newrelic.jar -Dnewrelic.environment=test -jar -DSpring.profiles.active=test /app/target/test-0.0.1-SNAPSHOT.jar
    depends_on:
    volumes:
      - ./:/app
      - ./newrelic:/newrelic/
```

- git push 후 배포를 한다.

## ECS & Code Deploy 의 경우

- AWS의 ECS로 이동하여 작업 정의를 클릭한다
- 뉴렐릭을 적용할 서버를 선택하고 새 개정 생성 을 누른다. (개정은 영어로 revision 이다)

![ECS 작업 정의](/assets/images/install-newrelic-on-spring-boot8.jpeg)

- 새 개정 생성 페이지에서 제일 하단으로 이동하여 볼륨에 "JSON을 통한 구성" 버튼을 누른다

!["JSON을 통한 구성" 버튼을 누른다](/assets/images/install-newrelic-on-spring-boot9.jpeg)

- JSON에서 command 부분을 아래와 같이 바꾼다. EC2를 사용할 때 entrypoint 에 적던 것과 동일하다. 바꾼 뒤 저장 버튼을 누른다

*mount를 시도하였지만 뜻대로 되지 않아 포기하였다.*

```yaml
"command": [
  "java",
  "-javaagent:/app/newrelic/newrelic.jar",
  "-jar",
  "-DSpring.profiles.active=release",
  "/app/target/test-0.0.1-SNAPSHOT.jar"
],
```

![command 변경](/assets/images/install-newrelic-on-spring-boot10.jpeg)

- 생성 버튼을 눌러 새 개정을 만든다.
- 배포를 한다.

## 결과

![결과](/assets/images/install-newrelic-on-spring-boot11.jpeg)
