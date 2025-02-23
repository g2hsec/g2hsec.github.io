---
layout: git-dumper
title: LinkFinder
categories: Tools
tag: [LinkFinder, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# git-dumper

> 웹 어플리케이션 개발시 개발자의 부주의로 인해 웹 서버에 노출된 .git 디렉토리는 민감한 소스 코드와 히스토리 정보가 외부에 유출될 수 있다. 이러한 노출된 .git 저장소를 자동으로 덤프(추출)하여, 실제로 어떤 정보가 공개되어 있는지 확인할 수 있도록 하는 도구이다.


## 설치 방법

```
pip install git-dumper
pip install -r requirements.txt
```

## 기본적인 사용 방법

```
git-dumper [options] URL DIR
git-dumper http://website.com/.git ~/website
```

<br>
이 외에도 다양한 옵션과 함께 사용이 가능하다.
<br>

```
usage: git-dumper [options] URL DIR

Dump a git repository from a website.

positional arguments:
  URL                   url
  DIR                   output directory

optional arguments:
  -h, --help            show this help message and exit
  --proxy PROXY         use the specified proxy
  -j JOBS, --jobs JOBS  number of simultaneous requests
  -r RETRY, --retry RETRY
                        number of request attempts before giving up
  -t TIMEOUT, --timeout TIMEOUT
                        maximum time in seconds before giving up
  -u USER_AGENT, --user-agent USER_AGENT
                        user-agent to use for requests
  -H HEADER, --header HEADER
                        additional http headers, e.g `NAME=VALUE`
  --client-cert-p12 CLIENT_CERT_P12
                        client certificate in PKCS#12 format
  --client-cert-p12-password CLIENT_CERT_P12_PASSWORD
                        password for the client certificate
```


<br>

# 특징
1. 노출된 .git 디렉토리 확인 및 덤프
2. 저장소의 전체 커밋 히스토리와 객체(Object) 다운로드
3. Git 객체와 메타데이터를 재구성하여 원본 소스 코드 복원