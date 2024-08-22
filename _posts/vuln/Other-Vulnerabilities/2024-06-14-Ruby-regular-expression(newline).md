---
layout: single
title: Ruby-regular-expression(newline)[\n]
categories: Other-Vuln
tag: [Ruby, regular expression, \n, newline]
toc: true
author_profile: false
---

# Ruby Regular expression bypass
> Ruby를 포함한 몇몇 언어에서는 정규표현식을 제공한다. 기본적으로 정규표현식은 "한줄모드"로 동작하며, 개발자들이 백엔드에서 특정 로직을 구현할시 ^와 $를 사용하여 문자의 시작과 끝을 정해놓는 경우가 존재한다. 

<span style="font-weight: bold; color: red;">\n(newline) 문자는 개행을 나타내며, 해당 문자가 존재할 경우, 물리적으로 여러 줄로 나누게된다. 이 때 한줄 모드로 검증을 수행하는 정규표현식 검사를 우회할 수 있게된다.
 </span>

 <div class="notice">
  ^[0-9a-z ]+$/i 와같이 특수문자를   배제하여 웹 취약점을 차단하려 하여도, a\n<$= > 와 같이 우회하여 특수문자가 삽입이 가능해진다. 혹은, url인코딩을 통해 a%5c%6e%3c%24%3d%20%3e와 같이 사용할 수 있다.  
</div>

즉, 개행 문자(\n)를 사용하면 정규 표현식의 ^와 $가 전체 문자열이 아니라 각 줄의 시작과 끝으로 매칭되어, 따라서 원래 의도한 대로 문자열 전체를 검사하지 않게 되며, 필터링을 우회할 수 있다.