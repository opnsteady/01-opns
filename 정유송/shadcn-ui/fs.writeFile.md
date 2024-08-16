https://nodejs.org/api/fs.html

## **`fs.writeFile(file, data[, options], callback)`**

https://nodejs.org/api/fs.html#fswritefilefile-data-options-callback

```tsx
function writeFile(path, data, options, callback) {
  callback ||= options;
  validateFunction(callback, 'cb');
  options = getOptions(options, {
    encoding: 'utf8',
    mode: 0o666,
    flag: 'w',
    flush: false,
  });
  const flag = options.flag || 'w';
  const flush = options.flush ?? false;

  validateBoolean(flush, 'options.flush');

  if (!isArrayBufferView(data)) {
    validateStringAfterArrayBufferView(data, 'data');
    data = Buffer.from(data, options.encoding || 'utf8');
  }

  if (isFd(path)) {
    const isUserFd = true;
    const signal = options.signal;
    writeAll(path, isUserFd, data, 0, data.byteLength, signal, flush, callback);
    return;
  }

  if (checkAborted(options.signal, callback))
    return;

  fs.open(path, flag, options.mode, (openErr, fd) => {
    if (openErr) {
      callback(openErr);
    } else {
      const isUserFd = false;
      const signal = options.signal;
      writeAll(fd, isUserFd, data, 0, data.byteLength, signal, flush, callback);
    }
  });
}
```

https://github.com/nodejs/node/blob/main/lib/fs.js#L811

### `||=`

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment

**Logical OR assignment** (논리 OR 할당)

단축 연산자: `x ||= y`  뜻: `x || (x = y)`

- `x ||= y`는 `x = x || y`를 축약해서 표현한 것
    - `x || y`의 결과가 `x`에 할당됨
    - `x`가 거짓이 아니라면 `y`가 할당되지 않음
        
        ```tsx
        const x = 1;
        x ||= 2; 
        // x = 1
        ```
        
        - 따라서 `const`로 변수를 선언했어도 오류가 발생하지 않음

**`callback ||= options`**

- options가 없는 경우 callback으로 사용

cf) `||` Logical OR `??` Nullish coalescing 널 병합 연산자

- falsy일 때 / null이나 undefined일 때

### flag, flush

- flag: 파일을 열 때 동작 방식 정의
    - `r` 읽기 전용 `w` 쓰기 모드 등
- flush: 파일에 데이터를 쓴 후 즉시 디스크에 반영할지 여부
    - `false`(기본값): 캐시에 저장되나 디스크에 즉시 반영되지 않음. 일정 시간 후 자동으로 디스크에 반영
    - `true`: 데이터를 디스크에 즉시 반영

### validate

https://github.com/nodejs/node/blob/main/lib/internal/validators.js

validate를 위한 함수들 존재

### writeAll

파일에 데이터를 쓰는 역할