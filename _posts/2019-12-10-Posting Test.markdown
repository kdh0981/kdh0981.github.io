---
layout: post
title:  "Springboot 외부 logback 파일 참조하기"
date:   2019-12-10 18:04:00 +0900
categories: jekyll update
---

Springboot 서비스 배포시 `logback.xml` 파일을 외부에서 읽도록 처리하는 것을 기존에는 `application.yml` 에 `logging.config = {외부 logback 경로}` 를 추가하는 방식으로 처리하였었는데, 오늘따라 비슷한 환경에서 다른 서비스를 띄우는데 파일을 읽지 못하는 등 에러가 나서 서비스가 죽는 일이 있었다.


IDE에서 실행하면 또 그런 문제가 안생겨서... 난감했었는데, 알아보니 **`executable jar`** 에서만 **'로그 커스터마이징 관련해서 클래스 로딩이 잘 되지 않는 이슈가 있다'**는 것을 spring 공식문서를 통해 알게 되었다.


아래와 같이 jar 실행시 **환경변수 추가**를 하는 방식으로 해결하였다.  
`java -Dlogback.confiurationFile=경로/logback.xml -jar ...`


아래 링크 4.6절 참조  
<https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging>