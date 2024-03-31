# devalue

https://github.com/Rich-Harris/devalue

## 해결하는 문제

* `JSON.stringify`가 못하는 것들
  + 참조 순환
  + undefined, Infinity, NaN, -0
  + RegExp, Error, Map, Set, WeakMap, WeakSet
  + Date

* 성능
* 보안

## 사용하는 방법

### uneval

```js
import * as devalue from 'devalue';

let obj = {
    message: 'hello'
};
devalue.uneval(obj); // '{message:"hello"}'

obj.self = obj;
devalue.uneval(obj); // '(function(a){a.message="hello";a.self=a;return a}({}))'
```

* 이 함수는 JavaScript 값을 취하고 JavaScript 코드를 반환하여 동등한 값을 생성합니다. 
  + 이는 역으로 eval과 비슷합니다.

* 가능한 가장 간결한 출력을 원하고 + 직렬화된 값을 핸들링하는 코드를 포함하지 않으려는 경우 uneval을 사용하십시오.

### stringify and parse

```js
import * as devalue from 'devalue';

let obj = {
    message: 'hello'
};

let stringified = devalue.stringify(obj); // '[{"message":1},"hello"]'
devalue.parse(stringified); // { message: 'hello' }

obj.self = obj;

stringified = devalue.stringify(obj); // '[{"message":1,"self":0},"hello"]'
devalue.parse(stringified); // { message: 'hello', self: [Circular] }
```

* JSON의 그것과 유사함

## 구현체

* svelte의 개발자이기도 한 작성자는 타입스크립트가 아닌, 자바스크립트(JSDOC)를 선호하는 것으로 유명

> https://github.com/sveltejs/svelte/pull/8569

* 그래서인지 당연하게도? 전부 자바스크립트로 작성되어 있음

### Exception

```js
https: //github.com/Rich-Harris/devalue/blob/50af63e2b2c648f6e6ea29904a14faac25a581fc/src/utils.js#L14C1-L24C2

    export class DevalueError extends Error {
        /**
         * @param {string} message
         * @param {string[]} keys
         */
        constructor(message, keys) {
            super(message);
            this.name = 'DevalueError';
            this.path = keys.join('');
        }
    }
```

### uneval

* 함수는 지원하지 않는 것을 상정하고 있어 먼저 핸들링

* 원시 값이 아닌 경우에 추가적인 핸들링

* 재귀적으로 연산

### stringify

* 전체적으로 배열 안에 [타입, 값] 형태로 관리
  + 각 타입 별로 핸들링하고 있음
  
* `if (thing === 0 && 1 / thing < 0)` 이건 무엇일까?
  + `-0`을 핸들링하는 것
    - `1 / -0`은 `-Infinity`, `1 / 0`은 `Infinity`
      - 자바스크립트는 엄격한 비교로 +0과 -0을 구분할 수 없음

    - `1 / thing < 0`
      - `1 / 0`은 `Infinity`, `1 / -0`은 `-Infinity`
      - `Infinity < 0`은 `false`, `-Infinity < 0`은 `true`
      

<details>

<summary>전문</summary>

이 코드 `thing === 0 && 1 / thing < 0` 는 JavaScript에서 매우 특별한 특성을 활용하는 예시입니다. 이 코드의 의미는 `thing` 변수가 0이며, 그 0이 음수인지를 검사하는 것입니다. JavaScript에서 0은 실제로 두 가지 값으로 표현될 수 있습니다: `+0` 과 `-0` . 이 두 값은 대부분의 연산에서 동일하게 취급되지만, 분모에 있을 때는 다른 결과를 나타냅니다. 즉, `1 / +0` 은 `Infinity` 로 평가되고, `1 / -0` 은 `-Infinity` 로 평가됩니다.

JavaScript에서는 이러한 특성을 이용하여 `thing` 이 `-0` 인지 확인할 수 있습니다. 코드의 첫 번째 부분 `thing === 0` 은 `thing` 이 `0` 인지 확인합니다. 이 때, `===` 연산자는 엄격한 비교를 수행하여 값과 타입이 모두 같아야 true를 반환합니다. 하지만 JavaScript에서는 `+0` 과 `-0` 을 엄격한 비교( `===` )로 구분할 수 없습니다; 즉, `+0 === -0` 은 `true` 입니다. 따라서 이 조건만으로는 `thing` 이 `+0` 인지 `-0` 인지 구분할 수 없습니다.

두 번째 부분 `1 / thing < 0` 은 `thing` 이 `-0` 일 때만 참이 됩니다. 왜냐하면 `1 / +0` 은 `Infinity` (양의 무한대)이고, `1 / -0` 은 `-Infinity` (음의 무한대)이기 때문입니다. 따라서 이 조건은 `thing` 이 `-0` 일 때만 `true` 가 되어 전체 표현식이 참이 됩니다.

요약하자면, 이 코드는 `thing` 이 정확히 `-0` 일 때만 `true` 를 반환합니다. JavaScript의 `-0` 과 `+0` 을 구분하는 독특한 방법 중 하나로, 특히 수학적 계산이나 물리 엔진 계산 등에서 `-0` 의 존재가 중요할 수 있는 경우에 유용하게 사용됩니다. 이러한 동작은 JavaScript의 IEEE 754 부동 소수점 숫자 표현 방식과 관련이 있으며, 이는 JavaScript가 숫자를 내부적으로 어떻게 처리하는지에 대한 깊은 이해를 요구합니다.

</details>

* `thing === void 0`은 무엇일까?
  + `undefined`를 핸들링하는 것
  + `void 0`은 `undefined`를 반환함
    - https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/void
  + void 0를 사용하는 이유는 JavaScript에서 undefined가 실제로는 전역 변수이며 초기 ECMAScript 버전에서는 이 변수에 다른 값을 할당할 수 있었기 때문
    - `undefined = true`를 작성하면 이후 undefined가 undefined가 아니게 될 수 있었음
