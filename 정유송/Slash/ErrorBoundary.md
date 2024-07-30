# ErrorBoundary

https://www.slash.page/ko/libraries/react/error-boundary/src/ErrorBoundary.i18n

선언적으로 에러를 관리하기 위해서 사용하는 컴포넌트입니다. `ErrorBoundary` 컴포넌트는 children의 render/useEffect에서 발생한 에러를 잡아 `renderFallback`으로 주어진 컴포넌트를 렌더링합니다.

## 리액트 공식 문서

https://ko.legacy.reactjs.org/docs/error-boundaries.html

https://ko.react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // state를 업데이트하여 다음 렌더링에 fallback UI가 표시되도록 합니다.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 에러 리포팅 서비스에 에러를 기록할 수도 있습니다.
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 사용자 지정 fallback UI를 렌더링할 수 있습니다.
      return this.props.fallback;
    }

    return this.props.children;
  }
}
```

```jsx
<ErrorBoundary fallback={<p>Something went wrong</p>}>
  <Profile />
</ErrorBoundary>
```

- class 컴포넌트로 구현되는 이유는 함수 컴포넌트에서는 아래와 같은 기능을 하는 메서드가 없기 때문
    - [`componentDidCatch(error, info)`](https://ko.react.dev/reference/react/Component#componentdidcatch)
        - 자식 컴포넌트가 에러를 발생시킬 때 호출
    - [`static getDerivedStateFromError`](https://ko.react.dev/reference/react/Component#static-getderivedstatefromerror)
        - 에러에 대한 응답으로 state를 업데이트하고 사용자에게 에러 메시지 표시
    - [`componentDidUpdate(prevProps, prevState, snapshot?)`](https://ko.react.dev/reference/react/Component#componentdidupdate)
        - 컴포넌트가 업데이트된 props 또는 state로 다시 렌더링된 직후 호출
- `getDerivedStateFromError`와 `componentDidCatch`를 활용한 가장 기본적인 구현
- 단순하고 직관적인 에러 처리

## react-error-boundary

https://github.com/bvaughn/react-error-boundary

```tsx
export class ErrorBoundary extends Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);

    this.resetErrorBoundary = this.resetErrorBoundary.bind(this);
    this.state = initialState;
  }

  static getDerivedStateFromError(error: Error) {
    return { didCatch: true, error };
  }

  resetErrorBoundary(...args: any[]) {
    const { error } = this.state;

    if (error !== null) {
      this.props.onReset?.({
        args,
        reason: "imperative-api",
      });

      this.setState(initialState);
    }
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    this.props.onError?.(error, info);
  }

  componentDidUpdate(
    prevProps: ErrorBoundaryProps,
    prevState: ErrorBoundaryState
  ) {
    const { didCatch } = this.state;
    const { resetKeys } = this.props;

    // There's an edge case where if the thing that triggered the error happens to *also* be in the resetKeys array,
    // we'd end up resetting the error boundary immediately.
    // This would likely trigger a second error to be thrown.
    // So we make sure that we don't check the resetKeys on the first call of cDU after the error is set.

    if (
      didCatch &&
      prevState.error !== null &&
      hasArrayChanged(prevProps.resetKeys, resetKeys)
    ) {
      this.props.onReset?.({
        next: resetKeys,
        prev: prevProps.resetKeys,
        reason: "keys",
      });

      this.setState(initialState);
    }
  }

  render() {
    const { children, fallbackRender, FallbackComponent, fallback } =
      this.props;
    const { didCatch, error } = this.state;

    let childToRender = children;

    if (didCatch) {
      const props: FallbackProps = {
        error,
        resetErrorBoundary: this.resetErrorBoundary,
      };

      if (typeof fallbackRender === "function") {
        childToRender = fallbackRender(props);
      } else if (FallbackComponent) {
        childToRender = createElement(FallbackComponent, props);
      } else if (fallback === null || isValidElement(fallback)) {
        childToRender = fallback;
      } else {
        if (isDevelopment) {
          console.error(
            "react-error-boundary requires either a fallback, fallbackRender, or FallbackComponent prop"
          );
        }

        throw error;
      }
    }

    return createElement(
      ErrorBoundaryContext.Provider,
      {
        value: {
          didCatch,
          error,
          resetErrorBoundary: this.resetErrorBoundary,
        },
      },
      childToRender
    );
  }
}
```

```tsx
<ErrorBoundary
  FallbackComponent={Fallback}
  onReset={(details) => {
    // Reset the state of your app so the error doesn't happen again
  }}
  onError={logError}
>
  <ExampleApplication />
</ErrorBoundary>;
```

- `componentDidUpdate`를 사용해 `resetErrorBoundary` 구현
    - `this.setState(initialState)` 초기 상태로 설정해 에러 정보를 초기화 함
        - 초기 상태: 에러 존재x, 자식 컴포넌트 렌더링 문제 x
- `fallback`, `fallbackRender`, `FallbackComponent`로 구분해 각각 JSX, 함수, 컴포넌트를 지정하도록 함
    - didCatch 상태에 따라 다른 처리
    - 정적인 UI와 동적인 UI 구분 → 상황에 맞는 선택 가능

## slash

```tsx
class BaseErrorBoundary extends Component<PropsWithRef<PropsWithChildren<Props>>, State> {
  state = initialState;

  static getDerivedStateFromError(error: Error) {
    return { error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    const { onError, ignoreError } = this.props;

    if (ignoreError?.(error)) {
      throw error;
    }

    onError?.(error, info);
  }

  resetErrorBoundary = () => {
    this.props.onReset?.();
    this.setState(initialState);
  };

  componentDidUpdate(prevProps: Props) {
    if (this.state.error == null) {
      return;
    }

    if (isDifferentArray(prevProps.resetKeys, this.props.resetKeys)) {
      this.resetErrorBoundary();
    }
  }

  render() {
    const { children, renderFallback } = this.props;
    const { error } = this.state;

    if (error != null) {
      return renderFallback({
        error,
        reset: this.resetErrorBoundary,
      });
    }

    return children;
  }
}

export const ErrorBoundary = forwardRef<{ reset(): void }, ComponentPropsWithoutRef<typeof BaseErrorBoundary>>(
  (props, resetRef) => {
    const group = useContext(ErrorBoundaryGroupContext) ?? { resetKey: 0 };
    const resetKeys = [group.resetKey, ...(props.resetKeys || [])];

    const ref = useRef<BaseErrorBoundary>(null);
    useImperativeHandle(resetRef, () => ({
      reset: () => ref.current?.resetErrorBoundary(),
    }));

    return <BaseErrorBoundary {...props} resetKeys={resetKeys} ref={ref} />;
  }
);
if (process.env.NODE_ENV !== 'production') {
  ErrorBoundary.displayName = 'ErrorBoundary';
}
```

```tsx
<ErrorBoundary
  renderFallback={error => <div>에러가 발생했어요. {error.message}</div>}
  onError={(error, { componentStack }) => {
    alert(error.message);
    console.log(componentStack);
  }}
  resetKeys={['key1', 'key2']}
  onReset={() => {}}
  ignoreError={error => error.message.includes('어쩌구')}
>
  <에러를_발생시킬_수_있는_컴포넌트 />
</ErrorBoundary>
```

- `BaseErrorBoundary`로 `ErrorBoundary`기능 구현 → `forwardRef`를 통해 클래스 인스턴스에 대한 참조를 부모 컴포넌트에 전달
    - 부모 컴포넌트가 자식 컴포넌트 메서드 호출 가능
- 에러를 무시하는 기능 `ignoreError` 존재
    - `throw error`
- 제공하는 기능은 react-error-boundary와 유사하나 구현이 좀 더 간단

## useErrorBoundary

`ErrorBoundary`는 이벤트 핸들러에서 발생한 에러 (+ 비동기 실행 후 발생한 오류)를 인지하지 못 함

`useErrorBoundary`를 사용해 `ErrorBoundary`가 인지하지 못 하는 에러를 전달

### react-error-boundary

```tsx
export function useErrorBoundary<TError = any>(): UseErrorBoundaryApi<TError> {
  const context = useContext(ErrorBoundaryContext);

  assertErrorBoundaryContext(context);

  const [state, setState] = useState<UseErrorBoundaryState<TError>>({
    error: null,
    hasError: false,
  });

  const memoized = useMemo(
    () => ({
      resetBoundary: () => {
        context.resetErrorBoundary();
        setState({ error: null, hasError: false });
      },
      showBoundary: (error: TError) =>
        setState({
          error,
          hasError: true,
        }),
    }),
    [context.resetErrorBoundary]
  );

  if (state.hasError) {
    throw state.error;
  }

  return memoized;
}
```

- `Context` 사용 : ErrorBoundary 전반에 걸쳐 상태, 로직 공유
    - `ErrorBoundaryContext`의 `resetBoundary`

### slash

```tsx
export const useErrorBoundary = <ErrorType extends Error>() => {
  const [error, setError] = useState<ErrorType | null>(null);

  if (error != null) {
    throw error;
  }

  return setError;
};
```

- 에러 던지는 역할에 집중!


#### 추가 참고 자료
 - [jbee.io | 선언적으로 로딩과 에러 상태 처리하기](https://jbee.io/react/error-declarative-handling-1/)
 - [Slash 21 발표 | Suspense와 에러 처리](https://toss.im/slash-21/sessions/3-1)
 - [https://github.com/hyesungoh/learningWhatIWant/blob/5af3c462bb896c69a4d636588326a2aac0edfe6c/Books/모던-리액트-Deep-Dive/2_리액트_핵심_요소_깊게_살펴보기/README.md?plain=1#L163-L175](https://github.com/hyesungoh/learningWhatIWant/blob/5af3c462bb896c69a4d636588326a2aac0edfe6c/Books/%EB%AA%A8%EB%8D%98-%EB%A6%AC%EC%95%A1%ED%8A%B8-Deep-Dive/2_%EB%A6%AC%EC%95%A1%ED%8A%B8_%ED%95%B5%EC%8B%AC_%EC%9A%94%EC%86%8C_%EA%B9%8A%EA%B2%8C_%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0/README.md?plain=1#L163-L175)
 - https://happysisyphe.tistory.com/66