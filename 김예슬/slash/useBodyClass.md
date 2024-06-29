# useBodyClass

- document.body에 클래스 추가
- 언마운트 시 클래스 해제
- useIsomorphicLayoutEffect 사용 (ssr에서 useEffect, csr에서 useLayoutEffect 사용하는 훅) -> 깜빡임 방지를 위해 페인트 전에 클래스 추가

## useIsomorphicLayoutEffect
