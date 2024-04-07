# 외부 구조 ( 인터페이스 )

- 컴포넌트 인터페이스 코드
  ```jsx
  <Button onClick={onOpen}>Open</Button>
  <Modal isOpen={isOpen} onClose={onClose} isCentered>
    <ModalOverlay />
    <ModalContent>
      <ModalCloseButton />
      <ModalHeader>Welcome Home</ModalHeader>
      <ModalBody>
        Sit nulla est ex deserunt exercitation anim occaecat. Nostrud
        ullamco deserunt aute id consequat veniam incididunt duis in sint
        irure nisi.
      </ModalBody>
      <ModalFooter>
        <Button onClick={onClose}>Cancel</Button>
        <Button>Save</Button>
      </ModalFooter>
    </ModalContent>
  </Modal>
  ```
- Button
- Modal
  - ModalOverlay
  - ModalContent
    - ModalCloseButton
    - ModalHeader
    - ModalBody
    - ModalFooter

## `useDisclosure`

https://chakra-ui.com/docs/hooks/use-disclosure

- 컴포넌트를 열고 닫을 때 사용할 수 있는 제어컴포넌트를 위한 hook
- `isOpen` `onOpen` `onClose` `onToggle` 등을 return 함
- **다음 주 할 것 : useDisclosure 내부 까보기**

# 내부 구조

```jsx
<ModalContextProvider value={context}>
  <ModalStylesProvider value={styles}>
    <AnimatePresence onExitComplete={onCloseComplete}>
      {context.isOpen && <Portal {...portalProps}>{children}</Portal>}
    </AnimatePresence>
  </ModalStylesProvider>
</ModalContextProvider>
```

- ModalContextProvider
- ModalStylesProvider
- AnimatePresence
- Portal

## `useModal`

- 모달 상태관리 : **`isOpen`**, **`onClose`** 와 같은 속성을 통해 모달의 열림 상태 관리, 사용자가 모달을 닫을 수 있는 방법구현(ex. 배경 클릭, Esc 키 누름)을 제어

  - 코드

    ```jsx
    const onKeyDown = useCallback(
      (event: React.KeyboardEvent) => {
        if (event.key === "Escape") {
          event.stopPropagation();

          if (closeOnEsc) {
            onClose?.();
          }

          onEsc?.();
        }
      },
      [closeOnEsc, onClose, onEsc]
    );

    const onOverlayClick = useCallback(
      (event: React.MouseEvent) => {
        event.stopPropagation();
        if (mouseDownTarget.current !== event.target) return;
        if (!modalManager.isTopModal(dialogRef.current)) return;
        if (closeOnOverlayClick) {
          onClose?.();
        }

        onOverlayClickProp?.();
      },
      [onClose, closeOnOverlayClick, onOverlayClickProp]
    );
    ```

- 접근성
  - 코드
    ```jsx
    const getDialogProps: PropGetter = useCallback(
      (props = {}, ref = null) => ({
        role: "dialog",
        ...props,
        ref: mergeRefs(ref, dialogRef),
        id: dialogId,
        **tabIndex: -1,
        "aria-modal": true,
        "aria-labelledby": headerMounted ? headerId : undefined,
        "aria-describedby": bodyMounted ? bodyId : undefined,**
        onClick: callAllHandlers(props.onClick, (event: React.MouseEvent) =>
          event.stopPropagation(),
        ),
      }),
      [bodyId, bodyMounted, dialogId, headerId, headerMounted],
    )
    ```
    ```jsx
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="dialog_label"
      aria-describedby="dialog_desc"
    >
      <h2 id="dialog_label">모달 제목</h2>
      <div id="dialog_desc">모달 내용 설명입니다.</div>
    </div>
    ```
