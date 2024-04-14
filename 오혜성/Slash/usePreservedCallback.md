# usePreservedCallback

## 해결하는 문제

* callback 함수의 레퍼런스 보존
  + 최신화된 callback을 사용하면서, 레퍼런스는 그대로 보존함

* 이해를 돕기 위한 코드 조각 https://stackblitz.com/edit/vitejs-vite-czy7u1?file=src%2FApp.tsx

## 해결하는 방법

```tsx
return useCallback(
  (...args: any[]) => {
    return callbackRef.current(...args);
  },
  [callbackRef]
) as Callback;
```

* 실행될 때 callbackRef.current의 최신 callback을 실행
* useCallback의 의존성 배열에 ref를 넣었지만, ref 객체라 실제로 의존성 배열이 갱신되지 않음

```
useCallback 훅을 사용하여 새로운 콜백 함수를 반환합니다. 이 콜백 함수는 실행될 때 callbackRef.current를 호출하며, 이는 항상 최신의 callback을 가리킵니다. useCallback의 의존성 배열에 callbackRef를 넣어주었지만, callbackRef는 ref 객체이므로 실제로는 의존성 배열이 바뀌지 않습니다. 여기서 중요한 점은, 이 코드가 callbackRef의 현재 값을 사용하여 콜백을 실행하므로, 외부에서 callback이 변경되어도 이 반환된 콜백 함수의 레퍼런스는 변경되지 않습니다. 이는 성능 최적화에 도움을 줄 수 있으며, 불필요한 리렌더링을 방지할 수 있습니다.
```
