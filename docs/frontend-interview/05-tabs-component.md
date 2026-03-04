# Bài 5: Tabs component

## Đề bài

> "Làm component Tabs: nhận props là mảng các tab (vd `[{ id: 'a', label: 'Tab A' }, { id: 'b', label: 'Tab B' }]`) và mảng nội dung tương ứng (hoặc children). Click vào tab nào thì hiển thị nội dung tab đó. Tab đang chọn có style khác (vd active)."

*(Kiểm tra: state để chọn tab, render có điều kiện, nhận props và map.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/Tabs.tsx` — component nhận `tabs` và `contents` (hoặc `children`), state `activeId`, render danh sách nút tab + panel nội dung.
  - **Tùy chọn:** `src/types/tabs.ts` — type `TabItem`, `TabsProps` nếu muốn tách type.
  - **Chỉnh:** `src/App.tsx` — import `Tabs` và render `<Tabs tabs={...} contents={[...]} />` (kèm dữ liệu mẫu).

- **Có cần tạo thêm file không**
  - **Bắt buộc:** Không. Interface có thể khai báo trong `Tabs.tsx`.
  - **Nên thêm (tùy):** File types nếu team hay tách type; hoặc component con `TabList` / `TabPanel` nếu interviewer yêu cầu tách nhỏ.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/Tabs.tsx` | Interface `TabItem`, `TabsProps`; state `activeId`; map `tabs` → nút tab (onClick set activeId); tìm `activeIndex` / `activeContent` từ `contents`; render `<div role="tablist">` và `<div role="tabpanel">`. Toàn bộ code mẫu Tabs nằm ở đây. |
| `src/App.tsx` | Import `Tabs`, truyền `tabs` và `contents` (ví dụ 2 tab với 2 đoạn JSX). Đoạn "Dùng:" trong code mẫu chính là cách gọi trong App. |
| `src/types/tabs.ts` (tùy chọn) | `export interface TabItem { id: string; label: string }`, `export interface TabsProps { tabs: TabItem[]; contents: React.ReactNode[] }`. |

**Gợi ý khi phỏng vấn:** Có thể nói "Em dùng role tablist/tab/tabpanel để hỗ trợ a11y; có thể mở rộng nhận children thay vì contents."

---

## Cách xử lý

1. **State:** `activeId` (string) — id của tab đang chọn, khởi tạo = tab đầu tiên (vd `tabs[0].id`).
2. **Render:** Map qua `tabs` render nút/button cho mỗi tab. `onClick` set `activeId = tab.id`. Class hoặc style active khi `tab.id === activeId`.
3. **Nội dung:** Tùy đề bài:
   - Nếu truyền **contents** (mảng): tìm item có `id === activeId` (hoặc index tương ứng) rồi render.
   - Nếu truyền **children** (ReactNode[]): dùng index hoặc cloneElement; hoặc quy ước children[i] ứng với tabs[i].
4. Có thể nhận props `tabs: { id: string; label: string }[]` và `contents?: ReactNode[]` hoặc `children`.

---

## Code mẫu

**File: `src/components/Tabs.tsx`**

```tsx
import { useState } from 'react'

interface TabItem {
  id: string
  label: string
}

interface TabsProps {
  tabs: TabItem[]
  contents: React.ReactNode[] // contents[i] matches tabs[i]
}

export function Tabs({ tabs, contents }: TabsProps) {
  const [activeId, setActiveId] = useState(tabs[0]?.id ?? '')

  const activeIndex = tabs.findIndex((t) => t.id === activeId)
  const activeContent = activeIndex >= 0 ? contents[activeIndex] : null

  return (
    <div>
      <div role="tablist" style={{ display: 'flex', gap: 8, marginBottom: 16 }}>
        {tabs.map((tab) => (
          <button
            key={tab.id}
            type="button"
            role="tab"
            aria-selected={tab.id === activeId}
            onClick={() => setActiveId(tab.id)}
            style={{
              fontWeight: tab.id === activeId ? 'bold' : 'normal',
              borderBottom: tab.id === activeId ? '2px solid blue' : 'none',
            }}
          >
            {tab.label}
          </button>
        ))}
      </div>
      <div role="tabpanel">{activeContent}</div>
    </div>
  )
}
```

**File: `src/App.tsx`** — import và dùng (code này nằm trong App):

```tsx
import { Tabs } from './components/Tabs'

// In JSX, e.g.:
<Tabs
  tabs={[
    { id: 'a', label: 'Tab A' },
    { id: 'b', label: 'Tab B' },
  ]}
  contents={[<p>Content A</p>, <p>Content B</p>]}
/>
```

**Ghi chú:** Có thể mở rộng nhận `children` và dùng `React.Children.toArray(children)[activeIndex]` nếu muốn content là children thay vì prop `contents`.
