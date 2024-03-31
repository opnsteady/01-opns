# 왜 그냥 빌트인 html 인 button을 사용하지 않고 chakra.div를 사용할까? 그 내부는 어떻게 생겼을까?

- `button.tsx` 코드

  ```jsx
  import { omitThemingProps, ThemingProps } from "@chakra-ui/styled-system"
  import { cx, dataAttr } from "@chakra-ui/utils"
  import { chakra, forwardRef, HTMLChakraProps, useStyleConfig } from "../system"
  import { useButtonGroup } from "./button-context"
  import { ButtonOptions } from "./button-types"

  export interface ButtonProps
    extends HTMLChakraProps<"button">,
      ButtonOptions,
      ThemingProps<"Button"> {}

  /**
   * Button component is used to trigger an action or event, such as submitting a form, opening a Dialog, canceling an action, or performing a delete operation.
   *
   * @see Docs https://chakra-ui.com/docs/components/button
   * @see WAI-ARIA https://www.w3.org/WAI/ARIA/apg/patterns/button/
   */
  export const Button = forwardRef<ButtonProps, "button">((props, ref) => {
    const group = useButtonGroup()
    const styles = useStyleConfig("Button", { ...group, ...props })

    const {
      isDisabled = group?.isDisabled,
      isActive,
      type,
      className,
      as,
      ...rest
    } = omitThemingProps(props)

    return (
      <chakra.button
        ref={ref}
        as={as}
        type="button"
        data-in-group={dataAttr(!!group)}
        data-active={dataAttr(isActive)}
        __css={styles}
        disabled={isDisabled}
        {...rest}
        className={cx("chakra-button", className)}
      />
    )
  })

  Button.displayName = "Button"

  ```

- `factory.ts` 코드

  ```tsx
  import { ElementType } from "react";
  import { ChakraStyledOptions, HTMLChakraComponents, styled } from "./system";
  import { ChakraComponent } from "./system.types";
  import { DOMElements } from "./system.utils";

  interface ChakraFactory {
    <T extends ElementType, P extends object = {}>(
      component: T,
      options?: ChakraStyledOptions
    ): ChakraComponent<T, P>;
  }

  function factory() {
    const cache = new Map<DOMElements, ChakraComponent<DOMElements>>();

    return new Proxy(styled, {
      /**
       * @example
       * const Div = chakra("div")
       * const WithChakra = chakra(AnotherComponent)
       */
      apply(target, thisArg, argArray: [DOMElements, ChakraStyledOptions]) {
        return styled(...argArray);
      },
      /**
       * @example
       * <chakra.div />
       */
      get(_, element: DOMElements) {
        if (!cache.has(element)) {
          cache.set(element, styled(element));
        }
        return cache.get(element);
      },
    }) as ChakraFactory & HTMLChakraComponents;
  }
  /**
   * The Chakra factory serves as an object of chakra enabled JSX elements,
   * and also a function that can be used to enable custom component receive chakra's style props.
   *
   * @see Docs https://chakra-ui.com/docs/styled-system/chakra-factory
   */
  export const chakra = factory();
  ```

### factory = chakra

- Chakra UI로 스타일링된 **컴포넌트를 생성**하는 프록시 함수
- `const cache = new Map<DOMElements, ChakraComponent<DOMElements>>()` 를 통해 캐시를 저장, 생성된 컴포넌트를 캐싱하여 매번 다시 생성하지 않고 재사용할 수 있도록 함

### Proxy

<aside>
💡 JavaScript의 `Proxy` 객체는 기본 동작을 가로채어 사용자 정의 동작을 추가할 수 있도록 함
- target: Proxy를 통해 감싸질 실제 객체
- handler: 프로퍼티들이 함수인 객체로 가로채는 메소드를 정의

</aside>

- 함수에 대한 작업을 가로채고, 그 작업들을 조작하거나 추가적인 로직을 수행하기 위함
- factory 함수 내에서 apply와 get을 가로채어, 동적으로 컴포넌트를 생성하고 캐싱하는 기능
- 장점
  - 동적 컴포넌트 생성
  - 캐싱을 통한 성능 최적화
  - 유연한 함수 호출 인터셉트

### apply

- styled 함수에 대한 호출
- 프록시 함수가 호출될 때 실행
- `styled(…argArray)` 를 호출하여 전달받은 인자들로 스타일링된 컴포넌트를 생성하고 반환함

### get

- 속성에 접근
- 프록시 객체의 속성에 접근할 때 실행
- 요청된 요소에 캐시가 없다면 `styled` 함수로 스타일링하여 캐시에 저장. 이후엔 캐시된 컴포넌트 반환

### Proxy 사용 예시

```jsx
let target = {
  message: "Hello, world!",
};

let handler = {
  get: function (obj, prop) {
    if (prop in obj) {
      return obj[prop];
    } else {
      return `Property ${prop} not found`;
    }
  },
};

let proxy = new Proxy(target, handler);

console.log(proxy.message); // "Hello, world!"
console.log(proxy.nonExistentProperty); // "Property nonExistentProperty not found"
```
