# Bài 12: React Context (theme / user)

## Đề bài

> "Dùng React Context: tạo ThemeContext (value là 'light' | 'dark'), Provider bọc App (hoặc một phần cây), state theme trong component chứa Provider. Một component con bất kỳ dùng useContext(ThemeContext) để đọc theme và hiển thị; có nút toggle theme (setState trong provider)."

*(Kiểm tra: createContext, Provider, useContext, state ở provider.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/contexts/ThemeContext.tsx` — createContext, ThemeProvider component có useState theme và value={{ theme, setTheme }} (hoặc toggleTheme); export useTheme hook.
  - **Chỉnh:** `src/main.tsx` hoặc `src/App.tsx` — bọc bằng ThemeProvider.
  - **Tạo mới:** `src/components/ThemeDisplay.tsx` — useContext(ThemeContext), hiển thị theme + nút Toggle.

- **Có cần tạo thêm file không**
  - Context + Provider trong một file; component dùng context có thể trong components/.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/contexts/ThemeContext.tsx` | createContext, default value; ThemeProvider với useState, value; useTheme return useContext. |
| `src/components/ThemeDisplay.tsx` | useTheme(), p hiển thị theme, button toggle. |
| `src/App.tsx` hoặc main.tsx | Wrap app with ThemeProvider; render ThemeDisplay. |

---

## Cách xử lý

1. **createContext:** `const ThemeContext = createContext<{ theme: 'light'|'dark'; setTheme: ... } \| null>(null)`.
2. **Provider:** Component có `const [theme, setTheme] = useState<'light'|'dark'>('light')`, return `<ThemeContext.Provider value={{ theme, setTheme }}>{children}</ThemeContext.Provider>`.
3. **Consumer:** `const ctx = useContext(ThemeContext)`; nếu !ctx throw hoặc return null; dùng ctx.theme, ctx.setTheme (hoặc toggle: setTheme(t => t === 'light' ? 'dark' : 'light')).

---

## Code mẫu

**File: `src/contexts/ThemeContext.tsx`**

```tsx
import { createContext, useContext, useState, ReactNode } from 'react'

type Theme = 'light' | 'dark'

interface ThemeContextType {
  theme: Theme
  setTheme: (t: Theme | ((prev: Theme) => Theme)) => void
}

const ThemeContext = createContext<ThemeContextType | null>(null)

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light')
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}

export function useTheme() {
  const ctx = useContext(ThemeContext)
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider')
  return ctx
}
```

**File: `src/components/ThemeDisplay.tsx`**

```tsx
import { useTheme } from '../contexts/ThemeContext'

export function ThemeDisplay() {
  const { theme, setTheme } = useTheme()

  return (
    <div>
      <p>Current theme: {theme}</p>
      <button
        type="button"
        onClick={() => setTheme((t) => (t === 'light' ? 'dark' : 'light'))}
      >
        Toggle theme
      </button>
    </div>
  )
}
```

**File: `src/App.tsx`** — wrap with ThemeProvider (in main.tsx or App):

```tsx
import { ThemeProvider } from './contexts/ThemeContext'
import { ThemeDisplay } from './components/ThemeDisplay'

// In main.tsx or App root:
<ThemeProvider>
  <ThemeDisplay />
  {/* rest of app */}
</ThemeProvider>
```
