# zip

- 아쉬운 점: 이터러블과는 호환되지 않음

## 알게된 점

- ts infer keyword

  > `type ElementOf<T> = T extends Array<infer U> ? U : never;`

  - infer 키워드는 조건부 타입에서 사용
  - 제약조건 extends가 아닌 조건부 타입 extends에서만 사용 가능
  - `U`가 추론 가능한 타입이면 참, 아니면 거짓
  - 위 코드에서 T는 배열, ElementOf 타입은 T 요소의 타입 (T 요소의 타입을 추론하여 사용)

- unknown vs any vs never

  - any는 모든 타입이 올 수 있고, 어떤 타입으로든 변경가능 (다른 타입의 값도 재할당이 가능)
  - unknown은 그에 비해 제한적. 아직 정해지지 않은 타입 -> **작업 수행 전 타입 검사 필요** -> 안전 보장
  - never는 발생하지 않는 값. 어떤 값도 반환하지 않거나 할당할 수 없는 타입

- [zipped type](https://stackoverflow.com/questions/62116454/how-to-type-define-a-zip-function-in-typescript)
  > `type Zipped<T extends unknown[][]> = Array<{ [Key in keyof T]: ElementOf<T[Key]> }>;`
  ```ts
  const zipped = zip([1, 2], ['a', 'b'])
  // zipped: Zipped<T>, T = [number[], string[]]
  // zipped 타입이 [a, b][] 라고 할 때
  // a 타입은 ElementOf<T[0]> = number
  // b 타입은 ElementOf<typeof t[1]> = string
  ```
  - zipped 타입은 `[number, string][]`

## 더 알아보기

- [iterable, generator type](https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html)
- [lodash에서도 이터러블과의 호환성 지원을 요청하는 이슈](https://github.com/lodash/lodash/issues/737)
