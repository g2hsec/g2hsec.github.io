---
layout: single
title: (bWAPP)XSS - Reflected (JSON)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(JSON)-archive/image.png)

- 입력 값은 JSON 형식으로 넘어가는 듯 함

```javascript
 <script>

        var JSONResponseString = '{"movies":[{"response":"HINT: our master really loves Marvel movies :)"}]}';

        // var JSONResponse = eval ("(" + JSONResponseString + ")");
        var JSONResponse = JSON.parse(JSONResponseString);

        document.getElementById("result").innerHTML=JSONResponse.movies[0].response;

</script>
```

- 코드를 보게되면, JSONResposeString 변수에 저장되어 있는 값(JSON)을 불러와 JSONResponse 변수에 담는다.
- 이후 result라는 id를 가진 요소에 innerHTML을 사용하여 JSONResponse 변수의 movies키 값의 0번째 데이터를 가져오고 있다.

Reflected 보다는 DOM Based에 가까워 보이며, 우선 입력값을 주어 코드가 어떤 형식으로 변환되는지 확인 했다.

```javascript
    <script>

        var JSONResponseString = '{"movies":[{"response":"a??? Sorry, we don&#039;t have that movie :("}]}';

        // var JSONResponse = eval ("(" + JSONResponseString + ")");
        var JSONResponse = JSON.parse(JSONResponseString);

        document.getElementById("result").innerHTML=JSONResponse.movies[0].response;

    </script>
```

- response 키 값에 대응 되는 데이터 부분이 역시 바뀌는 걸 알 수 있다.
- \<script\> 태그 문에서 작동하므로 해당 태그를 강제적으로 닫고 다른 \<script\> 태그를 생성하여 XSS 진행이 가능하다.

```javascript
</script><script>alert(1)</script>
```
## Level - Medium & High

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image-2.png)
- Medium Level에서는 잘못된 보안 대책을 구현하여 문제가 된다.
- addslashes 함수를 사용한 것으로 확인되며,
- 해당 함수는 특수 문자 앞에 “/” 백슬래쉬를 추가하여 해당 특수 문자를 이스케이프 처리를 위한 함수이다.
- 하지만 이 때 포함되는 특수 문자는 따옴표(’), 쌍따옴표(”), 백슬래쉬(/). null문자 가 포함되어
- XSS 공격에 사용되는 <> 와 같은 특수문자는 필터링되지 않는다.

![그림 1-4](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image-3.png)
- High Level에서는 htmlsepcialchars 함수를 사용하여, 각각의 특수문자에 대한 HTML 엔터티 인코딩을 통한 필터링을 적용하여, XSS 에 대한 대응이 이루어지고 있다.