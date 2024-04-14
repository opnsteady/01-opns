# ErrorBoundary

**선언적으로** 에러를 처리하는 컴포넌트

- 에러 상황일 때 fallback ui 렌더한다
- Error Fallback 컴포넌트에서 에러를 초기화 할 수 있다
- 의존성 값이 변경되면 알아서 reset 된다
- ```ts
  componentDidUpdate(prevProps: Props) {
    ...
    if (isDifferentArray(prevProps.resetKeys, resetKeys)) {
        this.resetErrorBoundary();
    }
  }
  ```

- 관심 에러만 핸들링 (ignoreError)

  ```ts
  componentDidCatch(error: Error, info: ErrorInfo) {
    const { onError, ignoreError } = this.props;

    if (ignoreError?.(error)) {
        throw error;
    }

    onError?.(error, info);
  }
  ```

# useErrorBoundary

- 리액트의 ErrorBoundary는 이벤트 핸들러, 비동기적 코드, SSR, 자식에서가 아닌 에러 경계 자체에서 발생하는 에러 내의 에러를 포착하지 못함
- 리액트가 감지하지 못하는 에러를 ErrorBoudary로 전달

## 알게된 점 & 느낀 점

- 함수 실행할 때의 null값 검사: `this.props.onReset?.()`
- 에러를 핸들링 (에러 catch, throw, reset)하는 BaseErrorBoundary 컴포넌트와 실제 노출하는 ErrorBoundary 컴포넌트 분리
- ErrorBoundary 컴포넌트에서 useImperativeHandle을 통해 BaseErrorBoundary의 메소드 중 필요한 부분(reset)만 노출

  ```tsx
  export const ErrorBoundary = forwardRef<
    { reset(): void },
    ComponentPropsWithoutRef<typeof BaseErrorBoundary>
  >((props, resetRef) => {
    const ref = useRef<BaseErrorBoundary>(null)
    useImperativeHandle(resetRef, () => ({
      reset: () => ref.current?.resetErrorBoundary(),
    }))

    return <BaseErrorBoundary {...props} ref={ref} />
  })
  ```

  - useImperativeHandle: 상위 컴포넌트에서 사용할 ref로 노출되는 객체를 지정할 수 있음
  - useImperativeHandle(ref, createHandle): createHandle로 반환되는 객체가 ref로 노출됨

## 참고

- https://www.jbee.io/articles/react/React%EC%97%90%EC%84%9C%20%EC%84%A0%EC%96%B8%EC%A0%81%EC%9C%BC%EB%A1%9C%20%EB%B9%84%EB%8F%99%EA%B8%B0%20%EB%8B%A4%EB%A3%A8%EA%B8%B0

## 더 알아보기

- https://happysisyphe.tistory.com/66
- https://www.jbee.io/articles/react/%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8%EC%9D%98%20%EC%82%AC%EC%9A%A9%EC%9E%90%20%EC%A4%91%EC%8B%AC%20%EC%98%88%EC%99%B8%20%EC%B2%98%EB%A6%AC
- https://www.jbee.io/articles/react/%EC%84%A0%EC%96%B8%EC%A0%81%EC%9C%BC%EB%A1%9C%20%EC%97%90%EB%9F%AC%20%EC%83%81%ED%99%A9%20%EC%A0%9C%EC%96%B4%ED%95%98%EA%B8%B0
