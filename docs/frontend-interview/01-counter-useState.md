# Bài 1: Counter với useState

## Đề bài

> "Cho tôi một component React: có một số đếm (counter) và hai nút **Tăng** / **Giảm**. Bấm Tăng thì số tăng 1, bấm Giảm thì số giảm 1. Số không được âm."

*(Câu hỏi kiểu này thường dùng để kiểm tra ứng viên đã dùng useState và xử lý event cơ bản.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/Counter.tsx` — chứa toàn bộ logic counter và JSX (state + 2 nút + hiển thị số).
  - **Chỉnh:** `src/App.tsx` — import `Counter` và render `<Counter />` để xem trên trang (vd thay hoặc thêm vào nội dung hiện tại).

- **Có cần tạo thêm file không**
  - **Không.** Chỉ cần 1 component trong 1 file. Không bắt buộc file type/constant riêng.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/Counter.tsx` | Component `Counter`: `useState(0)`, hai `button` onClick tăng/giảm, hiển thị `count`. Toàn bộ code mẫu bên dưới nằm trong file này. |
| `src/App.tsx` | Thêm `import { Counter } from './components/Counter'` và trong JSX thêm (hoặc thay) bằng `<Counter />`. |

**Gợi ý khi phỏng vấn:** Nếu interviewer chỉ nói "viết component", có thể hỏi rõ: "Em tạo file component mới `Counter.tsx` trong `src/components` và dùng trong `App` nhé?" để thể hiện ý thức cấu trúc project.

---

## Cách xử lý

1. Dùng **useState** lưu giá trị counter (number), khởi tạo 0.
2. Hai nút: **onClick** gọi setState — một cái `setCount(c => c + 1)`, một cái `setCount(c => c - 1)` (hoặc `c > 0 ? c - 1 : 0` nếu không cho âm).
3. Hiển thị `count` trong JSX (giữa hoặc cạnh nút).
4. Nếu yêu cầu "không âm": khi giảm, chỉ gọi setState khi `count > 0`, hoặc dùng `Math.max(0, count - 1)`.

---

## Code mẫu

**File: `src/components/Counter.tsx`**

```tsx
import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <span>Count: {count}</span>
      <button type="button" onClick={() => setCount((c) => c + 1)}>
        Increment
      </button>
      <button
        type="button"
        onClick={() => setCount((c) => (c > 0 ? c - 1 : 0))}
      >
        Decrement
      </button>
    </div>
  )
}
```

**File: `src/App.tsx`** — thêm import và render (ví dụ trong `return`):

```tsx
import { Counter } from './components/Counter'
// ... in JSX add: <Counter />
```

**Ghi chú:** Dùng `setCount(c => c + 1)` thay vì `setCount(count + 1)` để tránh stale closure khi có nhiều update liên tiếp.
