# useInputState

- input에 two way binding이 필요할 때
  - two way binding이란 state 내려주기만 하는 것이 아니라, 이벤트 핸들러를 통해 새로운 값을 부모로 올려주기도 하는 것
  - 주로 controlled component에서 사용되는 패턴
- controlled component를 사용할 때 주로 사용되는 로직 추상화
- 특히 target.value

알게된 점

- 호스트 엘리먼트 타입 상관없는 onChangeEventHandler의 타입: `ChangeEventHandler<HTMLElement & { value: string }>`
