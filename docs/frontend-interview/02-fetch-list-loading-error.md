# Bài 2: Fetch API — List + Loading & Error

## Đề bài

> "Viết component gọi API lấy danh sách users (ví dụ `https://jsonplaceholder.typicode.com/users`). Khi đang load thì hiển thị 'Loading...', khi lỗi thì hiển thị thông báo lỗi, khi thành công thì render list tên user. Dùng useEffect."

*(Bài này kiểm tra: useEffect, fetch/async, quản lý state loading/error/data.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/UserList.tsx` — component gọi API, quản lý `users` / `loading` / `error`, render list hoặc loading/error.
  - **Tùy chọn:** `src/types/user.ts` — nếu muốn tách type `User` ra file riêng (interviewer có thể hỏi "type để đâu?").
  - **Chỉnh:** `src/App.tsx` — import `UserList` và render `<UserList />`.

- **Có cần tạo thêm file không**
  - **Bắt buộc:** Không. Có thể khai báo `interface User` ngay trong `UserList.tsx`.
  - **Nên thêm (tùy quy ước):** `src/types/user.ts` nếu team hay tách type ra folder `types/` — khi phỏng vấn có thể nói: "Trong project em có thể tách type vào `types/user.ts` để dùng chung."

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/UserList.tsx` | Interface `User` (hoặc import từ types), state `users` / `loading` / `error`, `useEffect` gọi fetch, cleanup `cancelled`, render loading / error / `<ul>`. Toàn bộ logic và JSX trong file này. |
| `src/types/user.ts` (tùy chọn) | `export interface User { id: number; name: string; ... }` — chỉ khi tách type. |
| `src/App.tsx` | `import { UserList } from './components/UserList'` và `<UserList />`. |

**Gợi ý khi phỏng vấn:** Nếu đề cho URL cứng, có thể nói "Phần sau em có thể đưa URL vào env hoặc prop để dễ đổi." Thể hiện ý thức cấu hình và tái sử dụng.

---

## Cách xử lý

1. **State:** `users` (array hoặc null), `loading` (boolean), `error` (string hoặc null).
2. **useEffect** với dependency `[]` (chạy 1 lần khi mount):
   - Set `loading = true`, `error = null`.
   - Gọi `fetch(url)`, `await response.json()`.
   - Nếu `!response.ok` → set error, không set users.
   - Nếu ok → set users, clear error.
   - Trong `finally` (hoặc cả success/error) set `loading = false`.
3. **Try/catch** quanh fetch để bắt lỗi mạng hoặc parse.
4. **Render:**
   - Nếu `loading` → return "Loading...".
   - Nếu `error` → return thông báo lỗi.
   - Còn lại → render `<ul>{users.map(...)}</ul>` (key = user.id hoặc index nếu không có id).

---

## Code mẫu

**File: `src/components/UserList.tsx`**

```tsx
import { useEffect, useState } from 'react'

interface User {
  id: number
  name: string
}

export function UserList() {
  const [users, setUsers] = useState<User[] | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    let cancelled = false
    setLoading(true)
    setError(null)
    fetch('https://jsonplaceholder.typicode.com/users')
      .then((res) => {
        if (!res.ok) throw new Error('Failed to fetch')
        return res.json()
      })
      .then((data: User[]) => {
        if (!cancelled) setUsers(data)
      })
      .catch((e) => {
        if (!cancelled) setError(e.message ?? 'Unknown error')
      })
      .finally(() => {
        if (!cancelled) setLoading(false)
      })
    return () => { cancelled = true }
  }, [])

  if (loading) return <p>Loading...</p>
  if (error) return <p>Error: {error}</p>
  if (!users?.length) return <p>No users</p>

  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  )
}
```

**File: `src/App.tsx`** — thêm:

```tsx
import { UserList } from './components/UserList'
// in JSX: <UserList />
```

**Ghi chú:** Biến `cancelled` dùng để tránh setState sau khi component đã unmount (cleanup trong useEffect).
