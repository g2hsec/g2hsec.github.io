---
layout: single
title: LinkFinder
categories: Tools
tag: [LinkFinder, attacks, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# LinkFinder 란 무엇인가? 

> 많은 웹 어플레키엿능ㄴ JavaScript로 제작되었으며, 이러한 JavaScript 파일 내부에 숨겨진 Link, API EndPoint, URL등을 자동으로 추출해주는 도구이다. 이를 통해 코드 내에서 민감 정보가 노출되었는지와 같은 공격 표면을 빠르게 탐색할 수 있다.


## 설치 방법

```
git clone https://github.com/GerbenJavado/LinkFinder.git
cd LinkFinder
pip install -r requirements.txt

python3 linkfinder.py -h
```

## 기본적인 사용 방법

```
python3 linkfinder.py -i example.js -o cli

-i를 통해 파일을 지정하고 -o cli를 통해 콘솔에 출력한다.
```

<br>
이 외에도 다양한 옵션과 함께 사용이 가능하다.
<br>

1. 결과를 파일로 저장

    ```
    python3 linkfinder.py -i example.js -o html > results.html
    python3 linkfinder.py -i example.js -o json > results.json
    ```

2. URL 입력 사용

    ```
    python3 linkfinder.py -i https://example.com/app.js -o cli
    ```

3. 폴더 전체 분석

    ```
    python3 linkfinder.py -i "./static/js/*.js" -o cli
    ```

<br>

# 특징
1. 정규표현식 기반 Link, API EndPoint, URL 추출
2. 다양한 출력 포맷: 콘솔(cli), HTML, JSON 등
3. 여러 입력 방식 지원: 단일 파일, URL, 폴더