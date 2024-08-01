# shadow-panda

shadcn/ui를 기반으로 Panda CSS, Radix로 구축된 컴포넌트들

> shadcn도 radix 기반

## 사용 방법 비교

### shadcn/ui - button

1. 의존성 설치 필요

```bash
pnpm add @radix-ui/react-slot
```

2. 문서에서 제공되는 코드를 기반으로 컴포넌트 생성

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

3. 알잘딱 임포트해서 사용

```tsx
import { Button } from "@/components/ui/button"

<Button>버튼</Button>
```

* 위 의존성 설치와 컴포넌트 생성을 해주는 스크립트가 존재

```bash
pnpm dlx shadcn-ui@latest add button
```

### shadow-panda - button

1. 의존성 설치

```bash
pnpm add @radix-ui/react-slot
```

2. 문서에서 제공되는 코드를 기반으로 컴포넌트 생성

```tsx
import * as React from 'react'
import { Slot } from '@radix-ui/react-slot'
import { styled, type HTMLStyledProps } from '@shadow-panda/styled-system/jsx'
import { button } from '@shadow-panda/styled-system/recipes'

const BaseButton = React.forwardRef<
  HTMLButtonElement,
  React.ButtonHTMLAttributes<HTMLButtonElement> & { asChild?: boolean; children?: React.ReactNode }
>(({ asChild = false, ...props }, ref) => {
  const Comp = asChild ? Slot : 'button'
  return <Comp ref={ref} {...props} />
})
BaseButton.displayName = 'Button'

export const Button = styled(BaseButton, button)
export type ButtonProps = HTMLStyledProps<typeof Button>

```

   - shadcn과 다르게 스타일링이 shadown-panda에서 한 번 더 추상화되어 있어서 비교적 간단함

3. 알잘딱 임포트해서 사용

```tsx
import { Button } from '@/components/ui/button'

<Button>버튼</Button>
```

- 혹은 `recipe`만을 임포트해서 사용할 수 있음

```tsx
import { button, type ButtonVariantProps } from '@shadow-panda/styled-system/recipes'

<button className={button({ variant: 'outline', size: 'default' })}>Button</button>
```

## 구현

- pnpm workspace를 이용한 monorepo 구조
- tsup을 이용해 빌드
- changesets를 이용한 버전 관리 및 변경 감지
- panda css를 기반으로 스타일링한 구현체가 대부분
  - panda css의 preset 설정을 사용하도록
  - https://panda-css.com/docs/customization/presets

## panda-css emitPackage?

- shadow-panda 문서의 설정 방법에 `emitPackage`라는 panda 설정이 존재
- 근데 공식 문서에는 존재하지 않음
- 찾아보니 deprecated된 설정
  - https://github.com/chakra-ui/panda/pull/2696

- 해당 설정은 `node_modules`에 panda css의 빌드 타임 결과물을 넣어서, 사용하는 곳에서 의존성처럼 사용할 수 있도록 하는 것

- 삭제하게 된 이유
  - node_modules를 캐시하게 되면 업데이트된 스타일이 적용되지 않음
  - 자동 가져오기가 권장되지 않음
  - 일부 IDE에서 제대로 반영되지 않는 경우 존재

- 대안
  - 상대 경로 쓰기
  - tsconfig path alias 설정이나 package.json의 subpath imports 설정
    - https://nodejs.org/api/packages.html#subpath-imports
    - 가능한 경우 #imports를 사용하는 것을 권장함
    - 타입스크립트의 경우 5.4 버전부터 subpath imports를 공식 지원