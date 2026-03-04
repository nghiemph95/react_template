# Bài 13: Pagination

## Đề bài

> "Có một mảng items (vd 50 phần tử). Hiển thị 10 item mỗi trang; state currentPage (1-based). Nút Previous / Next (Previous disabled khi page 1, Next disabled khi trang cuối). Hiển thị 'Page X of Y' và danh sách items của trang hiện tại."

*(Kiểm tra: slice array theo page, disabled button, derived state.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/PaginationList.tsx` — props items (array), pageSize (default 10); state currentPage; slice items cho trang hiện tại; nút Prev/Next; hiển thị "Page X of Y" và list.
  - **Chỉnh:** `src/App.tsx` — tạo mảng mẫu, render PaginationList.

- **Có cần tạo thêm file không**
  - Không. Có thể nhận items từ props hoặc hardcode trong component.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/PaginationList.tsx` | Props items, pageSize; useState currentPage; totalPages = ceil(length/pageSize); paginated = items.slice((page-1)*pageSize, page*pageSize); Prev/Next setPage. |
| `src/App.tsx` | Mock items, <PaginationList items={items} pageSize={10} />. |

---

## Cách xử lý

1. **totalPages:** `Math.ceil(items.length / pageSize)`.
2. **Items của trang:** `items.slice((currentPage - 1) * pageSize, currentPage * pageSize)`.
3. **Previous:** onClick set currentPage(p => Math.max(1, p - 1)); disabled khi currentPage === 1.
4. **Next:** onClick set currentPage(p => Math.min(totalPages, p + 1)); disabled khi currentPage === totalPages.

---

## Code mẫu

**File: `src/components/PaginationList.tsx`**

```tsx
import { useState, ReactNode } from 'react'

interface PaginationListProps<T> {
  items: T[]
  pageSize?: number
  renderItem?: (item: T, index: number) => ReactNode
}

export function PaginationList<T>({
  items,
  pageSize = 10,
  renderItem = (item) => String(item),
}: PaginationListProps<T>) {
  const [currentPage, setCurrentPage] = useState(1)

  const totalPages = Math.ceil(items.length / pageSize) || 1
  const start = (currentPage - 1) * pageSize
  const pageItems = items.slice(start, start + pageSize)

  return (
    <div>
      <p>Page {currentPage} of {totalPages}</p>
      <button
        type="button"
        disabled={currentPage <= 1}
        onClick={() => setCurrentPage((p) => p - 1)}
      >
        Previous
      </button>
      <button
        type="button"
        disabled={currentPage >= totalPages}
        onClick={() => setCurrentPage((p) => p + 1)}
      >
        Next
      </button>
      <ul>
        {pageItems.map((item, i) => (
          <li key={start + i}>{renderItem(item, start + i)}</li>
        ))}
      </ul>
    </div>
  )
}
```

**File: `src/App.tsx`** — example:

```tsx
import { PaginationList } from './components/PaginationList'

const ITEMS = Array.from({ length: 50 }, (_, i) => `Item ${i + 1}`)

// In JSX:
<PaginationList items={ITEMS} pageSize={10} />
```
