# 함수 바인딩

`setTimeout`에 메서드를 전달할 때처럼, 객체 메서드를 콜백으로 전달할 때 '`this` 정보가 사라지는' 문제가 생깁니다.

이번 챕터에선 이 문제를 어떻게 해결할지에 대해 알아보겠습니다.

## 사라진 'this'

객체 메서드가 **객체 내부가 아닌 다른 곳에 전달되어 호출되면** this가 사라집니다.

`setTimeout`을 사용한 아래 예시에서 `this`가 어떻게 사라지는지 살펴봅시다.

```js run
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

// setTimeout 으로 인해 분리되어 user 을 잃어버림. 그래서 user.sayHi 를 인식할수 없게 됨.
*!*
setTimeout(user.sayHi, 1000); // Hello, undefined!
*/!*
```

`this.firstName`이 "John"이 되어야 하는데, 얼럿창엔 `undefined`가 출력됩니다.

이렇게 된 이유는 `setTimeout`에 객체에서 분리된 함수인 `user.sayHi`가 전달되기 때문입니다. 위 예시의 마지막 줄은 다음 코드와 같습니다. 

```js
// 분리된 이 함수의 this 는 window 가 할당되고. window.firstName 이 없으므로 undefined 가 출력된다.
let f = user.sayHi;
setTimeout(f, 1000); // user 컨텍스트를 잃어버림
```

브라우저 환경에서 `setTimeout` 메서드는 조금 특별한 방식으로 동작합니다. 인수로 전달받은 함수를 호출할 때, `this`에 `window`를 할당합니다(Node.js 환경에선 `this`가 타이머 객체가 되는데, 여기선 중요하지 않으므로 넘어가겠습니다). 따라서 위 예시의 `this.firstName`은 `window.firstName`가 되는데, `window` 객체엔 `firstName`이 없으므로 `undefined`가 출력됩니다. 다른 유사한 사례에서도 대부분 `this`는 `undefined`가 됩니다.

객체 메서드를 실제 메서드가 호출되는 곳(예시에선 `setTimeout` 스케줄러)으로 전달하는 것은 아주 흔합니다. 
이렇게 메서드를 전달할 때, **컨텍스트도 제대로 유지하려면** 어떻게 해야 할까요?


## 방법 2: bind

`func.bind(context (this 에 고정시킬 것), arg1(인수에 넣고싶은 값), arg2(인수에 넣고싶은 값2), ...)`

bind 로 this 를 context 로 고정시켜주기.

모든 함수는 `this`를 수정하게 해주는 내장 메서드 [bind](mdn:js/Function/bind)를 제공합니다.

기본 문법은 다음과 같습니다.

```js
// 더 복잡한 문법은 뒤에 나옵니다.
let boundFunc = func.bind(context);
```

`func.bind(context)`는 함수처럼 호출 가능한 '특수 객체(exotic object)'를 반환합니다. 이 객체를 호출하면 **`this`가 `context`로 고정된 함수 `func`가 반환**됩니다.

따라서 `boundFunc`를 호출하면 `this`가 고정된 `func`를 호출하는 것과 동일한 효과를 봅니다. 

아래 `funcUser`에는 `this`가 `user`로 고정된 `func`이 할당됩니다.

```js run  
let user = {
  firstName: "John"
};

function func() {
  alert(this.firstName);
}

// bind 로 func 에 user 를 고정시켜주었다.
*!*
let funcUser = func.bind(user);
funcUser(); // John  
*/!*
```

여기서 `func.bind(user)`는 `func`의 `this`를 `user`로 '바인딩한 변형'이라고 생각하시면 됩니다.

인수는 원본 함수 `func`에 '그대로' 전달됩니다.

```js run  
let user = {
  firstName: "John"
};

function func(phrase) {
  alert(phrase + ', ' + this.firstName);
}

// this를 user로 바인딩합니다.
// func 함수의 this 를 bind 를 사용하여 this = user 로 고정시켜 주어 결과값은 fistName 인 John 이 출력된다.
let funcUser = func.bind(user);

*!*
funcUser("Hello"); // Hello, John (인수 "Hello"가 넘겨지고 this는 user로 고정됩니다.)
*/!*
```

이제 객체 메서드에 `bind`를 적용해 봅시다.


```js run
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

// 객체에 bind 를 시켜줘도 됩니다. 
// user 의 객체 sayHi 의 this 값을 user 로 바인딩 시켜주어. user.firstName 을 유지하게 된다. 
*!*
let sayHi = user.sayHi.bind(user); // (*)
*/!*

// 이제 객체 없이도 객체 메서드를 호출할 수 있습니다.
sayHi(); // Hello, John!

// 고정된 sayHi 내부의 user.firstName 을 호출하면 항상 John 이 정상 출력된다.  
setTimeout(sayHi, 1000); // Hello, John!

// bind 로 고정시켜주었기 때문에 시간이 변해도 setTimeout 으로 함수가 분리되어도 user 가 고정되어있다.
// 1초 이내에 user 값이 변화해도
// sayHi는 기존 값을 사용합니다.
user = {
  sayHi() { alert("또 다른 사용자!"); }
};
```

`(*)`로 표시한 줄에서 메서드 `user.sayHi`를 가져오고, 메서드에 `user`를 바인딩합니다. `sayHi`는 이제 '묶인(bound)' 함수가 되어 단독으로 호출할 수 있고 `setTimeout`에 전달하여 호출할 수도 있습니다. 어떤 방식이든 컨택스트는 원하는 대로 고정됩니다.

아래 예시를 실행하면 인수는 '그대로' 전달되고 `bind`에 의해 `this`만 고정된 것을 확인할 수 있습니다. 

```js run
let user = {
  firstName: "John",
  say(phrase) {
    // user 에 있는 say 의 this 를 user 로 고정시켰다. 
    // 이제 phrase 안에 들가는 인수는 ${phrase} 에 출력돠고 user.firstName 인 John 이 고정되어 계속 출력된다.
    alert(`${phrase}, ${this.firstName}!`);
  }
};

let say = user.say.bind(user);

// ${phrase} 안에 hello 가 들어가 출력된다.
say("Hello"); // Hello, John (인수 "Hello"가 say로 전달되었습니다.)

// ${phrase} 안에 bye 가 들어가 출력된다.
say("Bye"); // Bye, John ("Bye"가 say로 전달되었습니다.)
```

소화할수 있는 내용부터 섭취하자~

## 요약

`func.bind(context, ...args)`는 `this`가 `context`로 고정되고 인수도 고정된 함수 `func`을 반환합니다.

`bind`는 보통 객체 메서드의 `this`를 고정해 어딘가에 넘기고자 할 때 사용합니다. `setTimeout`에 넘길 때 같이 말이죠. 
