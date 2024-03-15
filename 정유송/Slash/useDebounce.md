# useDebounce

## 설명 및 코드

`[lodash/debounce](https://lodash.com/docs/4.17.15#debounce)` 를 쉽게 사용할 수 있는 hook

```tsx
import debounce from 'lodash.debounce';
import { useEffect, useMemo } from 'react';
import { usePreservedCallback } from './usePreservedCallback';
import { usePreservedReference } from './usePreservedReference';

/** @tossdocs-ignore */
export function useDebounce<F extends (...args: any[]) => any>(
  callback: F,
  wait: number,
  options: Parameters<typeof debounce>[2] = {}
) {
  const preservedCallback = usePreservedCallback(callback);
  const preservedOptions = usePreservedReference(options);

  const debounced = useMemo(() => {
    return debounce(preservedCallback, wait, preservedOptions);
  }, [preservedCallback, preservedOptions, wait]);

  useEffect(() => {
    return () => {
      debounced.cancel();
    };
  }, [debounced]);

  return debounced;
}
```

### 내가 작성한 코드와 비교

```tsx
import { useEffect, useState } from 'react'

export const useDebounce = (value: string, delay?: number) => {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value)
    }, delay || 500)

    return () => {
      clearTimeout(timer)
    }
  }, [value, delay])

  return debouncedValue
}

```

- 내가 작성한 `useBounce`는 특정 입력 값을 debounce로 처하는 반면, Slash에서 작성한 훅은 함수를 인자로 받는다.
  - 좀 더 유연한 활용이 가능할 것으로 보임
- 함수 호출은 계산 비용이 많이 드는 연산, 재사용을 위해 `useMemo`를 사용
    - **참조 동일성**을 지키기 위한 측면도 있음
- `useEffect`내부에서 클린업 함수 `return () ⇒ {}`를 활용해 새로운 값이 들어올 때/함수 생성됐을 때 타이머를 취소하는 로직은 동일

## 더 공부한 내용

### `useMemo`

https://ko.react.dev/reference/react/useMemo

- 비용이 높은 연산의 결과를 저장해 두어 의존성 배열의 값이 변경되지 않았다면 해당 값을 반환하는 훅으로 메모이제이션의 한 방법.
- 새롭게 알게 된 내용은 `useMemo`가 참조 동일성을 지키기 위한 목적으로도 사용될 수 있다는 점이다.
- 공식 문서에선 `useMemo`에 대해 이와 같이 설명하고 있다.
    > `useMemo` 는 재렌더링 사이에 계산 결과를 캐싱할 수 있게 해주는 Rect Hook 입니다.
    > React는 `[Object.is](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/is)` 비교를 통해 각 의존성 들을 이전 값과 비교합니다.


-  렌더링 사이에 계산 결과가 변경되었는지 판단하기 위해선 참조 동일성을 사용한다. 매 렌더링마다 의존성 배열의 값을 비교하는데, 객체 값의 경우 두 변수가 같은 객체 즉, 동일한 참조를 바라보고 있다면 값이 동일하다고 판단하게 된다. (`Object.is`를 사용해 두 값이 같은 지를 판별하며 객체가 같은 값을 참조하는 지도 비교한다.) 따라서 의존성 배열의 값이 같다면 메모이제이션 한 결과 값 또한 같은 값으로 유지 된다. 따라서 참조 동일성을 지킬 수 있다.

([usePreservedCallback](https://github.com/toss/slash/blob/main/packages/react/react/src/hooks/usePreservedCallback.ts), [usePreservedReference](https://github.com/toss/slash/blob/main/packages/react/react/src/hooks/usePreservedReference.ts)도 같은 방식으로 참조 동일성을 유지하기 위한 훅으로 보인다.)

참고 https://goongoguma.github.io/2021/04/26/When-to-useMemo-and-useCallback/