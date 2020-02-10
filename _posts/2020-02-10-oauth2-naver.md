---
layout: post
title:  "Springboot Naver(네아로) OAUTH2 연동시 yaml 설정 에러 해결"
author: hyun
categories: [Springboot]
---
<!-- image: {경로} -->
<!-- rating: {0~5} -->

OAUTH2 연동시 네이버는 Spring Security를 공식적으로 지원하지 않아서,
기본 설정을 따로 추가해줘야 한다.

ex) application-oauth.yml
```
spring:
  security:
    oauth2:
      client:
        registration:
          naver:
            client-id: hNl_8gLDTiXQhlvqI0az
            client-secret: dSYZucHUgW
            scope: name,email,profile_image
            redirect_uri_template: '{baseUrl}/{action}/oauth2/code/{registrationId}'
            authorization_grant_type: authorization_code
            client-name: Naver

        provider:
          naver:
            authorization_uri: https://nid.naver.com/oauth2.0/authorize
            token_uri: https://nid.naver.com/oauth2.0/token
            user-info-uri: https://openapi.naver.com/v1/nid/me
            user_name_attribute: response
```

위처럼 진행할 때, `OAuth2ClientPropertiesRegistrationAdapter` 에서 
**redirectUriTemplate cannot be empty** 라는 에러가 발생하였다.


분명히 redirect_uri_template 을 프로퍼티에 명시하였는데?? 에러 지점을 찾아가보니
`OAuth2ClientPropertiesRegistrationAdapter` 에서
```map.from(properties::getRedirectUri).to(builder::redirectUriTemplate);```
이런식으로 매핑하고 있었다...


`redirectUri` 로 매핑하면서 메세지는 왜 헷갈리게 `redirectUriTemplate` 로 띄운 것일까..
위 설정에서 아래와 같이 수정하면 정상적으로 동작한다.
``` redirect_uri:  '{baseUrl}/{action}/oauth2/code/{registrationId}' ```


참고로 yaml 에서는 `/` (슬러시)를 그대로 쓰면 파싱 에러가 난다.
따옴표나 작은 따옴표로 감싸주도록 하자.


