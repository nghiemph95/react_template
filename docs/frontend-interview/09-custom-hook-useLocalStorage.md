# Bài 9: Custom Hook — useLocalStorage

## Đề bài

> "Viết custom hook `useLocalStorage(key, initialValue)`: trả về [value, setValue] giống useState, nhưng value được đồng bộ với localStorage (key). Khi mount đọc từ localStorage (parse JSON), khi setValue gọi thì cập nhật state và localStorage.setItem. Nếu localStorage không có giá trị thì dùng initialValue."

*(Kiểm tra: custom hook, useState + useEffect/sync localStorage, JSON parse/stringify.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/hooks/useLocalStorage.ts` — hook nhận key (string) và initialValue (T), return [value, setValue]; đọc localStorage.getItem(key) khi init; setValue cập nhật state + setItem.
  - **Tạo mới:** `src/components/LocalStorageDemo.tsx` — dùng useLocalStorage('name', ''), input controlled, hiển thị value (để test persist sau refresh).
  - **Chỉnh:** `src/App.tsx` — render LocalStorageDemo.

- **Có cần tạo thêm file không**
  - Hook trong `src/hooks/`, component demo trong `src/components/`.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/hooks/useLocalStorage.ts` | function useLocalStorage<T>(key, initialValue): [T, (v: T \| ((prev: T) => T)) => void]; useState với lazy init đọc từ localStorage; setValue wrapper gọi setState + setItem. |
| `src/components/LocalStorageDemo.tsx` | useLocalStorage('demo', ''), input value/onChange, p hiển thị value. |
| `src/App.tsx` | Import and render LocalStorageDemo. |

---

## Cách xử lý

1. **Lazy initial state:** `useState<T>(() => { const raw = localStorage.getItem(key); if (raw == null) return initialValue; try { return JSON.parse(raw) as T; } catch { return initialValue; } })`.
2. **setValue:** Khi gọi setValue (hoặc setValue(fn)), cần cập nhật state và localStorage. Có thể dùng useEffect sync state → localStorage khi value thay đổi; hoặc wrapper setValue: gọi setState, trong callback hoặc sau đó setItem (lưu ý setState bất đồng bộ — có thể setItem trong setState callback).
3. **setItem:** `localStorage.setItem(key, JSON.stringify(newValue))`.

---

## Code mẫu

**File: `src/hooks/useLocalStorage.ts`**

```tsx
import { useState, useCallback } from 'react'

export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  const [stored, setStored] = useState<T>(() => {
    try {
      const item = localStorage.getItem(key)
      return item != null ? (JSON.parse(item) as T) : initialValue
    } catch {
      return initialValue
    }
  })

  const setValue = useCallback(
    (value: T | ((prev: T) => T)) => {
      setStored((prev) => {
        const next = value instanceof Function ? value(prev) : value
        try {
          localStorage.setItem(key, JSON.stringify(next))
        } catch {
          // ignore
        }
        return next
      })
    },
    [key]
  )

  return [stored, setValue]
}
```

**File: `src/components/LocalStorageDemo.tsx`**

```tsx
import { useLocalStorage } from '../hooks/useLocalStorage'

export function LocalStorageDemo() {
  const [name, setName] = useLocalStorage('demo-name', '')

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Your name (persists in localStorage)"
      />
      <p>Stored: {name || '—'}</p>
    </div>
  )
}
```

**File: `src/App.tsx`** — add: `import { LocalStorageDemo } from './components/LocalStorageDemo'`, in JSX: `<LocalStorageDemo />`.
