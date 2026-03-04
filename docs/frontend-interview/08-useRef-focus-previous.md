# Bài 8: useRef — focus input & previous value

## Đề bài

> "1) Component có một nút 'Focus input': bấm vào thì focus vào ô input (dùng useRef). 2) Component có state count; hiển thị count và giá trị count **trước đó** (previous). Khi count thay đổi, previous phải là giá trị cũ (dùng useRef lưu previous)."

*(Kiểm tra: useRef cho DOM ref và cho lưu value không gây re-render.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/FocusInput.tsx` — ref gắn input, button click → ref.current?.focus().
  - **Tạo mới:** `src/components/PreviousCount.tsx` — state count, ref previousCount; useEffect cập nhật ref sau mỗi render (ref.current = count) rồi set state; hoặc trong render: hiển thị ref.current, sau đó ref.current = count.
  - **Chỉnh:** `src/App.tsx` — import và render cả hai (hoặc gộp thành một bài).

- **Có cần tạo thêm file không**
  - Có thể gộp hai phần vào một file `src/components/UseRefDemo.tsx` với hai section.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/FocusInput.tsx` | useRef<HTMLInputElement>(null), input ref={inputRef}, button onClick={() => inputRef.current?.focus()}. |
| `src/components/PreviousCount.tsx` | useState count, useRef previousCount; trước return: const prev = previousCount.current; previousCount.current = count; hiển thị count và prev; button tăng count. |
| `src/App.tsx` | Import and render both. |

---

## Cách xử lý

1. **Focus:** `const inputRef = useRef<HTMLInputElement>(null)`. Input có `ref={inputRef}`. Button `onClick` gọi `inputRef.current?.focus()`.
2. **Previous value:** `const [count, setCount] = useState(0)`, `const prevRef = useRef<number>()`. Trong render (trước return): lưu `const prev = prevRef.current` (giá trị cũ), sau đó `prevRef.current = count` (chuẩn bị cho lần render sau). Hiển thị `count` và `prev` (lần đầu prev có thể undefined).

---

## Code mẫu

**File: `src/components/FocusInput.tsx`**

```tsx
import { useRef } from 'react'

export function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null)

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="I will be focused" />
      <button type="button" onClick={() => inputRef.current?.focus()}>
        Focus input
      </button>
    </div>
  )
}
```

**File: `src/components/PreviousCount.tsx`**

```tsx
import { useRef, useState } from 'react'

export function PreviousCount() {
  const [count, setCount] = useState(0)
  const prevCountRef = useRef<number | undefined>(undefined)

  const previousCount = prevCountRef.current
  prevCountRef.current = count

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {previousCount ?? '—'}</p>
      <button type="button" onClick={() => setCount((c) => c + 1)}>
        Increment
      </button>
    </div>
  )
}
```

**File: `src/App.tsx`** — add imports and render `<FocusInput />`, `<PreviousCount />`.
