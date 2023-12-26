# async와 await

function 키워드 앞에 async 만 붙여주면 되고, 비동기로 처리되는 부분 앞에 await 만 붙여주면 된다.
 await 키워드를 사용하기 위해선 반드시 async function 정의가 되어 있어야 한다.
```jsx
async function 함수명() {
  await 비동기_처리_메서드_명();
}
```

`async`와 `await`라는 특별한 문법을 사용하면 프라미스를 좀 더 편하게 사용할 수 있습니다. `async`, `await`는 놀라울 정도로 이해하기 쉽고 사용법도 어렵지 않습니다.

## 프라미스 promise
콜백의 단점, 즉 프로그램의 진행을 다른 파트에 넘겨주지 않고도 개발자가 언제 작업이 끝날지 알 수 있고 그 다음에 무슨 일을 해야 할 지 스스로 결정할 수 있게 하는 것이 바로 '프라미스'입니다.

스크립트를 읽고, 결과에 따라 그다음(.then)에 무엇을 할지에 대한 코드를 작성하면 됩니다.

프로미스 객체는 규모가 커질수록 코드가 복잡해지고 중첩 사용으로 가독성이 떨어진다. 
**지나친 .then 의 남용**을 async 과 await 이 해결해준다.

```jsx
let promise = new Promise(function(resolve, reject) {
  // executor (제작 코드, '가수')
});
```

### promise VS async await 비교
```jsx
/* Promise Hell */
fetch('https://example.com/api')
  .then(response => response.json())
  .then(data => fetch(`https://example.com/api/${data.id}`))
  .then(response => response.json())
  .then(data => fetch(`https://example.com/api/${data.id}/details`))
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));
```
 함수의 리턴값을 변수가 받는 정의문 형식대로 되어 있어,
코드가 의도하고자 하는 바를 **동일 코드 레벨 라인에서** 알수가 있어 훨씬 편하다.
```jsx
async function getData() {
    const response = await fetch('https://example.com/api');
    const data = await response.json();
    const response2 = await fetch(`https://example.com/api/${data.id}`);
    const data2 = await response2.json();
    const response3 = await fetch(`https://example.com/api/${data.id}/details`);
    const data3 = await response3.json();
    console.log(data3);
}

getData();
```

## async 함수

`async` 키워드부터 알아봅시다. `async`는 function 앞에 위치합니다.

```js
// sync는 function 앞에 위치
async function f() {
  return 1;
}
```

function 앞에 `async`를 붙이면 해당 함수는 항상 프라미스를 반환합니다. 프라미스가 아닌 값을 반환하더라도 이행 상태의 프라미스(resolved promise)로 값을 감싸 이행된 프라미스가 반환되도록 합니다.

아래 예시의 함수를 호출하면 `result`가 `1`인 이행 프라미스가 반환됩니다. 직접 확인해 봅시다.

```js run
async function f() {
  return 1;
}

f().then(alert); // 1
```

명시적으로 프라미스를 반환하는 것도 가능한데, 결과는 동일합니다.

```js run
async function f() {
  return Promise.resolve(1);
}

f().then(alert); // 1
```

`async`가 붙은 함수는 반드시 프라미스를 반환하고, 프라미스가 아닌 것은 프라미스로 감싸 반환합니다. 굉장히 간단하죠? 그런데 `async`가 제공하는 기능은 이뿐만이 아닙니다. 또 다른 키워드 `await`는 `async` 함수 안에서만 동작합니다. `await`는 아주 멋진 녀석이죠.

## await

`await` 문법은 다음과 같습니다.

```js
// await는 async 함수 안에서만 동작합니다.
let value = await promise;
```

자바스크립트는 `await` 키워드를 만나면 프라미스가 처리될 때까지 기다립니다(await는 '기다리다'라는 뜻을 가진 영단어입니다 - 옮긴이). 결과는 그 이후 반환됩니다.

1초 후 이행되는 프라미스를 예시로 사용하여 `await`가 어떻게 동작하는지 살펴봅시다.
```js run
async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("완료!"), 1000)
  });

*!*
  let result = await promise; // 프라미스가 이행될 때까지 기다림 (*)
*/!*

  alert(result); // "완료!"
}

f();
```

함수를 호출하고, 함수 본문이 실행되는 도중에 `(*)`로 표시한 줄에서 실행이 잠시 '중단'되었다가 프라미스가 처리되면 실행이 재개됩니다. 이때 프라미스 객체의 `result` 값이 변수 result에 할당됩니다. 따라서 위 예시를 실행하면 1초 뒤에 '완료!'가 출력됩니다.

1. `await`는 말 그대로 프라미스가 처리될 때까지 함수 실행을 기다리게 만듭니다. 프라미스가 처리되면 그 결과와 함께 실행이 재개되죠. 
2. 프라미스가 처리되길 기다리는 동안엔 엔진이 다른 일(다른 스크립트를 실행, 이벤트 처리 등)을 할 수 있기 때문에, CPU 리소스가 낭비되지 않습니다.

`await`는 `promise.then`보다 가독성 좋고 쓰기도 쉽습니다.

````warn header="일반 함수엔 `await`을 사용할 수 없습니다."
`async` 함수가 아닌데 `await`을 사용하면 문법 에러가 발생합니다.

```js run
// async 없이 await 사용 불가
function f() {
  let promise = Promise.resolve(1);
*!*
  let result = await promise; // Syntax error
*/!*
}
```

function 앞에 `async`를 붙이지 않으면 이런 에러가 발생할 수 있습니다. 앞서 설명해 드린 바와 같이 `await`는 `async` 함수 안에서만 동작합니다.
````

<info:promise-chaining> 챕터의 `showAvatar()` 예시를 `async/await`를 사용해 다시 작성해봅시다.

1. 먼저 `.then` 호출을 `await`로 바꿔야 합니다.
2. function 앞에 `async`를 붙여 `await`를 사용할 수 있도록 해야 합니다.

```js run
async function showAvatar() {

  // JSON 읽기
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();

  // github 사용자 정보 읽기
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // 아바타 보여주기
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // 3초 대기
  await new Promise((resolve, reject) => setTimeout(resolve, 3000));

  img.remove();

  return githubUser;
}

showAvatar();
```

코드가 깔끔해지고 읽기도 쉬워졌습니다. 프라미스를 사용한 것보다 훨씬 낫네요.

````smart header="`await`는 최상위 레벨 코드에서 작동하지 않습니다."
`await`을 이제 막 사용하기 시작한 분들은 최상위 레벨 코드(top-level code)에 `await`을 사용할 수 없다는 사실을 잊곤 합니다. 아래와 같은 코드는 동작하지 않습니다.

```js run
// 최상위 레벨 코드에선 문법 에러가 발생함
let response = await fetch('/article/promise-chaining/user.json');
let user = await response.json();
```

하지만 익명 async 함수로 코드를 감싸면 최상위 레벨 코드에도 `await`를 사용할 수 있습니다.

```js
// 최상위 레벨 코드에선 문법 에러가 발생하지만 (익명)함수로 감싸면 사용 가능
(async () => { 
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();
  ...
})();
```
````

## 에러 핸들링

프라미스가 정상적으로 이행되면 `await promise`는 프라미스 객체의 `result`에 저장된 값을 반환합니다. 반면 프라미스가 거부되면 마치 `throw`문을 작성한 것처럼 에러가 던져집니다.

예시:

```js
async function f() {
  // 프라미스가 거부되면 마치 에러가 던져집니다.
*!*
  await Promise.reject(new Error("에러 발생!"));
*/!*
}
```

위 코드는 아래 코드와 동일합니다.

```js
async function f() {
  // 마치 `throw`문을 작성한 것처럼 똑같이 작용
*!*
  throw new Error("에러 발생!");
*/!*
}
```

실제 상황에선 프라미스가 거부 되기 전에 약간의 시간이 지체되는 경우가 있습니다. 이런 경우엔 `await`가 에러를 던지기 전에 지연이 발생합니다.

`await`가 던진 에러는 `throw`가 던진 에러를 잡을 때처럼 `try..catch`를 사용해 잡을 수 있습니다.

```js run
async function f() {

  try {
    let response = await fetch('http://유효하지-않은-주소');
    // 에러 발생시 catch 로 넘어감
  } catch(err) {
    // 에러를 잡을 때 trt catch 사용
*!*
    alert(err); // TypeError: failed to fetch
*/!*
  }
}

f();
```

에러가 발생하면 제어 흐름이 `catch` 블록으로 넘어갑니다. 여러 줄의 코드를 `try`로 감싸는 것도 가능합니다.

```js run
async function f() {
  
// try 안에 여러 줄의 코드 넣기 가능
  try {
    let response = await fetch('http://유효하지-않은-주소');
    let user = await response.json();
  } catch(err) {
    // fetch와 response.json에서 발행한 에러 모두를 여기서 잡습니다.
    alert(err);
  }
}

f();
```

`try..catch`가 없으면 아래 예시의 async 함수 `f()`를 호출해 만든 프라미스가 거부 상태가 됩니다. `f()`에 `.catch`를 추가하면 거부된 프라미스를 처리할 수 있습니다.

```js run
// try..catch가 없을때
async function f() {
  let response = await fetch('http://유효하지-않은-주소');
}

// f()는 거부 상태의 프라미스가 됩니다.
*!*
f().catch(alert); // TypeError: failed to fetch // (*)
*/!*

// .catch를 추가하는 걸 잊으면 처리되지 않은 프라미스 에러가 발생
```

`.catch`를 추가하는 걸 잊으면 처리되지 않은 프라미스 에러가 발생합니다(콘솔에서 직접 확인해 봅시다). 이런 에러는 <info:promise-error-handling> 챕터에서 설명한 전역 이벤트 핸들러 `unhandledrejection`을 사용해 잡을 수 있습니다.


```smart header="`async/await`와 `promise.then/catch`"
`async/await`을 사용하면 `await`가 대기를 처리해주기 때문에 `.then`이 거의 필요하지 않습니다. 여기에 더하여 `.catch` 대신 일반 `try..catch`를 사용할 수 있다는 장점도 생깁니다. 항상 그러한 것은 아니지만, `promise.then`을 사용하는 것보다 `async/await`를 사용하는 것이 대개는 더 편리합니다.

그런데 문법 제약 때문에 `async`함수 바깥의 최상위 레벨 코드에선 `await`를 사용할 수 없습니다. 그렇기 때문에 관행처럼 `.then/catch`를 추가해 최종 결과나 처리되지 못한 에러를 다룹니다. 위 예시의 `(*)`로 표시한 줄처럼 말이죠.
```

````smart header="`async/await`는 `Promise.all`과도 함께 쓸 수 있습니다."
여러 개의 프라미스가 모두 처리되길 기다려야 하는 상황이라면 이 프라미스들을 `Promise.all`로 감싸고 여기에 `await`를 붙여 사용할 수 있습니다.

```js
// 프라미스 처리 결과가 담긴 배열을 기다립니다.
let results = await Promise.all([
  fetch(url1),
  fetch(url2),
  ...
]);
```

실패한 프라미스에서 발생한 에러는 보통 에러와 마찬가지로 `Promise.all`로 전파됩니다. 에러 때문에 생긴 예외는 `try..catch`로 감싸 잡을 수 있습니다.
````

## 요약

function 앞에 `async` 키워드를 추가하면 두 가지 효과가 있습니다.

1. 함수는 언제나 프라미스를 반환합니다.
2. 함수 안에서 `await`를 사용할 수 있습니다.

프라미스 앞에 `await` 키워드를 붙이면 자바스크립트는 프라미스가 처리될 때까지 대기합니다. 처리가 완료되면 조건에 따라 아래와 같은 동작이 이어집니다.

1. 에러 발생 --  예외가 생성됨(에러가 발생한 장소에서 `throw error`를 호출한 것과 동일함)
2. 에러 미발생 -- 프라미스 객체의 result 값을 반환

`async/await`를 함께 사용하면 읽고, 쓰기 쉬운 비동기 코드를 작성할 수 있습니다.

`async/await`를 사용하면 `promise.then/catch`가 거의 필요 없습니다. 하지만 가끔 가장 바깥 스코프에서 비동기 처리가 필요할 때같이 `promise.then/catch`를 써야만 하는 경우가 생기기 때문에 `async/await`가 프라미스를 기반으로 한다는 사실을 알고 계셔야 합니다. 여러 작업이 있고, 이 작업들이 모두 완료될 때까지 기다리려면 `Promise.all`을 활용할 수 있다는 점도 알고 계시기 바랍니다.
````

