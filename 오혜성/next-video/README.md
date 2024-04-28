# next-video

## 해결하는 문제

* `<video>` 태그를 확장해 제공
* 자동 최적화
* UI 커스터마이징
* 미리보기 이미지
* 호환성
* Analytics
* AI 자막?

```tsx
import Video from 'next-video';
import getStarted from '/videos/get-started.mp4';

export default function Page() {
  return <Video src={getStarted} />;
}
```

* 백그라운드 비디오 컴포넌트는 자바스크립트 번들이 50% 수준이라고 함

## 해결하는 방법

* @mux 라이브러리에 일부 의존하고 있음

* poster 이미지는 일반 `img` 태그를 사용하고 있음

* 스타일링은 className 기반

* isReactComponent 유틸

```tsx
export function isReactComponent<TProps>(
  component: unknown
): component is React.ComponentType<TProps> {
  return (
    isClassComponent(component) ||
    typeof component === 'function' ||
    isExoticComponent(component)
  );
}

function isClassComponent(component: any) {
  return (
    typeof component === 'function' &&
    (() => {
      const proto = Object.getPrototypeOf(component);
      return proto.prototype && proto.prototype.isReactComponent;
    })()
  );
}

function isExoticComponent(component: any) {
  return (
    typeof component === 'object' &&
    typeof component.$$typeof === 'symbol' &&
    ['react.memo', 'react.forward_ref'].includes(component.$$typeof.description)
  );
}
```

* fast-refresh를 위해 `let [] = useState()`를 사용하고 있음
  + https://nextjs.org/docs/architecture/fast-refresh#fast-refresh-and-hooks

```tsx
  let [asset, setAsset] = useState(typeof src === 'object' ? src : undefined);

  // Required to make Next.js fast refresh when the local JSON file changes.
  // https://nextjs.org/docs/architecture/fast-refresh#fast-refresh-and-hooks
  if (typeof src === 'object') {
    asset = src;
    src = undefined;
  }
```
