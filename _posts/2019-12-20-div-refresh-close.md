---
layout: post
title:  "브라우저창 새로고침/닫기 구분하기"
author: hyun
categories: [springboot, javascript]
---
<!-- image: {경로} -->
<!-- rating: {0~5} -->

후배의 이슈를 함께 보던 중에 `브라우저를 닫을 때, 로그아웃 처리해주세요.` 라는 요구사항이 있었다.  
처음에 `unload` 이벤트로 잡으면 되지 않을까? 로 접근했었는데,  
`페이지 이동/새로고침/닫기` 동작시 모두 unload 에 잡혀서 구분이 안되는 문제가 있었다.


다른 방식으로 스크립트에서 처리해보려고 했지만,  
**브라우저를 닫았을 때는 클라이언트가 중단되버려서 이후 로직을 수행하지 않았다.**  
결국 API에서 flag 로 구분하는 식으로 처리하는 식으로 해결하였다.

``` java
@RestController
public class TestApiController {
  boolean isClose = false;
  @GetMapping("/kill")
  public void kill(boolean isClose) throws Exception {
    System.out.println("isClose: " + isClose);
    this.isClose = isClose;

    if ( !isClose ) {
      System.out.println("refresh...");
      return;
    }

    Thread.sleep(5000);
    if ( this.isClose ) {
      System.out.println("kill session!!!");
    }
  }
}
```
`unload` 시에 무조건 `isClose` 값을 true 로 전달하게 하고,  
페이지 로드시에 `isClose` 값을 false 로 전달하게 하면,  
새로고침 시에는 로그아웃 처리를 하지 않게 된다.


브라우저를 닫았을 경우에는***(클라이언트 죽음!!!)*** 페이지 로드를 하지 않으므로 `isClose` 값을  
false 로 전달하지 않는다.


즉, 정해진 딜레이(ex: 5초) 뒤에 로그아웃 처리를 하게 된다.


전체 테스트 코드  
<https://github.com/kdh0981/division-refresh-close>


> 더 좋은 방법이 있다면 자유롭게 의견 부탁드립니다. :D