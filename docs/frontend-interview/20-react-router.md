# Bài 20: React Router — Routes + Detail URL

## Đề bài

> "Thay vì dùng state `selectedId` để chuyển giữa list và detail, hãy dùng **React Router**: route `/` hiển thị list, route `/item/:id` hiển thị detail. Khi user click vào item thì **navigate** sang `/item/123`; khi refresh trang trên detail vẫn đúng; nút Back có thể dùng **navigate(-1)** hoặc link về `/`."

*(Kiểm tra: cài đặt react-router-dom, BrowserRouter, Routes, Route, useParams, useNavigate, Link/NavLink.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Cài đặt:** `npm install react-router-dom` (trong react-demo).
  - **Chỉnh:** `src/main.tsx` — bọc `<App />` bằng `<BrowserRouter>` (hoặc bọc trong App).
  - **Chỉnh:** `src/App.tsx` — dùng `<Routes>`, `<Route path="/" element={<ListPage />} />`, `<Route path="/item/:id" element={<DetailPage />} />`. ListPage và DetailPage có thể là component trong cùng file hoặc tách.
  - **Tạo mới:** `src/pages/ListPage.tsx` — fetch list, render list; mỗi item dùng `<Link to={\`/item/${item.id}\`}>` hoặc `useNavigate()` khi click.
  - **Tạo mới:** `src/pages/DetailPage.tsx` — `const { id } = useParams<{ id: string }>()`; fetch detail theo id (hoặc dùng data từ context); render detail; nút Back gọi `navigate(-1)` hoặc `<Link to="/">Back</Link>`.

- **Có cần tạo thêm file không**
  - Cần thêm `react-router-dom`. Có thể tách routes ra file `src/routes.tsx` hoặc giữ trong App.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/main.tsx` | Wrap root with `<BrowserRouter><App /></BrowserRouter>`. |
| `src/App.tsx` | `<Routes><Route path="/" element={<ListPage />} /><Route path="/item/:id" element={<DetailPage />} /></Routes>`. |
| `src/pages/ListPage.tsx` | List data; map to `<Link to={\`/item/${id}\`}>` or onClick navigate(\`/item/${id}\`). |
| `src/pages/DetailPage.tsx` | useParams().id; fetch or get detail; useNavigate() for Back. |

---

## Cách xử lý

1. **BrowserRouter:** Bọc toàn bộ app (thường ở main.tsx) để dùng được useNavigate, useParams, Link.
2. **Routes + Route:** Định nghĩa path và element; `path="/item/:id"` cho dynamic segment.
3. **useParams():** Trong DetailPage, `const { id } = useParams<{ id: string }>()` — id là string, parse sang number nếu cần.
4. **Navigation:** Từ list sang detail: `<Link to={\`/item/${item.id}\`}>` hoặc `const navigate = useNavigate(); onClick={() => navigate(\`/item/${id}\`)}`. Back: `navigate(-1)` hoặc `<Link to="/">Back</Link>`.
5. **Detail fetch:** Trong DetailPage, useEffect fetch khi id thay đổi; loading và error state.

---

## Code mẫu

### File: `src/main.tsx`

```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import App from './App'
import './style.css'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
)
```

### File: `src/App.tsx`

```tsx
import { Routes, Route } from 'react-router-dom'
import { ListPage } from './pages/ListPage'
import { DetailPage } from './pages/DetailPage'

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<ListPage />} />
      <Route path="/item/:id" element={<DetailPage />} />
    </Routes>
  )
}
```

### File: `src/pages/ListPage.tsx`

```tsx
import { useState, useEffect } from 'react'
import { Link } from 'react-router-dom'

interface Item {
  id: number
  title: string
}

export function ListPage() {
  const [items, setItems] = useState<Item[]>([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/posts?_limit=10')
      .then((res) => res.json())
      .then((data: Item[]) => setItems(data))
      .finally(() => setLoading(false))
  }, [])

  if (loading) return <p>Loading...</p>

  return (
    <div style={{ padding: 24 }}>
      <h1>List</h1>
      <ul>
        {items.map((item) => (
          <li key={item.id}>
            <Link to={`/item/${item.id}`}>{item.title}</Link>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

### File: `src/pages/DetailPage.tsx`

```tsx
import { useState, useEffect } from 'react'
import { useParams, useNavigate } from 'react-router-dom'

interface Item {
  id: number
  title: string
  body: string
}

export function DetailPage() {
  const { id } = useParams<{ id: string }>()
  const navigate = useNavigate()
  const [item, setItem] = useState<Item | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    if (!id) return
    setLoading(true)
    fetch(`https://jsonplaceholder.typicode.com/posts/${id}`)
      .then((res) => res.json())
      .then((data: Item) => setItem(data))
      .finally(() => setLoading(false))
  }, [id])

  if (loading) return <p>Loading...</p>
  if (!item) return <p>Not found</p>

  return (
    <div style={{ padding: 24 }}>
      <button type="button" onClick={() => navigate(-1)}>
        Back
      </button>
      <h1>{item.title}</h1>
      <p>{item.body}</p>
    </div>
  )
}
```

**Ghi chú:** Cần chạy `npm install react-router-dom` trong project. Nếu dùng Vite + React, type cho `useParams` có thể cần `import type { useParams } from 'react-router-dom'` tùy phiên bản.
