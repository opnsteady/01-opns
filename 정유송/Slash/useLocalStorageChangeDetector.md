# useLocalStorageChangeDetector

## 설명 및 코드

브라우저 탭의 컨텐츠가 visible 일 때 업데이트 되는 localStorage 값을 가져옵니다.

```tsx
// key: LocalStorage 값을 가져올 때 사용할 Key
const [value, { clearStorage }] = useLocalStorageChangeDetector(key);
```

구현 코드

```tsx
import { generateStorage } from '@toss/storage';
import { useMemo, useState } from 'react';
import { useVisibilityEvent } from './useVisibilityEvent';

const storage = generateStorage();

/** @tossdocs-ignore */
export function useLocalStorageChangeDetector(key: string) {
  const [value, setValue] = useState(storage.get(key));

  useVisibilityEvent(state => {
    if (state !== 'visible') {
      return;
    }

    const v = storage.get(key);
    setValue(v);
  });

  const utils = useMemo(() => {
    return {
      clearStorage: () => storage.remove(key),
    };
  }, [key]);

  return [value, utils] as const;
}
```

사용 예시

```tsx
const [deviceWasRegistered, { clearStorage }] = useLocalStorageChangeDetector('deviceWasRegistered');

useEffect(() => {
  if (deviceWasRegistered !== null) {
    tossAppBridge.showToast('기기가 정상적으로 등록되었습니다.');
    fetchDeviceList();
    clearStorage();
  }
}, [deviceWasRegistered]);
```

- localStorage에 값을 가져와 값이 변경되면 해당 키에 대한 값을 제거할 수 있다.

## 새롭게 알게 된 내용

### Document.visibilityState

- `visible`, `hidden` 값을 반환해 사용자에게 현재 페이지가 보이고 있는지 여부를 확인할 수 있다. 

- 사용자가 해당 웹 페이지에 접근하고 있는지 여부를 확인할 수 있기 때문에 불필요한 실행, api 요청 등을 방지할 수 있어 사용성을 높이는 효과를 가져온다.

참고 
https://developer.mozilla.org/ko/docs/Web/API/Document/visibilityState