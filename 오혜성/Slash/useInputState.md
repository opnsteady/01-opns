# useInputState

## 해결하는 문제

- 인풋에 바인딩 가능한 값을 제공

```ts
const [value, onChange] = useInputState();
```

## 해결하는 방법

- onChange 함수는 value가 있는 HTMLElement 타입에 사용할 수 있도록 되어 있음
- 변환을 위한 인자도 받고 있음

