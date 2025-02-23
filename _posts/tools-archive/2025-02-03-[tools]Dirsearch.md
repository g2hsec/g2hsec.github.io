---
layout: single
title: Dirsearch
categories: Tools
tag: [Dirsearch, attacks, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# Dirsearch 란 무엇인가? 

> 웹 기반의 침투테스트시 개발자 실수 혹은 의도적으로 숨겨진 디렉터리 및 파일들이 존재한다. 이러한 디렉터리 및 파일들은 민감한 정보를 포함하고 있으며, 악의적인 사용자에게 노출될 수 있다. Dirsearch는 이러한 웹 디렉토리 및 파일을 자동으로 스캔하며, 다양한 HTTP 메서드를 지원하고 멀티스레딩을 이용하여 빠른 탐ㅅ객이 가능하다.


## 설치 방법

```
Kali Linux에서는 기본적으로 Dirsearch가 내장되어 있다.

sudo apt update
git clone https://github.com/maurosoria/dirsearch.git
cd dirsearch
pip3 install -r requirements.txt
dirsearch -h
```

## 기본적인 사용 방법

```
python3 dirsearch.py -u http://example.com

```

## Example

***-w 옵션을 사용하여 사용자 지정 사전 파일을 사용할 수 있다.***

```
python3 dirsearch.py -u http://example.com

'''

[200] http://example.com/admin/
[403] http://example.com/private/
[301] http://example.com/images/ -> http://example.com/images/index.html

```
<br>
이 외에도 다양한 옵션과 함께 사용이 가능하다.
<br>

1. 특정 확장자 검색색

    ```
    python3 dirsearch.py -u http://example.com -e php,html,txt
    ```

2. 멀티스레딩 사용

    ```
    python3 dirsearch.py -u http://example.com -t 50
    ```

3. 특정 HTTP 상태 코드 필터링

    ```
    python3 dirsearch.py -u http://example.com --exclude-status=403
    ```

4. 출력 결과 저장

    ```
    python3 dirsearch.py -u http://example.com -o results.txt
    python3 dirsearch.py -u http://example.com -o results.json --format json
    ```

5. User-Agent 변경

    ```
    python3 dirsearch.py -u http://example.com --user-agent "User-Agent 변경 값"
    ```
    
6. 프록시를 이용한 우회

    ```
    python3 dirsearch.py -u http://example.com --proxy http://127.0.0.1:8080
    ```


이 외에도 -h 옵션을 통해 다양한 옵션 확인이 가능하며, -ls 옵션을 통해 출력되는 다양한 공개 API를 출력 하여 --sources 옵션을 통해 특정 소스만 사용하거나, <br>
무료 버전이 아닌 유료 버전을 사용하기 위해 API Key 를 등록하여 다양한 데이터를 확보할 수도 있다.

<br>

# 특징
1. 멀티스레딩 지원
2. 브루트포스 및 사전대입 기능능
3. 다양한 HTTP 옵션 및 프록;시 기능 지원