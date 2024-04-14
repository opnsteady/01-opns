# useForceUpdate

https://slash.page/ko/libraries/react/react/src/hooks/useforceupdate.i18n/

반환된 함수를 실행 시 강제로 리렌더가 실행됩니다.

구현 코드

```tsx
import { useReducer } from 'react';

const updater = (num: number): number => (num + 1) % 1_000_000;

/** @tossdocs-ignore */
export function useForceUpdate(): () => void {
const [, forceUpdate] = useReducer(updater, 0);

return forceUpdate;
}
```

예시

```tsx
const forceUpdate = useForceUpdate();

const set = useCallback(
  (items: T[]) => {
    listRef.current = items;
    forceUpdate();
  },
  [forceUpdate]
);
```

`[useReducer](https://ko.react.dev/reference/react/useReducer)`의 `dispatch` 함수(`forceUpdate`)를 호출해 리렌더링을 일으킴

컴포넌트를 강제로 다시 렌더링 시켜 화면을 업데이트 할 수 있으나 잦은 리렌더링은 성능에 부담이 되므로 필요한 경우에만 사용하는 것이 좋음

## 새롭게 알게 된 점

### Numeric separators

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#numeric_literals

가독성을 위해 `_`로 숫자를 구분 지을 수 있다. (그룹화)

`1_000_000`은 `1000000`과 같다.