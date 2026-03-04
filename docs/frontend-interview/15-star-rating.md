# Bài 15: Star Rating (clickable)

## Đề bài

> "Component Star Rating: 5 ngôi sao, nhận prop value (1–5) và onChange (callback). Click vào sao thứ n thì gọi onChange(n). Hover vào sao thứ n thì highlight từ 1 đến n (có thể dùng state hoverIndex); khi mouse leave thì về lại value. Hiển thị value (số) bên cạnh."

*(Kiểm tra: controlled component, callback prop, hover state, render list with index.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/StarRating.tsx` — props value (number), onChange (value: number) => void; state hoverIndex (number | null) cho hover; map 1..5 render star (filled nếu index <= (hoverIndex ?? value)); onClick star gọi onChange(index); onMouseEnter/Leave set hoverIndex.
  - **Chỉnh:** `src/App.tsx` — state rating, <StarRating value={rating} onChange={setRating} />, hiển thị rating.

- **Có cần tạo thêm file không**
  - Không.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/StarRating.tsx` | Props value, onChange; hoverIndex state; 5 stars, filled by index <= (hoverIndex ?? value); click => onChange(i), mouse enter/leave. |
| `src/App.tsx` | useState rating, StarRating value/onChange, display rating. |

---

## Cách xử lý

1. **Display value:** Khi hover: highlight 1..hoverIndex; khi không hover: highlight 1..value. Dùng `const display = hoverIndex ?? value` (hoặc hoverIndex != null ? hoverIndex : value).
2. **Star i (1..5):** filled khi `i <= display`. Click gọi `onChange(i)`; onMouseEnter set hoverIndex(i); onMouseLeave set hoverIndex(null).
3. **Star icon:** Có thể dùng ký tự '★' / '☆' hoặc SVG. Filled = '★' (hoặc color vàng), empty = '☆' (xám).

---

## Code mẫu

**File: `src/components/StarRating.tsx`**

```tsx
import { useState } from 'react'

interface StarRatingProps {
  value: number
  onChange: (value: number) => void
}

const MAX = 5

export function StarRating({ value, onChange }: StarRatingProps) {
  const [hover, setHover] = useState<number | null>(null)
  const display = hover ?? value

  return (
    <div style={{ display: 'flex', alignItems: 'center', gap: 4 }}>
      {Array.from({ length: MAX }, (_, i) => i + 1).map((i) => (
        <span
          key={i}
          role="button"
          tabIndex={0}
          style={{
            cursor: 'pointer',
            color: i <= display ? '#ffc107' : '#ccc',
            fontSize: 24,
          }}
          onClick={() => onChange(i)}
          onMouseEnter={() => setHover(i)}
          onMouseLeave={() => setHover(null)}
          onKeyDown={(e) => e.key === 'Enter' && onChange(i)}
        >
          ★
        </span>
      ))}
      <span style={{ marginLeft: 8 }}>{value} / {MAX}</span>
    </div>
  )
}
```

**File: `src/App.tsx`** — example:

```tsx
import { useState } from 'react'
import { StarRating } from './components/StarRating'

// In component:
const [rating, setRating] = useState(0)
// In JSX:
<StarRating value={rating} onChange={setRating} />
<p>You selected: {rating}</p>
```
