# es-toolkit

lodash와 비교해

- 작은 번들 사이즈
- 빠른 동작 속도

=== 높은 성능

→ 현대적인 JS로 구현했기 때문

## es-toolkit이 왜 (lodash보다) 빠른지 살펴보기

### chunk

**es-toolkit**

```tsx
export function chunk<T>(arr: readonly T[], size: number): T[][] {
  if (!Number.isInteger(size) || size <= 0) {
    throw new Error('Size must be an integer greater than zero.');
  }

  const chunkLength = Math.ceil(arr.length / size);
  const result: T[][] = Array(chunkLength);

  for (let index = 0; index < chunkLength; index++) {
    const start = index * size;
    const end = start + size;

    result[index] = arr.slice(start, end);
  }

  return result;
}
```

**lodash**

```tsx
function chunk(array, size = 1) {
    size = Math.max(toInteger(size), 0);
    const length = array == null ? 0 : array.length;
    if (!length || size < 1) {
        return [];
    }
    let index = 0;
    let resIndex = 0;
    const result = new Array(Math.ceil(length / size));

    while (index < length) {
        result[resIndex++] = slice(array, index, (index += size));
    }
    return result;
}

export default chunk;
```

- JS 메서드 사용 vs `toInteger`, `slice` 등 자체적으로 작성한 utils 함수 사용
- 유효성 검사로 맞지 않는 값에 대한 오류 처리를 선제적으로 진행

### difference

**es-toolkit**

```tsx
export function difference<T>(firstArr: readonly T[], secondArr: T[]): T[] {
  const secondSet = new Set(secondArr);

  return firstArr.filter(item => !secondSet.has(item));
}
```

**lodash**

```tsx
function difference(array, ...values) {
    return isArrayLikeObject(array)
        ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true))
        : [];
}

export default difference;
```

- `Set` 사용 vs `isArrayLikeObject`, `baseDifference`, `baseFlatten` 등 자체 utils 함수 사용

### drop

**es-toolkit**

```tsx
export function drop<T>(arr: readonly T[], itemsCount: number): T[] {
  return arr.slice(itemsCount);
}
```

**lodash**

```tsx
function drop(array, n = 1) {
    const length = array == null ? 0 : array.length;
    return length ? slice(array, n < 0 ? 0 : toInteger(n), length) : [];
}

export default drop;
```

- slice array 메서드 vs 자체 utils → 더 복잡한 자체 utils
    
    ```tsx
    arr.slice([begin[, end]])
    ```
    
    ```tsx
    // lodash
    function slice(array, start, end) {
        let length = array == null ? 0 : array.length;
        if (!length) {
            return [];
        }
        start = start == null ? 0 : start;
        end = end === undefined ? length : end;
    
        if (start < 0) {
            start = -start > length ? 0 : length + start;
        }
        end = end > length ? length : end;
        if (end < 0) {
            end += length;
        }
        length = start > end ? 0 : (end - start) >>> 0;
        start >>>= 0;
    
        let index = -1;
        const result = new Array(length);
        while (++index < length) {
            result[index] = array[index + start];
        }
        return result;
    }
    
    export default slice;
    ```
    

- 자체 utils을 사용하는 것은 호환성, 일관된 동작 보장, 확장성 측면에서 장점을 가질 것
- 그렇지만 번들사이즈, 성능상의 이점을 누리기 위해선 자체 메서드를 사용하는 편이 좋지 않을까 생각됨