---
layout: single
title: redirect for PHP header() vulnerability (Open redirect)
categories: web
tag: [php, function, redirect, open redirect]
toc: true
author_profile: false
sidebar:
    nav: "counts"
---
# 해당 취약점은 무엇인가?

>Redirection 후 PHP가 지속적으로 실행되는 취약점이라고 볼 수 있으며, 해당 취약점은 특정 웹 사이트에서 Redirection을 처리하는 방식의 오류로 발생된다.

# 해당 취약점으로 인한 피해는?  
- 인증 및 인가 되지 않는 퓨ㅔ이지에 대한 Access 및 웹 사이트에 대한 손상이 발생할 수 있다.
<aside class="callout" style="background-color: #f5f5f5; border: 1px solid #ddd; padding: 10px; border-radius: 5px; color: #000;">
  해당 취약점은 디렉터리인덱싱(URL 접근제한 미흡)및 Open Redirect와 유사하다고 볼 수 있다.
</aside>
***

## 어떻게 발생하는가?
해당 취약점은 PHP에서 header() 함수를 사용하여 redirection을 수행 하게 되는데, 이 때 header() 함수는
클라이언트에게 요청을 보낸 후 redirect할 URL을 알려주게 된다.
이 경우 header() 함수 뒤에 exit() 함수가 함께 존재하지 않으면, PHP는 페이지콘텐츠를 보내고 나서야 Client가 새 페이지로 
redirect하게 된다. 즉, clinent는 페이지의 컨텐츠를 받기 전에 redirection을 수행하지 못하게 된다.
```javascript
header("Location: /test.php")
```
1. 클라이언트에서 요청이 이루어지게 되면 응답 헤더에 Location: 헤더가 추가된다.
2. Location: 헤더는 요청을 보낸 후 redirect할 URL을 알려준다.
3. 이 떄 클라이언트측에서 인증 및 인가되지 않은 페이지에 대한 URL(웹 사이트의 다른 페이지)로 설정하여 요청한다.
4. 요청을 보내면 클라이언트가 새 페이지로 Redirection 되지만, PHP는 페이지 콘텐츠를 계속 실행하게 된다.
***