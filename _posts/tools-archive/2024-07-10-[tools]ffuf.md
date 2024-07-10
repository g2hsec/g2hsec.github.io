---
layout: single
title: (tools) ffuf
categories: Tools
tag: [ffuf, attacks, tool, tools, pentest, pentest tool,password, shadow]
toc: true
author_profile: false
---

# ffuf 란 무엇인가?

> ffuf는 Go로 작성된 빠른 웹 퍼저로, 일반적인 디렉토리 검색, 가상 호스트 검색(DNS 레코드 없음), 하위도메인 검색, GET 및 POST 매개변수 퍼징을 허용한다.

## 기본적인 사용 방법

```shell
ffuf -u http://example.com/FUZZ -w wordlist.txt
```

## 주요 옵션
1. -u (URL)
- 퍼징할 URL을 지정. URL에서 FUZZ 키워드를 사용하여 퍼징할 위치를 표시합니다.
- 예: ffuf -u http://example.com/FUZZ -w wordlist.txt

2. -w (Wordlist)
- 사용할 단어 리스트 파일을 지정.
- 예: ffuf -w wordlist.txt

3. -o (Output)
- 결과를 파일로 저장. JSON 형식으로 저장되며, 다른 형식도 지원한다.
- 예: ffuf -u http://example.com/FUZZ -w wordlist.txt -o output.json

4. -e (Extensions)
- 퍼징할 파일 확장자를 지정. 여러 확장자를 콤마로 구분하여 입력한다.
- 예: ffuf -u http://example.com/FUZZ -w wordlist.txt -e .php,.html,.txt

5. -mc (Match Codes)
- 특정 HTTP 상태 코드를 매칭하여 결과를 필터링.
- 예: ffuf -u http://example.com/FUZZ -w wordlist.txt -mc 200

6. -fc (Filter Codes)
- 특정 HTTP 상태 코드를 필터링하여 제외.
- 예: ffuf -u http://example.com/FUZZ -w wordlist.txt -fc 404

7. -r (Recursive)
- 디렉터리 발견 시 재귀적으로 퍼징.
- 예: ffuf -u http://example.com/FUZZ -w wordlist.txt -r
    - -recursion-depth (Recursive Depth)
    - 재귀적 퍼징의 깊이를 설정.
    - 예: ffuf -u http://example.com/FUZZ -w wordlist.txt -r -recursion-depth 3

8. -H (Headers)
- 요청에 사용할 HTTP 헤더를 설정.
- 예: ffuf -u http://example.com/FUZZ -w wordlist.txt -H "Host: example.com

9. -fs (Filter Size)
- 응답의 크기를 기준으로 결과를 필터링
- 예 : ffuf -u http://example.com/FUZZ -w wordlist.txt -fs 1234

## Example
### vHOST 탐색
- ffuf -w Wordlist.txt -u http://test.com -H "hots:FUZZ.test.com"

### Dir 탐색
- ffuf -w Wordlist.txt -u http://cozyhosting.htb/FUZZ
