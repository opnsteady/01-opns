# useStorageState

useStorageState 는 브라우저의 스토리지에 영속적으로 저장되는 상태를 React에서 편리하게 다룰 수 있도록 도와주는 Hook입니다.

```tsx
const [number, setNumber, refresh] = useStorageState('key', {defaultValue: 1});
```

## refresh

* storage 값과 동기화

## option.storage

* 기본 값은 safeLocalStorage
* safeSessionStorage 사용 가능

https://github.com/opnsteady/01-opns/blob/main/%EC%98%A4%ED%98%9C%EC%84%B1/Slash/storage.md

## 이슈

### 1. defaultValue가 초기에 스토리지에 저장되지 않음

* https://github.com/toss/slash/pull/494
* PR이 올라와 있긴 해요
* 직접 테스트 해보니 정상 동작함

### 2. `ToObject<T>` 타입

```ts
type ToObject<T> = T extends unknown[] | Record<string, unknown> ? T : never;
```
- 이런 타입으로 stringify 해도 문제 없는 타입인지(함수, 심볼) 확인하는데
- 해당 타입이 interface에 이상하게 동작함.. (라고 느낌)

```ts
type Foo = {
  id: number;
  title: string;
}

interface Bar {
  id: number;
  title: string;
  selectedOption: string;
}

type Baz = (a: number) => number;

type A = ToObject<Foo>; // Foo < 정상
type B = ToObject<Bar>; // never < 비정상으로 보임
type C = ToObject<Baz>; // never < 정상
```

- 그래서 질문을 올려봤는데
  - https://stackoverflow.com/questions/78775076/how-can-i-handle-object-type-with-satisfying-interface?noredirect=1#comment138887877_78775076

- 암묵적 인덱스 시그니처 때문이라는 답변을 받음
  - https://blog.herodevs.com/typescripts-unsung-hero-index-signatures-ddc3d1e34c9f

- 결국 '인터페이스는 선언 이후에 바뀔 수 있기 때문에 만족하지 않는다'로 이해됨