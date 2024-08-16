## shadcn/ui/button

```tsx
import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive:
          "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline:
          "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button"
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    )
  }
)
Button.displayName = "Button"

export { Button, buttonVariants }
```

## `asChild` prop

https://www.radix-ui.com/primitives/docs/guides/composition

`acChild`가 false이면 기본 컴포넌트를 렌더링 

`asChild`가 true이면 자식 컴포넌트 복제 & 해당 컴포넌트에 필요한 prop과 동작을 전달

컴포넌트에 ref를 전달해야 함 → [`forwardRef`](https://ko.react.dev/reference/react/forwardRef)를 사용

## radix/ui/Slot

https://www.radix-ui.com/primitives/docs/utilities/slot

props를 자식에 병합하는 역할을 하는 컴포넌트

```tsx
import React from 'react';
import { Slot } from '@radix-ui/react-slot';

function Button({ asChild, ...props }) {
  const Comp = asChild ? Slot : 'button';
  return <Comp {...props} />;
}
```

```jsx
import { Button } from './your-button';

export default () => (

  <Button asChild>
    <a href="/contact">Contact</a>
  </Button>
);
```

https://github.com/radix-ui/primitives/blob/main/packages/react/slot/src/Slot.tsx

### Slot

```tsx
const Slot = React.forwardRef<HTMLElement, SlotProps>((props, forwardedRef) => {
  const { children, ...slotProps } = props;
  const childrenArray = React.Children.toArray(children);
  const slottable = childrenArray.find(isSlottable);

  if (slottable) {
    const newElement = slottable.props.children as React.ReactNode;

    const newChildren = childrenArray.map((child) => {
      if (child === slottable) {
        // because the new element will be the one rendered, we are only interested
        // in grabbing its children (`newElement.props.children`)
        if (React.Children.count(newElement) > 1) return React.Children.only(null);
        return React.isValidElement(newElement)
          ? (newElement.props.children as React.ReactNode)
          : null;
      } else {
        return child;
      }
    });

    return (
      <SlotClone {...slotProps} ref={forwardedRef}>
        {React.isValidElement(newElement)
          ? React.cloneElement(newElement, undefined, newChildren)
          : null}
      </SlotClone>
    );
  }

  return (
    <SlotClone {...slotProps} ref={forwardedRef}>
      {children}
    </SlotClone>
  );
});
```

- `slottable` 컴포넌트가 있는지 확인
    - `slottable` 컴포넌트가 있다면, 자식 요소를 `newElement`에 할당
    - `slottable` 컴포넌트가 없다면, 기존 `children` 렌더링

### SlotClone

```tsx
const SlotClone = React.forwardRef<any, SlotCloneProps>((props, forwardedRef) => {
  const { children, ...slotProps } = props;

  if (React.isValidElement(children)) {
    const childrenRef = getElementRef(children);
    return React.cloneElement(children, {
      ...mergeProps(slotProps, children.props),
      // @ts-ignore
      ref: forwardedRef ? composeRefs(forwardedRef, childrenRef) : childrenRef,
    });
  }

  return React.Children.count(children) > 1 ? React.Children.only(null) : null;
});
```

- `children`이 유효한 React 요소라면, `cloneElement`로 `children` 복제
    - `mergeProps` 함수로 `slotProps`와 `children.props`를 병합
    - `composeRefs` 함수로 `ref` 결합
    
    ⇒ 자식 요소를 복제하고 해당 요소에 속성을 병합해 렌더링
    
- `children`이 유효한 React 요소가 아니라면 `null` 반환 / `children`의 개수가 1 초과라면 오류
    - `React.Children.only(childeren)` → `null`은 유효한 값이 아니므로 에러 발생
        - `null`은 React 요소가 아님
    
    ⇒ children이 단일 요소일 때만 처리 되도록 설정
    

### Slottable

```tsx
const Slottable = ({ children }: { children: React.ReactNode }) => {
  return <>{children}</>;
};

function isSlottable(child: React.ReactNode): child is React.ReactElement {
  return React.isValidElement(child) && child.type === Slottable;
}
```

- `child`가 React 요소인지, type이 `Slottable`인지 확인한다.
    - `Slottable`은 `children`을 가진 요소 → `Slot`이 렌더링할 대상

## 이런 패턴을 사용하는 이유?

- 유연한 컴포넌트 사용
- 자식 요소에 props와 동작(기능) 병합 용이

⇒ 재사용성