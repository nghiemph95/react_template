# Bài 21: React.lazy + Suspense — Code splitting

## Đề bài

> "Không load toàn bộ component cùng lúc khi vào app. Hãy dùng **React.lazy** để load **lazy** một component (vd trang Detail hoặc một tab nặng); bọc bằng **Suspense** với prop **fallback** (vd spinner hoặc 'Loading...'). Khi user navigate tới route đó (hoặc mở tab đó) thì component mới được tải — giảm bundle size ban đầu."

*(Kiểm tra: React.lazy(() => import(...)), Suspense, fallback. Có thể kết hợp với React Router: lazy load route component.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/pages/HeavyPage.tsx` — component "nặng" (vd nhiều logic hoặc import lớn); export default. Sẽ được lazy load.
  - **Chỉnh:** `src/App.tsx` — thay vì `import { HeavyPage } from './pages/HeavyPage'`, dùng `const HeavyPage = React.lazy(() => import('./pages/HeavyPage'))`; bọc `<Suspense fallback={<div>Loading...</div>}><HeavyPage /></Suspense>` (hoặc bọc route element). Khi render HeavyPage lần đầu, React load chunk và hiển thị fallback đến khi xong.

- **Có cần tạo thêm file không**
  - Có thể tách nhiều page và lazy từng cái; hoặc lazy từng route trong React Router.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/pages/HeavyPage.tsx` | Component bình thường, export default. (Có thể import thư viện nặng để thấy chunk tách.) |
| `src/App.tsx` | lazy(() => import('./pages/HeavyPage')); Suspense wrap component hoặc Route element; fallback UI. |

---

## Cách xử lý

1. **React.lazy:** Nhận function trả về `import()` — dynamic import. `const HeavyPage = lazy(() => import('./pages/HeavyPage'))`. Component phải **export default**.
2. **Suspense:** Bọc component lazy; prop **fallback** là ReactNode hiển thị khi component đang load (chunk chưa tải xong). Khi chunk load xong, React render component thật.
3. **Vị trí:** Thường bọc từng route: `<Route path="/heavy" element={<Suspense fallback={...}><HeavyPage /></Suspense>} />`. Hoặc bọc một nhóm route.
4. **Lỗi load:** Nếu import() fail (network error), có thể bọc thêm Error Boundary để bắt và hiển thị fallback.

---

## Code mẫu

### File: `src/pages/HeavyPage.tsx`

```tsx
export default function HeavyPage() {
  return (
    <div style={{ padding: 24 }}>
      <h1>Heavy / Lazy-loaded page</h1>
      <p>This component is loaded only when needed (code splitting).</p>
    </div>
  )
}
```

### File: `src/App.tsx`

```tsx
import { lazy, Suspense, useState } from 'react'

const HeavyPage = lazy(() => import('./pages/HeavyPage'))

export default function App() {
  const [showHeavy, setShowHeavy] = useState(false)

  return (
    <div style={{ padding: 24 }}>
      <h1>Home</h1>
      <button type="button" onClick={() => setShowHeavy(true)}>
        Load heavy page
      </button>
      {showHeavy && (
        <Suspense fallback={<p>Loading page...</p>}>
          <HeavyPage />
        </Suspense>
      )}
    </div>
  )
}
```

### Kết hợp với React Router (optional)

```tsx
import { lazy, Suspense } from 'react'
import { Routes, Route } from 'react-router-dom'

const ListPage = lazy(() => import('./pages/ListPage'))
const DetailPage = lazy(() => import('./pages/DetailPage'))

export default function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<ListPage />} />
        <Route path="/item/:id" element={<DetailPage />} />
      </Routes>
    </Suspense>
  )
}
```

**Ghi chú:** Mỗi `lazy(() => import(...))` tạo một chunk riêng (bundle split). Build (vd Vite) sẽ tách file; khi route hoặc component được render lần đầu, browser mới tải chunk đó. Fallback nên nhẹ (spinner, text) để UX tốt.
