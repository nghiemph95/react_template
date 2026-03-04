# Bài 4: Debounced Search

## Đề bài

> "Ô input search: khi user gõ, không gọi API mỗi lần gõ một ký tự mà đợi sau khi user **ngừng gõ 300ms** mới gọi API (hoặc set state search term để component khác dùng). Mô phỏng gọi API bằng console.log(searchTerm)."

*(Kiểm tra: useEffect, cleanup, debounce / dependency vào giá trị đã debounce.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/DebouncedSearch.tsx` — input controlled, state `inputValue` và `searchTerm`, 2 useEffect (debounce + "gọi API").
  - **Tùy chọn:** `src/hooks/useDebounce.ts` — custom hook `useDebounce(value, delay)` trả về giá trị đã debounce (tách ra nếu muốn tái sử dụng).
  - **Chỉnh:** `src/App.tsx` — import và render `<DebouncedSearch />`.

- **Có cần tạo thêm file không**
  - **Bắt buộc:** Không. Toàn bộ logic có thể nằm trong một component.
  - **Nên thêm (tùy):** `src/hooks/useDebounce.ts` nếu interviewer hỏi "debounce dùng lại chỗ khác không?" — "Em có thể tách thành hook useDebounce(value, delay) để dùng cho mọi input cần debounce."

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/DebouncedSearch.tsx` | State `inputValue`, `searchTerm`; useEffect debounce (setTimeout 300ms, cleanup clearTimeout); useEffect với deps `[searchTerm]` chứa console.log (hoặc fetch); JSX input value/onChange. Code mẫu bên dưới nằm trong file này. |
| `src/hooks/useDebounce.ts` (tùy chọn) | `export function useDebounce<T>(value: T, delay: number): T` — state nội bộ + useEffect set value sau delay, return value debounced. Component chỉ cần `const searchTerm = useDebounce(inputValue, 300)`. |
| `src/App.tsx` | `import { DebouncedSearch } from './components/DebouncedSearch'` và `<DebouncedSearch />`. |

**Gợi ý khi phỏng vấn:** Có thể nói "300ms em để constant ở đầu file; trong project thật có thể đưa vào config hoặc prop."

---

## Cách xử lý

1. **State:** `inputValue` (string, value của input), có thể thêm `searchTerm` (string — giá trị đã debounce, dùng để "gọi API").
2. Input: **controlled** — `value={inputValue}`, `onChange` set inputValue.
3. **useEffect** với dependency `[inputValue]`:
   - Set một `timer = setTimeout(() => setSearchTerm(inputValue), 300)` (hoặc gọi API trực tiếp trong timeout).
   - **Cleanup:** `return () => clearTimeout(timer)`.
4. Một **useEffect** khác với dependency `[searchTerm]`: khi searchTerm thay đổi thì gọi API (ở đây `console.log(searchTerm)`).

Hoặc gộp: trong timeout của bước 3 gọi luôn API thay vì set searchTerm, tùy cách tổ chức.

---

## Code mẫu

**File: `src/components/DebouncedSearch.tsx`**

```tsx
import { useEffect, useState } from 'react'

const DEBOUNCE_MS = 300

export function DebouncedSearch() {
  const [inputValue, setInputValue] = useState('')
  const [searchTerm, setSearchTerm] = useState('')

  // Debounce: update searchTerm only after user stops typing for DEBOUNCE_MS
  useEffect(() => {
    const timer = setTimeout(() => {
      setSearchTerm(inputValue)
    }, DEBOUNCE_MS)
    return () => clearTimeout(timer)
  }, [inputValue])

  // Call API when searchTerm changes (in production: fetch(searchTerm))
  useEffect(() => {
    if (!searchTerm) return
    console.log('Search API with:', searchTerm)
    // fetch(`/api/search?q=${searchTerm}`)...
  }, [searchTerm])

  return (
    <input
      type="text"
      placeholder="Search..."
      value={inputValue}
      onChange={(e) => setInputValue(e.target.value)}
    />
  )
}
```

**File: `src/App.tsx`** — thêm:

```tsx
import { DebouncedSearch } from './components/DebouncedSearch'
// in JSX: <DebouncedSearch />
```

**Ghi chú:** Cleanup `clearTimeout(timer)` đảm bảo nếu user gõ tiếp trong 300ms thì lần timeout cũ bị hủy, chỉ lần cuối mới chạy.
