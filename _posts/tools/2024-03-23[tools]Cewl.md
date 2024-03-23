---
layout: single
title: (tools) Cewl
categories: THM
tag: [wordlist, attacks, tool, tools, pentest, pentest tool,password]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/tools/image.png)

# Cewl이란?

> 특정 사이트 내에서  Password cracking에 사용되는 사전 단어 목록을 만들어 내는 도구이다. 웹 사이트에서 특정한 패턴이나 단어를 추출하여 사용자 정의 단어목록을 생성한다.

## 설치 방법

```shell
sudo apt install cewl
```

## 기본적인 사용 방법

```shell
cewl [옵션] [출력 파일]

ex) cewl -u http://example.com -m4 -d2 -o wordlist.txt
```

## 주요 옵션

| 옵션                  | 설명                                                    |
|-----------------------|--------------------------------------------------------|
| -u 또는 --url         | 목표로 하는 웹 페이지의 URL을 지정.               |
| -m 또는 --min-word-length | 생성된 단어 목록에 포함할 단어의 최소 길이를 지정. |
| -d 또는 --depth       | 단어 목록 생성 중에 동일한 단어를 무시할 반복 횟수를 지정. |
| -x 또는 --exclude     | 생성된 단어 목록에서 제외할 특정 단어를 지정.    |
| -o 또는 --output      | 생성된 단어 목록을 저장할 출력 파일의 경로와 이름을 지정. |
| -H 또는 --headers     | HTTP 요청에 사용할 사용자 정의 헤더를 지정.       |
| -A 또는 --auth        | HTTP 요청에 사용할 사용자 정의 인증 정보를 지정.