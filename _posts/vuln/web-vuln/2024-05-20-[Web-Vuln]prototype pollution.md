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