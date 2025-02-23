---
layout: single
title: Waybackurls
categories: Tools
tag: [Waybackurls, attacks, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# Waybackurls 란 무엇인가? 

> 많은 웹서버들은 다양한 업데이트와 변화를 거쳐 지속적으로 변화한다. 이는 과거에 존재했던 URL에서 디렉토리 구조, 파일, 민감한 데이터등의 변화를 알 수 있다면 침투테스트 과정에서 유의미한 정보가 될 것이다. Waybackurls는 Wayback Machine(웹 아카이브) 에서 과거에 저장된 URL을 추출하는 도구이다. 이를 통해 웹 사이트의 이전 버전에서의 숨겨진 리소스,, 파일을 찾는데 도움이 될 수 있다.


## 설치 방법

```
sudo apt update
sudo apt install golang
go install github.com/tomnomnom/waybackurls@latest
```

## 기본적인 사용 방법

```
waybackurls http://example.com
```

## Example

```
waybackurls http://example.com 

'''

http://example.com/admin/
http://example.com/login/
http://example.com/images/

```
<br>
이 외에도 다양한 옵션과 함께 사용이 가능하다.
<br>

1. 여러 도메인 처리

    ```
    cat domains.txt | waybackurls
    ```

2. URL 필터링

    ```
    waybackurls http://example.com | grep .php
    ```

3. 멀티스레딩 사용

    ```
    waybackurls -t 50 http://example.com
    ```



<br>

# 특징
1. 웹 아카이브를 통해 과거 URL을 자동 수집
2. 특정 도메인의 과거 리소스 검색
3. 다양한 URL 분석 및 멀티스레딩 지원원