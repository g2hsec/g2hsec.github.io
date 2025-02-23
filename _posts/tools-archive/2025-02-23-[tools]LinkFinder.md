---
layout: single
title: LinkFinder
categories: Tools
tag: [LinkFinder, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# LinkFinder

> 많은 웹사이트에서 사용되는 JavaScript 파일 내부에 숨겨진 link, API Endpoint, URL 등을 추출해주는 도구이다. LinkFinder를 통해 코드 내에 하드코딩과 같이 민감 정보가 노출되었는지 파악할 수 있으며, 공격 표면을 보다 빠르게 탐색할 수 있다.


## 설치 방법

```
git clone https://github.com/GerbenJavado/LinkFinder.git
cd LinkFinder
pip install -r requirements.txt
```

## 기본적인 사용 방법

```
python3 linkfinder.py -i example.js -o cli

-i 옵션을 통해 파일을 지정하고 -o cli를 통해 콘솔창에 출력한다.
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
1. 정규 표현식 기반의 Link, URL, API EndPoint 추출출
2. 다양한 출력 포맷 지원원
3. 빠르고 효율적인 코드 분석석