---
layout: single
title: LinkFinder
categories: Tools
tag: [LinkFinder, attacks, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# LinkFinder 란 무엇인가? 

> JSleak는 LinkFinder과 유사하며, JavaScript에서 의도치 핞게 노출되는 민감 정보와 같은 부분을 탐색하며, API Key, Token, password, URL파라미터등을 포함한 민감 정보를 탐지하는 도구이다.

다양한 정규식 패턴을 사용하여 탐지하며, 해당 정규식은 아래에서 가져와 사용이 가능하다.<br>
https://github.com/byt3hx/jsleak<br>
뭘 사용할지 애매하다면 https://raw.githubusercontent.com/mazen160/secrets-patterns-db/refs/heads/master/datasets/trufflehog-v3.yaml 을 사용하면 된다.

## 설치 방법

```
go version 1.15, 1.16
 -> go get github.com/channyein1337/jsleak

go version 1.17+
 -> go install github.com/channyein1337/jsleak@latest
```

## 기본적인 사용 방법 (  선택한 정규식 파일로 실행 )

```
echo "http://testphp.vulnweb.com/" | jsleak -t trufflehog-v3.yaml -s
```

<br>
이 외에도 다양한 옵션과 함께 사용이 가능하다.
<br>

1. 비밀 정보 탐색

    ```
    (-t 옵션을 통한 파일 지정은 생략이 가능하다.)
    echo http://testphp.vulnweb.com/ | jsleak  -t secret.yaml -s
    ```

2. Link 탐색

    ```
    echo http://testphp.vulnweb.com/ | jsleak -l
    ```

3. 전체 URL 탐색

    ```
    echo http://testphp.vulnweb.com/ | jsleak -e
    ```

4. 상태값 체크
    
    ```
    echo http://testphp.vulnweb.com/ | jsleak -c 20 -k
    ```

5. 혼합 사용
    
    ```
    echo http://testphp.vulnweb.com/ | jsleak -c 20 -l -s 
    ```

6. 실행중인 URLs 
    
    ```
    cat urls.txt | jsleak -l -s -c 30
    ```

<br>

# 특징
1. 정규표현식 기반 Link, API EndPoint, URL 추출
2. JavaScript 코드 내 민감한 정보 탐색: API 키, 비밀번호, 토큰 등의 포함 여부 확인
3. URL이 살아 있는지 상태 코드 확인