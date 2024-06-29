# toss/es-toolkit

- 현대적인 자바스크립트 유틸리티 라이브러리
  - 높은 성능
  - 작은 번들 사이즈
  - 강력한 타입


## 새롭게 안 점

- `{ bench } from 'vitest'`
  - 성능 비교를 위해 bench 함수를 제공하더라구요
  - jest는 따로 제공하지 않는 것으로 보입니다

```tsx
import { bench, describe } from 'vitest';
import { chunk as chunkToolkit } from 'es-toolkit';
import { chunk as chunkLodash } from 'lodash';

describe('chunk', () => {
  bench('es-toolkit', () => {
    chunkToolkit([1, 2, 3, 4, 5, 6], 3);
  })

  bench('lodash', () => {
    chunkLodash([1, 2, 3, 4, 5, 6], 3);
  })
});
```