# DebounceClick

https://slash.page/ko/libraries/react/react/src/components/DebounceClick/DebounceClick.i18n

click event에 debounce를 적용할 수 있는 유틸 컴포넌트입니다.

```tsx
import { Children, cloneElement, ReactElement } from 'react';
import { useDebounce } from '../../hooks/useDebounce';

/** @tossdocs-ignore */
interface Props {
  /**
   * @description 이벤트를 묶어서 한번에 보낼 시간으로 ms 단위
   * e.g.) 200ms 일 때, 200ms 안에 발생한 이벤트를 무시하고 마지막에 한번만 방출합니다.
   */
  wait: Parameters<typeof useDebounce>[1];
  options?: Parameters<typeof useDebounce>[2];
  children: ReactElement;
  /**
   * @default 'onClick'
   * @description 이벤트 Prop 이름으로 'onClick' 이름 외로 받을 때 사용합니다.
   * e.g. "onCTAClick", "onItemClick" ...
   */
  capture?: string;
}

export function DebounceClick({ capture = 'onClick', options, children, wait }: Props) {
  const child = Children.only(children);
  const debouncedCallback = useDebounce(
    (...args: any[]) => {
      if (child.props && typeof child.props[capture] === 'function') {
        return child.props[capture](...args);
      }
    },
    wait,
    options
  );

  return cloneElement(child, {
    [capture]: debouncedCallback,
  });
}
```

```tsx
<DebounceClick wait={200}>
  <Button
    onClick={() => {
      alert('onClick 이벤트 발생');
    }}
  >
    클릭
  </Button>
</DebounceClick>
```

- 자식 컴포넌트의 click 이벤트를 debounce하는 역할

## 새롭게 안 내용

### `Children.only(children)`

`Children.only(children)`은 `children`이 단일 React 엘리먼트인지 확인한다.

https://ko.react.dev/reference/react/Children#children-only

### `cloneElement`

`cloneElement(element, props, ...children)`

element를 기준으로 새로운 React 엘리먼트를 만든다. 엘리먼트를 복제해도 원본은 수정되지 않는다.

https://ko.react.dev/reference/react/cloneElement