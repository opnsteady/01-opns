# withAsyncBoundary

hoc 타입 선언

```tsx
export default function withAsyncBoundary<
  Props extends Record<string, unknown> = Record<string, never>
>(
  Component: ComponentType<Props>,
  asyncBoundaryProps: ComponentProps<typeof AsyncBoundary>
) {
  const Wrapped = (props: Props) => <Component {...props} />
  return Wrapped
}
```

- props를 Component에 그대로 전달할 때, props의 타입을 any가 아닌 `Record<string, unknown>` 타입으로 지정
- 제네릭 타입 선언 시 기본 타입 지정도 가능
- [{}, Object, Record<string, unknown> 의 차이점](https://www.reddit.com/r/typescript/comments/tq3m4f/the_difference_between_object_and_recordstring/)
  - 할당 가능한 값의 범위로 따지면 `{}` > Object > `Record<string, unknown>`
  - {}의 경우 string, number와 같은 원시타입도 할당 가능! (Null, undefined 제외 모두 할당 가능)
  - Object는 객체 타입 모두 가능 (함수, 배열, ...)
  - Record... 는 {}, {foo: 'bar'}와 같은 객체 타입만 할당 가능 (위의 코드에서 의도한 타입의 범위는 여기에 해당)
  - `Record<string, never>`는 빈 객체만 해당!
- ComponentProps<typeof Component> 로 Component의 props 타입 접근 가능

알게된 점

- Record<Key, Value>
- hoc를
