# next-view-transitions

## 해결하는 문제

* View Transition API를 Next.js App Router에서 사용할 수 있도록 도와줌

* 데모: https://next-view-transitions.vercel.app/

```tsx
import { ViewTransitions } from 'next-view-transitions'

export default function Layout({ children }) {
  return (
    <ViewTransitions>
      <html lang='en'>
        <body>
          {children}
        </body>
      </html>
    </ViewTransitions>
  )
}
```

```tsx
import { Link } from 'next-view-transitions'

export default function Component() {
  return (
    <div>
      <Link href='/about'>Go to /about</Link>
    </div>
  )
}
```

## 해결하는 방법

### ViewTransitions

* Context API를 사용

* 브라우저 네이티브 트랜지션을 이용해 상태를 관리하기 위해 `useSyncExternalStore`를 사용
  + https://react-ko.dev/reference/react/useSyncExternalStore

* `use` 훅을 사용해 Context를 사용??
  + https://react.dev/reference/react/use
  + https://yceffort.kr/2023/06/react-use-hook

### use hook

* Promise 값이나 context 값을 사용할 수 있음
* 다른 훅들과 다르게 loop나 if문 안에서 사용할 수 있음

* context - 값 읽기
* 프로미스 - 서버 컴포넌트에서 클라이언트 컴포넌트로 데이터 스트리밍
  + 위 리액트 공식 문서의 예제가 잘되어 있음!
  

## 데모에서 궁금했던 점 - 브라우저 서포트 처리

* `'startViewTransition' in document` 이런 형태로 했나 싶었는데

```css
.support .yes {
    display: none;
    color: rgb(0, 120, 0);
    background-color: rgba(0, 120, 0, 0.1);
}

@supports (view-transition-name: figure-caption) {
    .support .yes {
        display: inline;
    }

    .support .no {
        display: none;
    }
}
```

* `@supports`를 사용해 브라우저 서포트를 확인함
  + https://developer.mozilla.org/en-US/docs/Web/CSS/@supports
