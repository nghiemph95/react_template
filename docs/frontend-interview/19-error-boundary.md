# Bài 19: Error Boundary

## Đề bài

> "Khi một component con throw lỗi (render hoặc lifecycle), cả cây React có thể unmount và màn hình trắng. Làm sao để **bắt lỗi** và hiển thị fallback UI (vd thông báo 'Something went wrong' + nút Try again) thay vì crash? Implement một **Error Boundary** — bọc một phần cây component, khi con bên trong lỗi thì boundary bắt và render fallback."

*(Kiểm tra: hiểu Error Boundary, class component với componentDidCatch / getDerivedStateFromError. Lưu ý: không có hook tương đương, phải dùng class.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/ErrorBoundary.tsx` — **class component** kế thừa `React.Component`; implement `static getDerivedStateFromError(error)` hoặc `componentDidCatch(error, errorInfo)`; state `hasError` (và optional `error`); nếu hasError render fallback (prop `fallback` hoặc default UI), không thì render `children`.
  - **Tạo mới:** `src/components/BuggyChild.tsx` — component cố ý throw error khi state (vd nút "Throw" set state và render throw new Error()) để test.
  - **Chỉnh:** `src/App.tsx` — bọc một phần (vd BuggyChild) bằng `<ErrorBoundary fallback={...}>...</ErrorBoundary>`.

- **Có cần tạo thêm file không**
  - Không. Có thể dùng thư viện `react-error-boundary` (function component wrapper) — khi phỏng vấn có thể nói "Production em hay dùng react-error-boundary, nhưng em hiểu bên dưới là class component với componentDidCatch."

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/ErrorBoundary.tsx` | Class component; state { hasError, error }; getDerivedStateFromError return { hasError: true, error }; render: if hasError return fallback (prop hoặc default), else return children. |
| `src/components/BuggyChild.tsx` | useState; button "Throw error" set state; khi state true thì throw new Error() trong render — để test boundary. |
| `src/App.tsx` | ErrorBoundary bọc BuggyChild; fallback có nút "Try again" (reset bằng key hoặc callback). |

---

## Cách xử lý

1. **Error Boundary phải là class component** — React chưa có hook cho componentDidCatch/getDerivedStateFromError.
2. **getDerivedStateFromError(error):** static method, nhận error, return state update (vd `{ hasError: true, error }`) để trigger re-render và hiển thị fallback.
3. **componentDidCatch(error, errorInfo):** dùng để log (vd gửi lên Sentry). Optional.
4. **Render:** `if (this.state.hasError) return this.props.fallback ?? <div>Something went wrong</div>`; else `return this.props.children`.
5. **Try again:** Có thể nhận prop `onReset` và gọi khi user bấm; hoặc parent đổi `key` của ErrorBoundary để remount và reset state.

---

## Code mẫu

### File: `src/components/ErrorBoundary.tsx`

```tsx
import { Component, type ReactNode } from 'react'

interface Props {
  children: ReactNode
  fallback?: ReactNode
  onReset?: () => void
}

interface State {
  hasError: boolean
  error: Error | null
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false, error: null }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback ?? (
          <div style={{ padding: 24, border: '1px solid #ef4444', borderRadius: 8 }}>
            <h2>Something went wrong</h2>
            <p>{this.state.error?.message}</p>
            {this.props.onReset && (
              <button type="button" onClick={this.props.onReset}>
                Try again
              </button>
            )}
          </div>
        )
      )
    }
    return this.props.children
  }
}
```

### File: `src/components/BuggyChild.tsx`

```tsx
import { useState } from 'react'

export function BuggyChild() {
  const [shouldThrow, setShouldThrow] = useState(false)

  if (shouldThrow) throw new Error('Intentional error for testing')

  return (
    <div>
      <p>This component can throw an error.</p>
      <button type="button" onClick={() => setShouldThrow(true)}>
        Throw error
      </button>
    </div>
  )
}
```

### File: `src/App.tsx` (excerpt)

```tsx
import { useState } from 'react'
import { ErrorBoundary } from './components/ErrorBoundary'
import { BuggyChild } from './components/BuggyChild'

export default function App() {
  const [key, setKey] = useState(0)

  return (
    <div>
      <ErrorBoundary key={key} onReset={() => setKey((k) => k + 1)}>
        <BuggyChild />
      </ErrorBoundary>
    </div>
  )
}
```

**Ghi chú:** Error Boundary không bắt lỗi trong event handler, async code, hoặc server-side render — chỉ lỗi trong render và lifecycle của cây con.
