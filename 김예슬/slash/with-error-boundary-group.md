# withErrorBoundaryGroup

## 컨셉

- 하위 중첩된 ErrorBoundary를 한 번에 관리할 수 있는 컴포넌트 (reset 함수를 통해 한 번에 초기화 가능)
- 한 번에 관리하기 위해서는 상위에서 모든 하위 ErrorBoundary가 공유하는 resetKey 상태 선언 필요
- 중첩된 ErrorBoundary 관리를 위해서는 상위에 선언된 resetKey를 전달하기 위해 props drilling 필요
  => 관련 코드의 응집도 ⬇️, 유지보수성 ⬆️

## 세부 구현

1. ErrorBoundaryGroup에서 resetKey 선언
2. ErrorBoundary 마운트시 context 상의 resetKey 주입
   - ErrorBoundary 내부에서 참조하는 resetkeys가 변경되면 에러 초기화
3. 외부로 노출되는 useErrorBoudary에서는 reset 함수만 노출

## 아직 모르겠는 점

- blockOutside props의 동작 원리

## 알게된 점

- JSX.IntrinsicElements
  - html 태그의 타입이 맵핑된 객체
  - 타입 확장을 통해 내장 html 태그 타입을 추가할 수도 있음 Ex. 웹 컴포넌트
- JSXElementInstructor
  ```ts
  type JSXElementConstructor<P> =
    | ((props: P) => ReactElement | null)
    | (new (props: P) => Component<P, any>)
  ```
  - P를 props로 하는 순수 js 혹은 리액트 컴포넌트 타입

```tsx
<Component extends keyof JSX.IntrinsicElements | JSXElementConstructor<any>>
```

- html 호스트 엘리먼트 혹은 컴포넌트 타입
