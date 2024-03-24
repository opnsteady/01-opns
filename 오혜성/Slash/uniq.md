# uniq

## 해결하는 문제

- 동일한 값을 제거한 배열을 반환함
- 동일 여부는 SameValueZero(동일 값 제로 동등) 연산을 사용

## 해결하는 방법

- Set을 사용하고 스프레드 연산자를 사용하여 배열로 변환함

## 알게된 점

- 동일 여부는 SameValueZero(동일 값 제로 동등) 연산을 사용
  - https://developer.mozilla.org/ko/docs/Web/JavaScript/Equality_comparisons_and_sameness#%EB%8F%99%EC%9D%BC_%EA%B0%92_%EC%A0%9C%EB%A1%9C_%EB%8F%99%EB%93%B1
  - `+0`과 `-0`을 같은 것으로 간주
  - `NaN`은 `NaN`과 같은 것으로 간주
  - Array.includes, Map, Set 메서드에서 사용됨