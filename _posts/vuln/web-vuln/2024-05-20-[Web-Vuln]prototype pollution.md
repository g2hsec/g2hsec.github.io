---
layout: single
title: prototype pollution
categories: Web-vuln
tag: [prototype pollution, prototype, web, web vulnerability, injection]
toc: true
author_profile: false
---



# prototype 이란?

> Javascript는 프로토타입 기반 언어라 불리며, prototype이란 Javascript에서 객체를 상속하기 위해 사용되는 방식을 의미한다. 자바스크립트의 함수는 기본적으로 객체이며, 모든 함수는 자동으로 prototype 이라는 속성을 가지고 있다. 이 때 새로운 객체를 만들 때 객체의 prototype역할을 하게된다. 즉, 프로토타입 객체는 상위 프로토타입 객체로부터 메소드와 속성을 상속 받을 수 있고, 그 상위 프로토 타입 객체도 마찬가지이며, 이를 프로토 타입 체인이라 부른다.

```javascript
function Person(first, last, age, gender, interests) {
}

var person1 = new Person("Tammi", "Smith", 32, "neutral", [
  "music",
  "skiing",
  "kickboxing",
]);

Person.prototype.farewell = function () {
  alert(this.name.first + " has left the building. Bye for now!");
};
```

해당 코드와 같이 Prototype를 사용하여 수정이 가능하다. 
Person이라는 메소드를 정의하고, person1이라는 객체를 생성하게 된다. 이 때 person1이 프로토타입 객체가 된다. 그 후 prototype속성에 farewell이라는 새로운 메소드를 추가시켰다. prototype에 새로운 메소드를 추가하면 동일한 생성자로 생성된 모든 객체에서 추가된 메소드를 바로 사용할 수 있가 된다.
<br><br>
즉, person1 객체에서 farewell() 메서드를 사용할 수 있게된다. 이는 prototype객체는 모든 인스턴스에서 공유하기 때문에 정의하는 즉시 별도의 갱신 과정 없이 접근이 가능하다.
<hr>
javascript에서는 개체의 속성을 참조할 때 javascript 엔진은 우선 개체 자체에서 해당 속성에 직접 액세스 하려고 시도하며, 객체에 일치하는 속성이 없으면 javascript엔진은 대신 객체의 prototype에서 해당 속성을 찾는다. 

> a 라는 객체의 프로토 타입에 b메서드를 추가하게 되면, a 생성자로 생성된 모든 객체에서 참조할 수 있게 된다.

![그림 1-1](/assets/image/vuln/web-vuln/Prototype-Pollution/image.png)
<br>
비어있는 객체를 만든 후 해당 객체의에 대해 정의된 속성이나 메서드가 없더라도 object.prototype의 내장된 속성 및 메서드가 존재 하는 것을 볼 수 있다.


```javascript
String.prototype.upper = function(){}
let searchstring = "TEST"
searchstring.upper()
```

위와 같은 코드가 조재하면 String 객체의 prototype에 upper메서드를 추가하므로, String 객체의 속성 및 메서드를 참조 할수 있다.



## Prototype Chain

객체의 프로토타입은 또 다른 새로운 객체이며, 자체 프로토타입또한 가지고 있어야 한다. 즉, 프토토타입 체인이라 불리는 상위 객체로부터 속성 및 메소드를 상속 받으며, 또 다른 새로운 객체를 만들게 되면 새롭게 생성된 객체 또한 상위 객체로부터 속성 및 메소드를 상속받게된다. 이를 통해 특정 객체가 특정 속성에 접근하려 할 때 해당 객체를 탐색 후 찾지 못한다면 객체의 프로토타입 속성을 탐색하며, null을 가진 객체에 도달할 때 까지 지속된다.

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(this.name + " makes a noise.");
};

function Dog(name, breed) {
  Animal.call(this, name); // 상위 생성자 호출
  this.breed = breed;
}

Dog.prototype = Object.create(Animal.prototype);

Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log(this.name + " barks.");
};

var dog1 = new Dog("Rex", "German Shepherd");

dog1.speak();
dog1.bark();  
```

해당 코드는 prototype Chain을 설명하기 위한 코드로, dog1객체는 speak() 메서드와 bark()메서드가 존재하지 않더라도, 오류가 발생하지 않고 정상적인 출력값을 출력한다.
<br>
**<u>proto속성을 통해 상위 prototype과 서로 연결되어 있는 형태를 prototype Chain이라 부른다.</u>** 

모든 객체에 해당 프로토타입에 액세스 할 수 있는 속성이 존재한다.
<br>
__proto__ 라는 속성을 통해 접근할 수 있다.

```javascript
username.__proto__                        // String.prototype
username.__proto__.__proto__              // Object.prototype
username.__proto__.__proto__.__proto__    // null
```

<div class="notice">
  <h4>이와 같이 prototype을 통해 상속된 객체를 탐색하다 최상위 객체인 Object.prototype에 영향을 주게되면, 모든 객체에 속성이 포함되게 된다.</h4>
</div>

# Prototype-Pollution 

Prototype-Pollution의 경우 javascript 함수가 key를 먼저 삭제하지 않고 사용자가 제어할 수 있는 속성을 포함하는 객체를 기존 객체에 재귀적으로 병합할 때 발생한다.<br>
공격자는 __proto__와 같은 속성을 통해 중첩된 Property와 함께 proto와 같은 키를 가진 Property를 삽입할 수 있다.
> Property란 객체와 연관된 값을 의미함, 키와 값으로 구성된다.
이러한 오염은 모든 prototype를 오염시킬 수 있지만 내장된 전역 Object.prototype에서 가장 일반적으로 발생된다.

```javascript
function Test(name) {
    this.name = name;
}
var var1 = new Test("Jason")
```

위와 같은 코드가 있다고 가정 해보자.

```
var1.__proto__.__proto__;
Test.__proto__.__proto__;
```

이 때 위와 같이 __proto__를 통해 Object.prototype에 접근할 수 있다.

# Prototype pollution 공격 표면

## Prototype Pollution이 발생하는 패턴
1. 사용자의 입력 값이 Property에 설정하는 경우 __proto__와 같은 propery변경

```javascript
function isObject(obj) {
  return obj !== null && typeof obj === 'object';
}
 
function setValue(obj, key, value) {
  const keylist = key.split('.');
  const e = keylist.shift();
  if (keylist.length > 0) {
    if (!isObject(obj[e])) obj[e] = {};
    setValue(obj[e], keylist.join('.'), value);
  } else {
    obj[key] = value;
    return obj;
  }
}
 
const obj1 = {};
setValue(obj1, "__proto__.crackk", 1);
const obj2 = {};
console.log(obj2.crackk); // 1
```

2. 객체 병합을 통한 pollution

```javascript
function merge(a, b) {
  for (let key in b) {
    if (isObject(a[key]) && isObject(b[key])) {
      merge(a[key], b[key]);
    } else {
      a[key] = b[key];
    }
  }
  return a;
}
 
const obj1 = {a: 1, b:2};
const obj2 = JSON.parse('{"__proto__":{"crackk":1}}');
merge(obj1, obj2);
const obj3 = {};
console.log(obj3.crackk); // 1
```

3. 객체 복사를 통한 pollution

```javascript
function clone(obj) {
  return merge({}, obj);
}
 
const obj1 = JSON.parse('{"__proto__":{"crackk":1}}');
const obj2 = clone(obj1);
const obj3 = {};
console.log(obj3.crackk); // 1
```

Prototype pollution을 일으키기 위해서는 사용자의 입력이 prototype object에 Property를 추가할 수 있어야 한다.
1. 쿼리 또는 문자열을 통한 URL
2. JSON형식의 입력값
3. Web Message

## 쿼리 또는 문자열을 통한 URL
URL Query에서 URL 파서가 해당 Query를 파싱하여 임의의 문자열로 해석할 수 있다. 이 때 키:값 쌍으로 분해할 경우 pollution이 발생하며, XSS 와 같은 공격이 이루어질 수 있다.

```
https://vulnerable-website.com/?__proto__[evilProperty]=payload
'''
https://vulnerable-website.com/?__proto__[innerHTML]=<img/src/onerror%3dalert(1)>
```

## JSON형식의 입력값
사용자 입력값을 통해 제어가능한 객체는 JSON.parse() 메서드를 사용하는 경우가 많다. 이 때 JSON.parse()는 __proto__와 같은 키를 포함하여 JSON객체의 모든 키를 임의의 문자열로 취급하게 된다. 이를 통해 JSON값 입력을 통해 Prototype pollution이 가능하다.

```
const objectFromJson = JSON.parse('{"__proto__": {"evilProperty": "payload"}}');
'''
https://vulnerable-website.com/?__proto__{"innerHTML": "<img/src/onerror%3dalert(1)>"}
```

이와 같이 JSON.parse()를 통해 생성된 객체가 기존 객체에 병합되면 이전 URL을 통한 Pollution과 같이 Prototype pollution이 발생할 수 있다.

# 취약 유/무 확인

```
vulnerable-website.com/?__proto__[foo]=bar
OR
vulnerable-website.com/?__proto__.foo=bar
'''
Object.prototype.foo
// "bar" indicates that you have successfully polluted the prototype
// undefined indicates that the attack was not successful
``` 

위와 같이 쿼리 문자열 혹은 JSON 입력을 통해 임의의 속성 삽입을 통해  브라우저 콘솔에서 임의의 propery로 Pollution 유/무를 확인할 수 있다.

# Referance
1. https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes
2. https://hackyboiz.github.io/2021/10/30/l0ch/2021-10-30/