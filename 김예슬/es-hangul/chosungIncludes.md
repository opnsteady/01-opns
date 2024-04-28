# chosungIncludes

- 문자열에 주어진 초성의 포함여부를 반환하는 함수

## [공백 처리를 어디에서 할 것인가: 내부에서 처리 vs 외부에 위임](https://github.com/toss/es-hangul/issues/41)

문제

- eg. 프론트엔드 개발자, ㅍㄹㅌㅇㄷㄱㅂㅈ => true ? false

과정

- 해당 문제는 "함수의 관심사" 범위를 어디까지로 볼 것인가
- 이 함수의 관심사는 "한글" -> 공백이 필수적, 공백도 다루는 것까지가 이 함수의 관심사
- 공백을 다루는 것이 "엣지 케이스"인가? -> 한글에서 공백은 보편적인 케이스

결론

- 함수 내부에서 공백 또한 처리하도록 수정
- 인자로 받은 문자열에 공백을 제거하여 초성을 비교하는 로직 추가

## [테스트 케이스 리팩토링](https://github.com/toss/es-hangul/issues/79)

- 테스트 로직과 값의 분리
  - it.each 파라미터화 테스팅을 통해 테스트 로직과 값 분리
  - 어떤 코드가 이해하기 쉬운가 (가독성) vs 코드 간결성
- 테스트 케이스 그룹화: 성공 케이스와 실패 케이스 분리

### [파라미터화 테스팅: test.each](https://jestjs.io/docs/api#testeachtablename-fn-timeout)

> test.each(table)(name, fn)

- 테스트케이스에 사용되는 값(`table`)을 인자로 받아 반복되는 테스트 코드를 작성하도록 도움
- 테스트를 설명하는 name에는 table 요소의 값을 사용할 수도 있음
- table이 2차원 배열일 경우 요소가 name에 순서대로 전달 (printf formatting을 따름. 인자의 타입에 따라)

  ```js
  function sum(a, b) {
    return a + b
  }

  // name에도 인자의 순서대로 전달
  describe('sum function', () => {
    test.each([
      [1, 2, 3],
      [2, 3, 5],
      [3, 4, 7],
    ])('should add %i and %i to equal %i', (a, b, expected) => {
      expect(sum(a, b)).toBe(expected)
    })
  })
  ```

- table이 객체의 배열일 경우 속성명을 변수로 사용 가능 (key 패스도 사용 가능 eg. $variable.key.nested.value)

  ```js
  const cases = [{ first: 1, second: 2, expected: 3 }]
  test.each(cases)(
    'should add $first and $second to equal $expected',
    ({ first, second, expected }) => {
      expect(sum(first, second)).toBe(expected)
    }
  )
  ```
