---
layout: single
title: (bWAPP)XSS - Reflected (HREF)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1 ](/assets/image/bwapp/xss/Reflected%20(HREF)-archive/image.png)
- 영화 투표기능이 존재하며, 이름을 입력해야한다.

![그림 1-2](/assets/image/bwapp/xss/Reflected%20(HREF)-archive/image-1.png)
- 투표 가능한 영화 목록이 있으며.
- 입력한 이름은 상단에 출력된다.

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(HREF)-archive/image-2.png)
- 일반적인 스크립트 입력시 문자열 그대로 출력된다.

![그림 1-4](/assets/image/bwapp/xss/Reflected%20(HREF)-archive/image-3.png)
- 투표 클릭시 Redirection 되는 페이지이다.

```html
        <tr height="30">

            <td>G.I. Joe: Retaliation</td>
            <td align="center">2013</td>
            <td>Cobra Commander</td>
            <td align="center">action</td>
            <td align="center"> <a href=xss_href-3.php?movie=1&name=a&action=vote>Vote</a></td>

        </tr>    
```

- 해당 페이지 개발자 도구를 통한 소스코드이다.
- 해당 부분을 보면 a태그의 href 속성을 사용하면서 파라미터값을 함께 주는 걸 확인할 수 있다.
- 이 때 > 로 a태그를 완성시키고 \<script> 태그를 삽입하여 악성스크립트 실행이 가능하다.
- 혹은 a태그 내에 onmouserover 와 같은 이벤트 핸들러를 사용한 스크립트 실행도 가능하다.

```
/bWAPP/xss_href-2.php?name=a><script>alert(1)</script>&action=vote
```

![그림 1-5](/assets/image/bwapp/xss/Reflected%20(HREF)-archive/image-4.png)

- 성공적으로 alert가 실행된다.

## Level - Medium & High

>두 Level 모두 xss_href-3.php 를 호출하는 과정에서 urlencode() 함수를 통한 인코딩이 이루어져 공격이 이루어지지 않는다.