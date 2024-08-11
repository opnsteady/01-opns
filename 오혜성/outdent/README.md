# Outdent
* https://www.npmjs.com/package/outdent
* 읽기 쉬운 코드를 위해 들여 쓰기(indent)를 사용하는데, 이때 결과물에서는 내어쓰기(outdent)해주는 라이브러리

## CJS 지원

- 기본적으로 ESM 형태로 내보내기 되고 있지만
  - defaut export, named export 모두 지원함
- tsup 같은 도구를 이용하지 않고 린하게 CJS를 지원하고 있음

```ts
// Named exports.  Simple and preferred.
// import outdent from 'outdent';
export default defaultOutdent;
// import {outdent} from 'outdent';
export { defaultOutdent as outdent };

// In CommonJS environments, enable `var outdent = require('outdent');` by
// replacing the exports object.
// Make sure that our replacement includes the named exports from above.
declare var module: any;
if (typeof module !== "undefined") {
  // In webpack harmony-modules environments, module.exports is read-only,
  // so we fail gracefully.
  try {
    module.exports = defaultOutdent;
    Object.defineProperty(defaultOutdent, "__esModule", { value: true });
    (defaultOutdent as any).default = defaultOutdent;
    (defaultOutdent as any).outdent = defaultOutdent;
  } catch (e) {}
}

```
