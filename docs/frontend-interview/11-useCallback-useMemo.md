# Bài 11: useCallback & useMemo

## Đề bài

> "1) Component cha truyền callback xuống component con (vd onClick). Con được bọc React.memo. Giải thích tại sao con vẫn re-render khi cha re-render, và dùng useCallback để callback không đổi reference mỗi lần cha render. 2) Có một list dài và hàm filter/sort tốn CPU: chỉ khi list hoặc filter thay đổi mới tính lại, dùng useMemo."

*(Kiểm tra: useCallback tránh tạo function mới mỗi render; useMemo cache kết quả tính toán.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/UseCallbackDemo.tsx` — cha có state, truyền onClick xuống con; con (React.memo) nhận onClick; cha dùng useCallback(onClick, [deps]).
  - **Tạo mới:** `src/components/UseMemoDemo.tsx` — state list + filterTerm; filteredList = useMemo(() => list.filter(...), [list, filterTerm]); render filteredList.
  - **Chỉnh:** `src/App.tsx` — render cả hai (hoặc gộp).

- **Có cần tạo thêm file không**
  - Có thể tách component con thành file riêng cho phần useCallback.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/UseCallbackDemo.tsx` | Parent: useState, useCallback for handler; Child (memo): nhận onClick, console.log('Child render'). |
| `src/components/UseMemoDemo.tsx` | useState list, filterTerm; useMemo filtered; input + ul. |
| `src/App.tsx` | Import and render. |

---

## Cách xử lý

1. **useCallback:** `const handleClick = useCallback(() => { ... }, [dep1, dep2])`. Con bọc bằng `React.memo(Child)` — con chỉ re-render khi props thay đổi (reference); nếu cha tạo function mới mỗi lần thì onClick !== prev.onClick nên con vẫn re-render. useCallback trả về cùng reference khi deps không đổi.
2. **useMemo:** `const filtered = useMemo(() => items.filter(...), [items, filterTerm])`. Chỉ khi items hoặc filterTerm đổi mới chạy lại filter.

---

## Code mẫu

**File: `src/components/UseCallbackDemo.tsx`**

```tsx
import { useState, useCallback, memo } from 'react'

const Child = memo(function Child({ onClick }: { onClick: () => void }) {
  console.log('Child render')
  return <button type="button" onClick={onClick}>Click (child)</button>
})

export function UseCallbackDemo() {
  const [count, setCount] = useState(0)
  const handleClick = useCallback(() => setCount((c) => c + 1), [])

  return (
    <div>
      <p>Count: {count}</p>
      <Child onClick={handleClick} />
    </div>
  )
}
```

**File: `src/components/UseMemoDemo.tsx`**

```tsx
import { useState, useMemo } from 'react'

const ITEMS = Array.from({ length: 100 }, (_, i) => ({ id: i, name: `Item ${i}` }))

export function UseMemoDemo() {
  const [filter, setFilter] = useState('')

  const filtered = useMemo(
    () => ITEMS.filter((item) => item.name.toLowerCase().includes(filter.toLowerCase())),
    [filter]
  )

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter"
      />
      <ul>{filtered.map((item) => <li key={item.id}>{item.name}</li>)}</ul>
    </div>
  )
}
```

**File: `src/App.tsx`** — add imports and render `<UseCallbackDemo />`, `<UseMemoDemo />`.
