
# 클래스 상속

클래스 상속을 사용하면 클래스를 다른 클래스로 확장할 수 있습니다.

기존에 존재하던 기능을 토대로 새로운 기능을 만들 수 있죠.

## 'extends' 키워드 

먼저, 클래스 `Animal`을 만들어보겠습니다.

```js
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed = speed;
    alert(`${this.name} 은/는 속도 ${this.speed}로 달립니다.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name} 이/가 멈췄습니다.`);
  }
}

let animal = new Animal("동물");
```

객체 `animal`과 클래스 `Animal`의 관계를 그림으로 나타내면 다음과 같습니다.

![](rabbit-animal-independent-animal.svg)

또 다른 클래스 `Rabbit`을 만들어보겠습니다.

토끼는 동물이므로 `Rabbit`은 동물 관련 메서드가 담긴 `Animal`을 확장해서 만들어야 합니다. 이렇게 하면 토끼도 동물이 할 수 있는 '일반적인' 동작을 수행할 수 있습니다.

클래스 확장 문법 `class Child extends Parent`를 사용해 클래스를 확장해 봅시다.

`Animal`을 상속받는 `class Rabbit`를 만들어 보겠습니다.

```js
// extends 클래스 확장 사용
// 이제 Rabbit.prototype에서 메서드를 찾지 못하면 Animal.prototype에서 메서드를 가져옵니다.
*!*
class Rabbit extends Animal {
*/!*
  hide() {
    alert(`${this.name} 이/가 숨었습니다!`);
  }
}

let rabbit = new Rabbit("흰 토끼");

rabbit.run(5); // 흰 토끼 은/는 속도 5로 달립니다.
rabbit.hide(); // 흰 토끼 이/가 숨었습니다!
```

클래스 `Rabbit`을 사용해 만든 객체는 `rabbit.hide()` 같은 `Rabbit`에 정의된 메서드에도 접근할 수 있고, `rabbit.run()` 같은 `Animal`에 정의된 메서드에도 접근할 수 있습니다.

키워드 `extends`는 프로토타입을 기반으로 동작합니다. `extends`는 `Rabbit.prototype.[[Prototype]]`을 `Animal.prototype`으로 설정합니다. 그렇기 때문에 `Rabbit.prototype`에서 메서드를 찾지 못하면 `Animal.prototype`에서 메서드를 가져옵니다.

![](animal-rabbit-extends.svg)

엔진은 다음 절차를 따라 메서드 `rabbit.run`의 존재를 확인합니다(그림을 아래부터 보세요).
1. 객체 `rabbit`에 `run`이 있나 확인합니다. `run`이 없네요.
2. `rabbit`의 프로토타입인 `Rabbit.prototype`에 메서드가 있나 확인합니다. `hide`는 있는데 `run`은 없습니다.
3. `extends`를 통해 관계가 만들어진 `Rabbit.prototype`의 프로토타입, `Animal.prototype`에 메서드가 있나 확인합니다. 드디어 메서드 `run`을 찾았습니다.

<info:native-prototypes>에서 알아본 것처럼 자바스크립트의 내장 객체는 프로토타입을 기반으로 상속 관계를 맺습니다. `Date.prototype.[[Prototype]]`이 `Object.prototype`인 것처럼 말이죠. `Date` 객체에서 일반 객체 메서드를 사용할 수 있는 이유가 바로 여기에 있습니다. 


## 메서드 오버라이딩

이제 한발 더 나아가 메서드를 오버라이딩 해봅시다. 특별한 사항이 없으면 `class Rabbit`은 `class Animal`에 있는 **메서드를 '그대로' 상속**받습니다. 

### stop()
그런데 `Rabbit`에서 `stop()` 등의 메서드를 자체적으로 정의하면, 상속받은 메서드가 아닌 자체 메서드가 사용됩니다.

```js
class Rabbit extends Animal {
  stop() {
    // rabbit.stop()을 호출할 때 
    // Animal의 stop()이 아닌, 이 메서드가 사용됩니다.
  }
}
```

개발을 하다 보면 부모 메서드 전체를 교체하지 않고, 부모 메서드를 토대로 일부 기능만 변경하고 싶을 때가 생깁니다. 부모 메서드의 기능을 확장하고 싶을 때도 있죠. 이럴 때 커스텀 메서드를 만들어 작업하게 되는데, 이미 커스텀 메서드를 만들었더라도 이 과정 전·후에 부모 메서드를 호출하고 싶을 때가 있습니다. 

키워드 `super`는 이럴 때 사용합니다.

- `super.method(...)`는 부모 클래스에 정의된 메서드, `method`를 호출합니다.
- `super(...)`는 부모 생성자를 호출하는데, 자식 생성자 내부에서만 사용 할 수 있습니다.

### super.method()
자신이 상속받은 부모의 생성자를 호출하는 메소드.
부모 클래스로부터 상속받은 필드나 메소드를 자식 클래스에서 참조하는 데 사용하는 참조 변수입니다.

이런 특징을 이용해 토끼가 멈추면 자동으로 숨도록 하는 코드를 만들어보겠습니다.

```js run
class Animal {

  constructor(name) {
    this.speed = 0;
    this.name = name;
  }

  run(speed) {
    this.speed = speed;
    alert(`${this.name}가 속도 ${this.speed}로 달립니다.`);
  }

  stop() {
    this.speed = 0;
    alert(`${this.name}가 멈췄습니다.`);
  }

}

class Rabbit extends Animal {
  hide() {
    alert(`${this.name}가 숨었습니다!`);
  }

*!*
  stop() {
    super.stop(); // 부모 클래스의 stop을 호출해 멈추고,
    this.hide(); // 숨습니다.
  }
*/!*
}

let rabbit = new Rabbit("흰 토끼");

rabbit.run(5); // 흰 토끼가 속도 5로 달립니다.
rabbit.stop(); // 흰 토끼가 멈췄습니다. 흰 토끼가 숨었습니다!
```

`Rabbit`은 이제 실행 중간에 부모 클래스에 정의된 메서드 `super.stop()`을 호출하는 `stop`을 가지게 되었네요.


## 생성자 오버라이딩

생성자 오버라이딩은 좀 더 까다롭습니다.

지금까진 `Rabbit`에 자체 `constructor`가 없었습니다.

### constructor 
공식 페이지에서는 constructor란 클래스 내에서 객체를 생성하고 초기화하기 위한 특별한 메서드라고 소개하고 있습니다. 즉, 객체의 기본 상태(state)를 설정하는 공간인 것이죠. 

[명세서](https://tc39.github.io/ecma262/#sec-runtime-semantics-classdefinitionevaluation)에 따르면, 클래스가 다른 클래스를 상속받고 `constructor`가 없는 경우엔 아래처럼 **'비어있는' `constructor`가 만들어집니다.**

```js
class Rabbit extends Animal {
  // 자체 생성자가 없는 클래스를 상속받으면 자동으로 만들어짐
*!*
  constructor(...args) {
    super(...args);
  }
*/!*
}
```

보시다시피 생성자는 기본적으로 부모 `constructor`를 호출합니다. 이때 부모 `constructor`에도 인수를 모두 전달합니다. 클래스에 자체 생성자가 없는 경우엔 이런 일이 모두 자동으로 일어납니다.

이제 `Rabbit`에 커스텀 생성자를 추가해보겠습니다. 커스텀 생성자에서 `name`과  `earLength`를 지정해보겠습니다.

```js run
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  // ...
}

class Rabbit extends Animal {

*!*
  constructor(name, earLength) {
    this.speed = 0;
    this.name = name;
    this.earLength = earLength;
  }
*/!*

  // ...
}

*!*
// 동작하지 않습니다!
let rabbit = new Rabbit("흰 토끼", 10); // ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
*/!*
```

아이코! 에러가 발생하네요. 토끼를 만들 수 없습니다. 무엇이 잘못된 걸까요?

이유는 다음과 같습니다.

- **상속 클래스의 생성자에선 반드시 `super(...)`를 호출해야 하는데, `super(...)`를 호출하지 않아 에러가 발생했습니다. `super(...)`는 `this`를 사용하기 전에 반드시 호출해야 합니다.**

그런데 왜 `super(...)`를 호출해야 하는 걸까요?

당연히 이유가 있습니다. 상속 클래스의 생성자가 호출될 때 어떤 일이 일어나는지 알아보며 이유를 찾아봅시다.

자바스크립트는 '상속 클래스의 생성자 함수(derived constructor)'와 그렇지 않은 생성자 함수를 구분합니다. 상속 클래스의 생성자 함수엔 특수 내부 프로퍼티인 `[[ConstructorKind]]:"derived"`가 이름표처럼 붙습니다.

일반 클래스의 생성자 함수와 상속 클래스의 생성자 함수 간 차이는 `new`와 함께 드러납니다.

- 일반 클래스가 `new`와 함께 실행되면, 빈 객체가 만들어지고 `this`에 이 객체를 할당합니다.
- 반면, 상속 클래스의 생성자 함수가 실행되면, 일반 클래스에서 일어난 일이 일어나지 않습니다. 상속 클래스의 생성자 함수는 빈 객체를 만들고 `this`에 이 객체를 할당하는 일을 부모 클래스의 생성자가 처리해주길 기대합니다.

이런 차이 때문에 상속 클래스의 생성자에선 `super`를 호출해 부모 생성자를 실행해 주어야 합니다. 그렇지 않으면 `this`가 될 객체가 만들어지지 않아 에러가 발생합니다.

아래 예시와 같이 `this`를 사용하기 전에 `super()`를 호출하면 `Rabbit`의 생성자가 제대로 동작합니다.

```js run
class Animal {

  constructor(name) {
    this.speed = 0;
    this.name = name;
  }

  // ...
}

class Rabbit extends Animal {

  constructor(name, earLength) {
*!*
    super(name);
*/!*
    this.earLength = earLength;
  }

  // ...
}

*!*
// 이제 에러 없이 동작합니다.
let rabbit = new Rabbit("흰 토끼", 10);
alert(rabbit.name); // 흰 토끼
alert(rabbit.earLength); // 10
*/!*
```

## 요약

1. 클래스 확장하기: `class Child extends Parent`
    - `Child.prototype.__proto__`가 `Parent.prototype`이 되므로 메서드 전체가 상속됩니다.
2. 생성자 오버라이딩:
    - `this`를 사용하기 전에 `Child` 생성자 안에서 `super()`로 부모 생성자를 반드시 호출해야 합니다.
3. 메서드 오버라이딩:
    - `Child`에 정의된 메서드에서 `super.method()`를 사용해 `Parent`에 정의된 메서드를 사용할 수 있습니다.
