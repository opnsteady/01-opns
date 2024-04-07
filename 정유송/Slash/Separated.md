# Separated

https://slash.page/ko/libraries/react/react/src/components/Separated/Seperated.i18n

각 요소 사이에 반복되는 컴포넌트를 삽입하고자 할 때 사용할 수 있는 편리한 컴포넌트입니다.

```tsx
/** @tossdocs-ignore */
/** @jsxImportSource react */
import { Children, Fragment, isValidElement, PropsWithChildren, ReactNode } from 'react';

interface Props extends PropsWithChildren {
  with: ReactNode;
}

export function Separated({ children, with: separator }: Props) {
  const childrenArray = Children.toArray(children).filter(isValidElement);
  const childrenLength = childrenArray.length;

  return (
    <>
      {childrenArray.map((child, i) => (
        <Fragment key={i}>
          {child}
          {i + 1 !== childrenLength ? separator : null}
        </Fragment>
      ))}
    </>
  );
}
```

```tsx
<Separated with={<Border type="padding24" />}>
  {LIST.map(item => (
    <div>item.title</div>
  ))}
</Separated>
```

- 흔히 사용하는 패턴을 추상화 해 재사용성을 높임

## 새롭게 안 내용

### `Children.toArray(children)`

`children`에 속한 엘리먼트를 평면 배열(=1차원 배열)로 반환한다. 

각 child 들에 고유한 key를 할당한다. - 렌더링 최적화를 위한 목적

`filter`, `sort`, `reverse`와 같은 배열의 내장 메서드를 사용할 수 있게 된다. 

https://ko.react.dev/reference/react/Children#children-toarray

https://fe-developers.kakaoent.com/2021/211022-react-children-tip/

### `isValidElement`

`isValidElement(value)`

값이 React 엘리먼트인지 확인한다.

https://ko.react.dev/reference/react/isValidElement