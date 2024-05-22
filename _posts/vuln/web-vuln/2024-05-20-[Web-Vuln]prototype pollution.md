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
// Animal 생성자 함수 정의
function Animal(name) {
  this.name = name;
}

// Animal 프로토타입에 메서드 추가
Animal.prototype.speak = function() {
  console.log(this.name + " makes a noise.");
};

// Dog 생성자 함수 정의
function Dog(name, breed) {
  Animal.call(this, name); // 상위 생성자 호출
  this.breed = breed;
}

// Dog 프로토타입을 Animal 프로토타입으로 설정
Dog.prototype = Object.create(Animal.prototype);

// Dog 프로토타입 생성자를 Dog로 설정
Dog.prototype.constructor = Dog;

// Dog 프로토타입에 메서드 추가
Dog.prototype.bark = function() {
  console.log(this.name + " barks.");
};

// 객체 생성
var dog1 = new Dog("Rex", "German Shepherd");

// 메서드 호출
dog1.speak(); // 출력: Rex makes a noise.
dog1.bark();  // 출력: Rex barks.

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
Test.__proto__.__protO__;
```

이 때 위와 같이 __proto__를 통해 Object.prototype에 접근할 수 있다.

# Prototype pollution 공격 표면
Prototype pollution을 일으키기 위해서는 사용자의 입력이 prototype object에 Property를 추가할 수 있어야 한다.
1. 쿼리 또는 문자열을 통한 URL
2. JSON형식의 입력값
3. Web Message










# Referance
1. https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes
