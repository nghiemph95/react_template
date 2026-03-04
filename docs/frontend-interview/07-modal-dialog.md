# Bài 7: Modal / Dialog

## Đề bài

> "Làm component Modal: nhận prop `open` (boolean) và `onClose` (callback). Khi open=true hiển thị overlay (nền tối) và nội dung modal ở giữa; click overlay hoặc nút Close gọi onClose. Có thể nhận `children` làm nội dung modal."

*(Kiểm tra: conditional render, portal (optional), overlay click, callback props.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/Modal.tsx` — nhận `open`, `onClose`, `children`; render overlay + box khi open; onClick overlay và nút Close gọi onClose.
  - **Chỉnh:** `src/App.tsx` — state `modalOpen`, nút mở modal, `<Modal open={modalOpen} onClose={() => setModalOpen(false)}>content</Modal>`.

- **Có cần tạo thêm file không**
  - Không. Có thể dùng `ReactDOM.createPortal` để render modal vào `document.body` (tránh z-index) — tùy yêu cầu.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/Modal.tsx` | Props `open`, `onClose`, `children`; if (!open) return null; div overlay (position fixed, onClick onClose) + div content (stopPropagation); button Close. |
| `src/App.tsx` | useState modalOpen; button "Open modal" set true; Modal with onClose set false. |

---

## Cách xử lý

1. Nếu `!open` return `null`.
2. Overlay: `position: fixed`, full screen, `backgroundColor: 'rgba(0,0,0,0.5)'`, `onClick={onClose}`.
3. Content box: `position: fixed` center (hoặc flex center trên overlay), `onClick={(e) => e.stopPropagation()}` để click vào box không đóng.
4. Nút Close bên trong box gọi `onClose`.

---

## Code mẫu

**File: `src/components/Modal.tsx`**

```tsx
import { ReactNode } from 'react'

interface ModalProps {
  open: boolean
  onClose: () => void
  children: ReactNode
}

export function Modal({ open, onClose, children }: ModalProps) {
  if (!open) return null

  return (
    <div
      style={{
        position: 'fixed',
        inset: 0,
        backgroundColor: 'rgba(0,0,0,0.5)',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        zIndex: 1000,
      }}
      onClick={onClose}
      role="dialog"
      aria-modal="true"
    >
      <div
        style={{
          background: 'white',
          padding: 24,
          borderRadius: 8,
          minWidth: 200,
          maxWidth: '90%',
        }}
        onClick={(e) => e.stopPropagation()}
      >
        {children}
        <button type="button" onClick={onClose} style={{ marginTop: 16 }}>
          Close
        </button>
      </div>
    </div>
  )
}
```

**File: `src/App.tsx`** — example usage:

```tsx
import { useState } from 'react'
import { Modal } from './components/Modal'

// In component:
const [open, setOpen] = useState(false)
// In JSX:
<button type="button" onClick={() => setOpen(true)}>Open modal</button>
<Modal open={open} onClose={() => setOpen(false)}>
  <p>Modal content here.</p>
</Modal>
```
