# useDisclosure

- 컴포넌트를 열고 닫을 때 사용할 수 있는 제어컴포넌트를 위한 hook
- `isOpen` `onOpen` `onClose` `onToggle` 등을 return 함
- 코드
  ```jsx

  export interface UseDisclosureProps {
    isOpen?: boolean
    defaultIsOpen?: boolean
    onClose?(): void
    onOpen?(): void
    id?: string
  }

  export function useDisclosure(props: UseDisclosureProps = {}) {
    const {
      onClose: onCloseProp,
      onOpen: onOpenProp,
      isOpen: isOpenProp,
      id: idProp,
    } = props

    const onOpenPropCallbackRef = useCallbackRef(onOpenProp)
    const onClosePropCallbackRef = useCallbackRef(onCloseProp)
    const [isOpenState, setIsOpen] = React.useState(props.defaultIsOpen || false)
    const [isControlled, isOpen] = useControllableProp(isOpenProp, isOpenState)

    const id = useId(idProp, "disclosure")

    const onClose = React.useCallback(() => {
      if (!isControlled) {
        setIsOpen(false)
      }
      onClosePropCallbackRef?.()
    }, [isControlled, onClosePropCallbackRef])

    const onOpen = React.useCallback(() => {
      if (!isControlled) {
        setIsOpen(true)
      }
      onOpenPropCallbackRef?.()
    }, [isControlled, onOpenPropCallbackRef])

    const onToggle = React.useCallback(() => {
      const action = isOpen ? onClose : onOpen
      action()
    }, [isOpen, onOpen, onClose])

    return {
      isOpen: !!isOpen,
      onOpen,
      onClose,
      onToggle,
      isControlled,
      getButtonProps: (props: any = {}) => ({
        ...props,
        "aria-expanded": isOpen,
        "aria-controls": id,
        onClick: callAllHandlers(props.onClick, onToggle),
      }),
      getDisclosureProps: (props: any = {}) => ({
        ...props,
        hidden: !isOpen,
        id,
      }),
    }
  }

  export type UseDisclosureReturn = ReturnType<typeof useDisclosure>

  ```

## **인터페이스**

- `isOpen?: boolean`: 컴포넌트가 현재 열려 있는지 여부를 지정. (for 제어 컴포넌트)
- `defaultIsOpen?: boolean`: 컴포넌트가 초기에 열려 있는지 여부를 지정 (for 비제어 컴포넌트)
- `onClose?(): void`: 컴포넌트가 닫힐 때 호출
- `onOpen?(): void`: 컴포넌트가 열릴 때 호출
- `id?: string`: 컴포넌트 id 설정 (for 접근성)

## 구현

- **useCallbackRef**
  - React의 `useCallback` 훅과 유사하게 작동 : ( 렌더링 시 값을 유지하기 위한 훅 )
  - 함수를 `useRef`를 통해 저장하고 내부에서 구현된 `useSafeLayoutEffect` 훅으로 관리
- **useControllableProp**
  ```jsx
  export function useControllableProp<T>(prop: T | undefined, state: T) {
    const isControlled = prop !== undefined
    const value = isControlled && typeof prop !== "undefined" ? prop : state
    return [isControlled, value] as const
  }
  ```
  - `isOpenProp` 으로 true든 false든 유효한 값이 넘어오면, `isControlled=true` (제어컴포넌트로 사용하겠다.)
  - 제어컴포넌트라면 받아온 isOpenProp을, 아니라면 defaultIsOpen(또는 false)을 value로 사용
  - `isControlled` (제어컴포넌트 여부) 와 `value` (open 여부) return
- **useId**
  ```jsx
  const id = useId(idProp, "disclosure");
  ```
  - React의 `useId` 훅을 사용하여 재구성한 훅
  - 외부에서 제공된 ID와 `prefix` 를 받아서 이를 기반으로 최종적인 ID값 구성
- **onClose**
  - 비제어컴포넌트면 내부에서 `setIsOpen(false)`
  - `useCallbackRef` 로 재구성된 onClose 함수 실행
  - `useCallback` 으로 묶어둠
- **onOpen**
  - 비제어 컴포넌트면 내부에서 `setIsOpen(true)`
  - `seCallbackRef` 로 재구성된 onOpen 함수 실행
  - `useCallback` 으로 묶어둠
- **onToggle**
  - 현재 `isOpen` 상태에 따라 `onClose` 또는 `onOpen`
  - `useCallback` 으로 묶어둠

## 반환

- `isOpen`: 컴포넌트의 현재 상태
- `onOpen`: 컴포넌트를 열기 위한 함수
- `onClose`: 컴포넌트를 닫기 위한 함수
- `onToggle`: 컴포넌트의 현재 상태에 따라 열거나 닫기 위한 함수입니다.
- `isControlled`: 제어 컴포넌트 여부
- `getButtonProps`: 버튼 속성
- `getDisclosureProps`: 모달 속성
  ```jsx
  function Basic() {
    const { getDisclosureProps, getButtonProps } = useDisclosure();

    const buttonProps = getButtonProps();
    const disclosureProps = getDisclosureProps();
    return (
      <>
        <Button {...buttonProps}>Toggle Me</Button>
        <Text {...disclosureProps} mt={4}>
          This text is being visibly toggled hidden and shown by the button.
          <br />
          (Inspect these components to see the rendered attributes)
        </Text>
      </>
    );
  }
  ```
