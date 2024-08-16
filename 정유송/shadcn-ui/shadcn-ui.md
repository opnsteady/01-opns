# 구현

radix ui를 기반으로 함 

radix ui에 구현되어 있지 않은 컴포넌트의 경우 (필요 시 합성 컴포넌트 형태로) 직접 구현하거나 다른 라이브러리 활용

직접 구현: button, card, input, pagination, [skeleton](https://tailwindcss.com/docs/animation#pulse), table, textarea

다른 라이브러리 활용: calendar, carousel, drawer, resizable, sooner

[react-day-picker](https://daypicker.dev/), [embla-carousel-react](https://www.embla-carousel.com/get-started/react/), [vaul](https://vaul.emilkowal.ski/), [react-resizable-panels](https://react-resizable-panels.vercel.app/), [sooner](https://sonner.emilkowal.ski/)

## 사용된 라이브러리

### [class-variance-authority](https://cva.style/docs)

variant 처리, 동적 스타일링에 도움

```tsx
import { cva, type VariantProps } from "class-variance-authority"

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
```

### [lucide-react](https://lucide.dev/guide/packages/lucide-react)

아이콘 관리: 컴포넌트화

https://lucide.dev/icons/

```tsx
import { ChevronLeft, ChevronRight } from "lucide-react"

function Calendar({
	...
	}) {
	return (
    <DayPicker
	    ...
	    components={{
        IconLeft: ({ ...props }) => <ChevronLeft className="h-4 w-4" />,
        IconRight: ({ ...props }) => <ChevronRight className="h-4 w-4" />,
      }}
      {...props}
    />
	)
}
```

### react-hook-form

form 관리에 사용
