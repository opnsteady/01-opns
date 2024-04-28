# usePrevious

## 해결하는 문제

* 변경 이전 값을 반환함

## 해결하는 방법

* useEffect의 실행 시점을 이용해 변경 이전 값을 반환함

```
리액트의 훅 중 useEffect는 컴포넌트의 렌더링 결과가 화면에 반영된 후에 비동기적으로 실행됩니다. 이 특성이 usePrevious 훅이 이전 값을 추적할 수 있게 하는 핵심입니다.

    초기 렌더링 시: 컴포넌트가 처음 렌더링될 때, useRef를 사용하여 생성된 ref 객체의 .current 속성은 defaultValue 또는 초기 value로 설정됩니다. 이 시점에서는 useEffect가 아직 실행되지 않았기 때문에, ref.current에 저장된 값은 초기값입니다.

    값이 변경되고 다시 렌더링 될 때: value에 변경이 발생하면, 컴포넌트는 다시 렌더링됩니다. 이때 usePrevious 훅이 반환하는 ref.current의 값은 아직 업데이트되기 전의 value입니다. 왜냐하면, useEffect 내의 코드는 렌더링이 화면에 반영된 후에야 실행되기 때문입니다. 따라서, 이 시점에서 ref.current를 반환하면 이전 렌더링 시점에서의 value가 됩니다.

    useEffect 실행: 렌더링이 화면에 반영된 직후, useEffect 내부의 코드가 실행됩니다. 이 코드는 ref.current를 최신 value로 업데이트합니다. 이 업데이트는 현재 렌더링 사이클이 끝난 후에 발생하기 때문에, usePrevious 훅을 통해 반환받은 값은 업데이트 이전의 값, 즉 "이전 값"이 됩니다.

    다음 렌더링 시점: 다음 렌더링 사이클에서 value가 다시 변경되면, 2단계와 3단계가 반복됩니다. 이때 ref.current는 이전 렌더링 시점의 value를 가리키고 있을 것이고, useEffect의 실행으로 최신 값으로 업데이트됩니다.

결론적으로, useEffect가 렌더링 사이클이 끝난 후에 실행되는 특성 덕분에, usePrevious 훅은 value의 이전 값을 ref.current를 통해 효과적으로 추적하고 반환할 수 있습니다. 이러한 방식으로 리액트의 렌더링 사이클과 훅의 실행 시점을 이용한 것입니다.
```

- 이해를 돕기 위한 코드 조각 https://stackblitz.com/edit/vitejs-vite-czy7u1?file=src%2FApp.tsx