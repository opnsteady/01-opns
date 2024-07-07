# overlay-kit
* React에서 오버레이를 선언적으로 관리하기 위한 라이브러리

```tsx
import { overlay } from 'overlay-kit';

<Button 
  onClick={() => {
    overlay.open(({ isOpen, close }) => {
      return <Dialog open={isOpen} onClose={close} />;
    })
  }}
>
  Open
</Button>
```

## 궁금했던 점

### 왜 Provider를 여러 개 사용하면 안되나?

* React의 useSyncExternalStorage api를 이용해, 자바스크립트 객체(배열)을 상태로 관리하고 있음
* 해당 자원이 중복될 수 있기 때문으로 파악

```ts
// packages/src/context/store.ts

let overlays: OverlayData = {
  current: null,
  overlayOrderList: [],
  overlayData: {},
};
let listeners: Array<() => void> = [];

function emitChangeListener() {
  for (const listener of listeners) {
    listener();
  }
}

export function dispatchOverlay(action: OverlayReducerAction) {
  overlays = overlayReducer(overlays, action);
  emitChangeListener();
}

/**
 * @description for useSyncExternalStorage
 */
export const registerOverlaysStore = {
  subscribe(listener: () => void) {
    listeners = [...listeners, listener];

    console.log('l: ', listeners);
    return () => {
      listeners = listeners.filter((l) => l !== listener);
    };
  },
  getSnapshot() {
    return overlays;
  },
};
```

* 프로바이더를 여러 개 사용하며, 상대 주소를 가지는 모달을 띄울 경우 다음과 같은 현상이 생길 수 있음

https://github.com/hyesungoh/comet-land/assets/26461307/76b594e6-23a3-4de2-ac5f-3fb1826c8709

* 이를 방지하려면?
  + 클로저를 이용하거나 프로바이더 내에서 독립적인 자원을 사용하도록 해야할 거 같은데 ...
  + 조금 만져보니까 구조를 크게 바꿔야할 거 같은 느낌이라 손을 안대는게 나을거 같다는 생각...으로 이어짐
