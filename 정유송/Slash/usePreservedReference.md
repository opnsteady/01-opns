# usePreservedReference

https://slash.page/ko/libraries/react/react/src/hooks/usePreservedReference.i18n

comparator로 비교했을 때 값이 변경되었을 때에만 레퍼런스를 변경하도록 합니다.

기본으로는 JSON.stringify를 했을 때 동일한 값이면 레퍼런스를 유지합니다.

https://developer.mozilla.org/ko/docs/Web/JavaScript/Equality_comparisons_and_sameness

```tsx
// 값의 동일성을 검증을 JSON.stringify로 하는 검증 함수
function areDeeplyEqual<T extends NotNullishValue>(x: T, y: T) {
  return JSON.stringify(x) === JSON.stringify(y);
}

function usePreservedReference<T extends NotNullishValue>(
  // 레퍼런스를 보존할 값
  value: T,
  // 값의 동일성을 검증하는 함수
  // default: areDeeplyEqual
  areValuesEqual: (a: T, b: T) => boolean = areDeeplyEqual
): T;
```

구현

```tsx
import { useEffect, useState } from 'react';

// eslint-disable-next-line @typescript-eslint/ban-types
type NotNullishValue = {};

/** @tossdocs-ignore */
export function usePreservedReference<T extends NotNullishValue>(
  value: T,
  areValuesEqual: (a: T, b: T) => boolean = areDeeplyEqual
) {
  const [reference, setReference] = useState<T>(value);

  useEffect(() => {
    if (!areValuesEqual(value, reference)) {
      setReference(value);
    }
  }, [areValuesEqual, reference, value]);

  return reference;
}

function areDeeplyEqual<T extends NotNullishValue>(x: T, y: T) {
  return JSON.stringify(x) === JSON.stringify(y);
}
```

예제

```tsx
const params = usePreservedReference(loggerParams, areParamsEqual);
```

## 새롭게 알게 된 점

### comparator

comparator로 비교하는 것은 함수에 들어온 두 값을 비교하는 방식을 말함
(vs. comparable)

참고: https://dding9code.tistory.com/68

- `areDeeplyEqual`에서 x, y값을 매개변수로 받아 비교하고 있음

### JSON.stringify(`배열`/`객체`)

`stringify`를 사용하면 배열과 객체도 그 자체로 문자열로 변환된다.

즉, 단순한 값 비교가 가능하다.