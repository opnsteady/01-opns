# useIsomorphicLayouEffect

https://slash.page/ko/libraries/react/react/src/hooks/useIsomorphicLayoutEffect.i18n

Client-side에서는 useLayoutEffect 방식을 쓰고, Server-side에서는 useEffect 방식을 쓰도록 하는 hook 입니다.

Server-side에서는 useLayoutEffect 함수를 사용하면 warning 오류가 발생하기 때문에 사용하는 함수입니다.

구현 코드
```tsx
import { isClient } from '@toss/utils';
import { useEffect, useLayoutEffect } from 'react';

/** @tossdocs-ignore */
export const useIsomorphicLayoutEffect = isClient() ? useLayoutEffect : useEffect;
```
사용 예시
```tsx
useIsomorphicLayoutEffect(() => {
  setSwipeRefreshEnabled(false);
}, []);
```

```tsx
/** @tossdocs-ignore */

export function isServer() {
  return typeof window === 'undefined' || 'Deno' in globalThis;
}

/** @tossdocs-ignore */
import { isServer } from './isServer';

export function isClient() {
  return !isServer();
}
```

## useEffect와 useLayoutEffect 
### hook-flow
useLayoutEffect와 useEffect의 실행 시점

https://github.com/donavon/hook-flow

### useEffect

https://ko.react.dev/reference/react/useEffect

layout, paint 후에 실행

### useLayoutEffect

https://ko.react.dev/reference/react/useLayoutEffect

`useLayoutEffect`는 브라우저가 화면을 다시 그리기 전에 실행되는 [`useEffect`](https://ko.react.dev/reference/react/useEffect)입니다.

layout, paint 전에 실행

**서버 렌더링에서 `useLyaoutEffect`를 사용하면 에러가 발생한다** 

https://gist.github.com/gaearon/e7d97cdf38a2907924ea12e4ebdf3c85

DOM이 존재하지 않아 layout 정보가 없어 useLayoutEffect의 동작을 보장할 수 없음

[https://velog.io/@khy226/SSR에서는-UseLayoutEffect-대신-useEffect를-사용하자](https://velog.io/@khy226/SSR%EC%97%90%EC%84%9C%EB%8A%94-UseLayoutEffect-%EB%8C%80%EC%8B%A0-useEffect%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90)

**참고**

다른 라이브러리의 아이소모픽 이펙트 구현체
https://github.com/imbhargav5/rooks/blob/main/packages/rooks/src/hooks/useIsomorphicEffect.ts

레이아웃 이펙트 보다 빠른 시점이 실행되는 훅
https://react.dev/reference/react/useInsertionEffect
