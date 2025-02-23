---
layout: single
title: subfinder
categories: Tools
tag: [subfinder, attacks, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# subfinder 란 무엇인가? 

> 웹 기반의 침투테스트시 서브도메인 즉, 하위도메인 정보를 수집해주는 도구이다. 많은 기업과 조직은 주요 도메인 이외에 다양한 서브(하위)도메인을 운영하고 있으며, 이중 일반적으로 공개하지 않는 민감한 앤드포인트가 존재하는 도메인이 존재할 수 있다. 이러한 다양한 서브도메인을 효과적으로 찾을 수 있도록 도와주는 OSINT 기반의 도구이다.

💡 **<u>Subfinder는 **Active 방식(직접 스캔)**이 아닌 **Passive 방식(공개된 데이터를 활용)**을 사용하여 타겟에 영향을 주지 않고도 많은 서브도메인을 수집할 수 있다.</u>** 
{: .notice--primary} 

## 설치 방법

```
sudo apt update
sudo apt install subfinder -y

혹은

go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

subfinder -h

```

## 기본적인 사용 방법

```
subfinder -d example.com
```

## Exampleㅈ

```
subfinder -d example.com

'''

blog.example.com
mail.example.com
vpn.example.com
dev.example.com
```
<br>
이 외에도 다양한 옵션과 함께 사용이 가능하다.
<br>

1. 특정 포멧으로 출력

    ```
    subfinder -d example.com -o results.txt
    subfinder -d example.com -o results.json -oJ
    ```

2. 여러 도메인 동시 실행

    ```
    subfinder -dL domains.txt -o results.txt

    ```

이 외에도 -h 옵션을 통해 다양한 옵션 확인이 가능하며, -ls 옵션을 통해 출력되는 다양한 공개 API를 출력 하여 --sources 옵션을 통해 특정 소스만 사용하거나, <br>
무료 버전이 아닌 유료 버전을 사용하기 위해 API Key 를 등록하여 다양한 데이터를 확보할 수도 있다.

<br>

# 특징
1. 간단한 명령어와 이와 상반되는 빠른 탐색
2. API 활용시 더욱 강력한 데이터 수집 가능
3. Passive방식을 통해 타겟에 영향을 주지 않고 안전하게 정보 수집 가능