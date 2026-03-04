# Bài 14: Accordion / Collapse

## Đề bài

> "Component Accordion: nhận mảng items (vd [{ id, title, content }]). Mỗi item là một section: click vào title thì mở/đóng phần content (chỉ một section mở tại một thời điểm — mở cái mới thì đóng cái cũ). Section đang mở có style khác (vd bold title)."

*(Kiểm tra: state which id is open, conditional render content, single open at a time.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/Accordion.tsx` — props items; state openId (string | null); map items: title onClick toggle openId (nếu click vào đang mở thì set null), content hiển thị khi item.id === openId.
  - **Chỉnh:** `src/App.tsx` — truyền items mẫu, render Accordion.

- **Có cần tạo thêm file không**
  - Không. Type cho item có thể inline.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/Accordion.tsx` | Props items: { id, title, content }[]; useState openId; map: button/label (title), onClick set openId to id or null; div content when open. |
| `src/App.tsx` | Sample items, <Accordion items={...} />. |

---

## Cách xử lý

1. **State:** `openId: string | null` — id của section đang mở, null = tất cả đóng.
2. **Click title:** `setOpenId(prev => prev === item.id ? null : item.id)` — toggle: nếu đang mở thì đóng, không thì mở cái này (đóng cái khác).
3. **Content:** Chỉ render content khi `item.id === openId`.

---

## Code mẫu

**File: `src/components/Accordion.tsx`**

```tsx
import { useState } from 'react'

interface AccordionItem {
  id: string
  title: string
  content: React.ReactNode
}

interface AccordionProps {
  items: AccordionItem[]
}

export function Accordion({ items }: AccordionProps) {
  const [openId, setOpenId] = useState<string | null>(null)

  return (
    <div>
      {items.map((item) => {
        const isOpen = openId === item.id
        return (
          <div key={item.id} style={{ border: '1px solid #ccc', marginBottom: 8 }}>
            <button
              type="button"
              onClick={() => setOpenId((prev) => (prev === item.id ? null : item.id))}
              style={{
                width: '100%',
                textAlign: 'left',
                padding: 12,
                fontWeight: isOpen ? 'bold' : 'normal',
                background: isOpen ? '#f0f0f0' : 'transparent',
              }}
            >
              {item.title}
            </button>
            {isOpen && (
              <div style={{ padding: 12, borderTop: '1px solid #ccc' }}>
                {item.content}
              </div>
            )}
          </div>
        )
      })}
    </div>
  )
}
```

**File: `src/App.tsx`** — example:

```tsx
import { Accordion } from './components/Accordion'

const ITEMS = [
  { id: '1', title: 'Section 1', content: <p>Content for section 1.</p> },
  { id: '2', title: 'Section 2', content: <p>Content for section 2.</p> },
  { id: '3', title: 'Section 3', content: <p>Content for section 3.</p> },
]

// In JSX:
<Accordion items={ITEMS} />
```
