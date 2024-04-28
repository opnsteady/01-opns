# react-transition-progress
* [데모](https://react-transition-progress.vercel.app/)

진행률 표시를 위한 도구

## 사용 방법

```tsx
import { ProgressBar, ProgressBarProvider } from "react-transition-progress";

<ProgressBarProvider>
  <ProgressBar />
</ProgressBarProvider>
```

* Provider 세팅 필요

<br />

```tsx
"use client";
import { useState, startTransition } from "react";
import { useProgress } from "react-transition-progress";

export default function MyComponent() {
  const startProgress = useProgress();
  const [count, setCount] = useState(0);
  return (
    <>
      <h1>Current count: {count}</h1>
      <button
        onClick={() => {
          startTransition(async () => {
            startProgress();
            // Introduces artificial slowdown
            await new Promise((resolve) => setTimeout(resolve, 2000));
            setCount((count) => count + 1);
          });
        }}
      >
        Trigger transition
      </button>
    </>
  );
}
```

- transition 시 startProgress 사용해서 진행률 표시

<br />

```tsx
import { Link } from "react-transition-progress/next";

<Link href="/about">Go to about page</Link>
```

- Next.js 헬퍼 Link 컴포넌트 제공

## 구현

- 내부적으로 framer-motion 사용 중
  - LazyMotion을 내부적으로 사용
  - https://github.com/vercel/react-transition-progress/blob/29fb5713b6f5a27354f9c66109220fd5613e2ed0/src/index.tsx#L162-L172

<br />

- useProgress는 컨텍스트의 반환 값을 사용

```tsx
https://github.com/vercel/react-transition-progress/blob/29fb5713b6f5a27354f9c66109220fd5613e2ed0/src/index.tsx#L181-L188

export function useProgress(): StartProgress {
    const progress = useProgressBarContext();

    const startProgress: StartProgress = () => {
        progress.start();
    }
    return startProgress
}
```

- start 외에 다른 메서드들이 있으나, 해당 인터페이스만 노출시키는 모습

<br />

- 그렇다면 컨텍스트는?

```tsx
https://github.com/vercel/react-transition-progress/blob/29fb5713b6f5a27354f9c66109220fd5613e2ed0/src/index.tsx#L20-L25

const ProgressBarContext = createContext<ReturnType<typeof useProgressInternal> | null>(
    null
);
```

- `useProgressInternal` 훅의 반환 값을 타입으로 사용

<br />

```tsx
https://github.com/vercel/react-transition-progress/blob/29fb5713b6f5a27354f9c66109220fd5613e2ed0/src/index.tsx#L66-L107

export function useProgressInternal() {
    const [loading, setLoading] = useOptimistic(false)

    const spring = useSpring(0, {
        damping: 25,
        mass: 0.5,
        stiffness: 300,
        restDelta: 0.1,
    });

    useInterval(
        () => {
            // If we start progress but the bar is currently complete, reset it first.
            if (spring.get() === 100) {
                spring.jump(0);
            }

            const current = spring.get();
            spring.set(Math.min(current + getDiff(current), 99));
        },
        loading ? 750 : null
    );

    useEffect(() => {
        if (!loading) {
            spring.jump(0);
        }
    }, [spring, loading]);

    /**
     * Start the progress.
     */
    function start() {
        setLoading(true)
    }

    return { loading, spring, start };
}
```

- spring 값을 넓이로 사용하고, 각 인터벌마다 랜덤 값을 기준으로 너비를 넓힘

- useOptimistic을 사용함
  - https://ko.react.dev/reference/react/useOptimistic
  - 초기에는 인자로 받은 값인 `false`를 사용해 loading은 false
  - 이후 `setLoading(true)`문을 통해서 true가 되고, useOptimistic의 updateFn을 사용하지 않았기 때문에 작업 이후 초기 값인 false가 되는 것으로 보임
    - `optimisticState: 결과적인 낙관적인 상태입니다. 작업이 대기 중이지 않을 때는 state와 동일하며, 그렇지 않은 경우 updateFn에서 반환된 값과 동일합니다.`

<br />

- useInterval을 만들어서 사용하는데요

```tsx
https://github.com/vercel/react-transition-progress/blob/29fb5713b6f5a27354f9c66109220fd5613e2ed0/src/index.tsx#L110-L135

function useInterval(callback: () => void, delay: number | null) {
    const savedCallback = useRef(callback);

    useEffect(() => {
        savedCallback.current = callback;
    }, [callback]);

    useEffect(() => {
        function tick() {
            savedCallback.current();
        }

        if (delay !== null) {
            tick();

            let id = setInterval(tick, delay);
            return () => clearInterval(id);
        }
    }, [delay]);
}
```

- 조건문 내부에서 effect 클린업을 사용할 수도 있겠구나 알게됨

<br />

- Next.js 헬퍼 Link 컴포넌트는

```tsx
https://github.com/vercel/react-transition-progress/blob/29fb5713b6f5a27354f9c66109220fd5613e2ed0/src/next.tsx#L21-L38

export function Link({
    href,
    children,
    replace,
    ...rest
}: Parameters<typeof NextLink>[0]) {
    const router = useRouter();
    const startProgress = useProgress()

    return (
        <NextLink
            href={href}
            onClick={(e) => {
                e.preventDefault();
                startTransition(() => {
                    startProgress()
                    const url = href.toString()
                    if (replace) {
                        router.replace(url)
                    } else {
                        router.push(url)
                    }
                })
            }}
            {...rest}
        >
            {children}
        </NextLink>
    );
}
```

- 기본 클릭 동작을 막고, startTransition과 자체 훅을 이용해 제공

## 빌드 도구

- bunchee 사용중
  - zero config 번들러
  - [같은 버셀 동료](https://github.com/huozhi)가 만들었네요

## 리드미에 있는 타입 문제 (startTransition)

- `import {startTransition} from 'react`의 startTransition은 비동기 함수를 허용하지 않음
  - `import {useTransition} from 'react'`에서 반환하는 startTransition은 비동기 함수를 허용
  - 그래서 공식 문서를 살짝 수정해서 PR을 올려봤으요...
  - https://github.com/vercel/react-transition-progress/pull/3