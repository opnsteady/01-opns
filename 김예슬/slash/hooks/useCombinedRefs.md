## Ref와 CallbackRef

- ref에는 ref 객체뿐 아니라 callbackRef라고 하는 함수를 전달할 수도 있다.
  ```
  type RefCallback<T> = { bivarianceHack(instance: T | null): void }["bivarianceHack"];
  type Ref<T> = RefCallback<T> | RefObject<T> | null;
  ```
- 결국 모든 ref prop은 그냥 함수다…!
  ```tsx
  ref={ref}
  ref={(node) => ref.current = node}
  // 둘은 같다
  ```
- 리액트는 ref가 실행되어야하는지 확인하기 위해 참조 안정성을 사용한다
- useCallback으로 감싼 함수(callbackRef)를 전달하면 첫 렌더링 시에만 실행된다. (같은 참조이기 때문에)
- **컴포넌트의 생명주기가 아니라, DOM 노드의 생명주기에 바인드** 되어있기 때문에 **첫 렌더링 이후 dom 노드와 직접적인 상호작용**을 위해 useEffect + setState 대신 callbackRef를 사용할 수 있다

## 더 알아보기

- [callbackRef의 사용사례 - ref 값의 변경은 리렌더링도 useEffect도 일으키지 않는다. 때문에 조건부 렌더링되는 dom node를 참조하는 ref의 값이 stale한 상태가 되는 경우가 있는데, 이를 callbackRef로 해결할 수 있다! ](https://medium.com/@teh_builder/ref-objects-inside-useeffect-hooks-eb7c15198780)

## 참고

- [[번역] callback ref 사용으로 useEffect 방지하기 - callback ref 정의 및 사용 사례 전반적 참고](https://velog.io/@cnsrn1874/번역-callback-refs-사용으로-useEffect-방지하기)
