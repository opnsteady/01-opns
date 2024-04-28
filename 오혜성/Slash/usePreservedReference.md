# usePreservedReference

## 해결하는 문제

* 값의 레퍼런스(참조)를 유지시킴

## 해결하는 방법

* 비교 함수를 통해 값이 변경되었을 때만 참조를 갱신
  + 기본 비교 함수는 `JSON.stringify`를 통해 비교

* 참조를 유지하기 위해 state 사용

## type `{}` vs type `object`

* type `{}`는 not nullish
  + `null`과 `undefined`를 제외한 모든 값
  
* type `object`는 객체만 허용

https://type-level-typescript.com/articles/difference-between-object-types-in-typescript
