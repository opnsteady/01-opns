# usePreservedCallback

- useCallback의 단점을 보안하고 성능을 최적화시킨, 리렌더링 시에도 함수의 참조 안정성을 보장하는 hook

## useCallback의 단점

- 의존성 값이 여러 개이거나 자주 변하는 값인 경우 (eg. state, props) 결국 다시 계산되어 참조가 변경됨 -> 메모이제이션의 효과가 크지 못함
- 의존하고 있는 값을 의존성 배열에 넣지 않을 경우 메모된 함수는 이전 값을 기억하게 되어 의도치 않게 동작 -> (staled closure 문제)

## usePreservedCallback는 이를 어떻게 해결하였나

- **callback 함수의 참조값을 ref 에 저장**하고 callback 함수가 변경되었을 때 ref 값 업데이트
- 항상 최신의 scope를 참조하면서도 함수의 참조 안정성은 유지

## 참고

- - [usePreservedCallback vs useCallback](https://velog.io/@wns450/React%EC%9D%98-useCallback-%EC%B5%9C%EC%A0%81%ED%99%94%ED%95%9C-%EC%BB%A4%EC%8A%A4%ED%85%80-%ED%9B%85-usePreservedCallback)
