# Slash - storage

## 해결하는 문제점

* localStorage나 sessionStorage를 사용할 수 없는 환경에서 안전하게 사용할 수 있게 함

* TypedStorage
  + parse, stringify를 추상화함
  + type safe하게 사용할 수 있음

```ts
const s = new TypedStorage<'foo' | 'bar'>('key');
s.set('baz'); // type error
```

* NumberTypedStorage
  + TypedStorage를 확장함
  + increase, decrease API를 제공

## 해결하는 방법

* local혹은 session storage를 사용할 때, 사용할 수 있다면 해당 스토리지를 사용하지만 사용하지 못한다면 인메모리 스토리지를 사용함
  + 인메모리 스토리지는 Map를 이용해 구현되어 있음

* 인메모리 스토리지와 각 스토리지들이 같은 인터페이스를 구현하도록 인터페이스와 class implements를 사용함

* 각 스토리지를 사용 가능 여부를 확인할 때 직접 생성한 난수로 키, 값을 저장해 try catch로 분기함

* TypedStorage는 boolean(true, false)가 사용될 때 각 값으로 추론되지 않고, boolean으로 추론되게끔 하기 위해 컨디셔널 타입을 사용함
  + `T extends boolean ? boolean : T`

* NumberTypedStorage는 `extends TypedStorage<number>`에 더해 각 메서드를 구현해 제공되고 있음
