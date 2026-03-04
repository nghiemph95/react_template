# Bài 3: Form với validation

## Đề bài

> "Làm form đăng ký: **Email** (bắt buộc, đúng format email), **Password** (bắt buộc, tối thiểu 6 ký tự). Khi submit nếu lỗi thì hiển thị lỗi bên dưới từng field, không submit. Khi hợp lệ thì alert hoặc log ra object { email, password }."

*(Kiểm tra: controlled input, validation, hiển thị error theo field.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/SignupForm.tsx` — form (input email, password), state, hàm validate, submit handler, hiển thị lỗi từng field.
  - **Tùy chọn:** `src/utils/validation.ts` hoặc `src/lib/validateSignup.ts` — hàm `validate(email, password)` trả về object lỗi (tách ra nếu muốn dùng lại hoặc test riêng).
  - **Chỉnh:** `src/App.tsx` — import và render `<SignupForm />`.

- **Có cần tạo thêm file không**
  - **Bắt buộc:** Không. Có thể để `validate` và type `FormErrors` ngay trong `SignupForm.tsx`.
  - **Nên thêm (tùy):** File `validation.ts` / `validateSignup.ts` nếu interviewer hỏi "validation để đâu?" — trả lời: "Em có thể tách hàm validate ra file riêng để unit test và tái sử dụng."

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/SignupForm.tsx` | Type `FormErrors`, hàm `validate(email, password)`, state `email` / `password` / `errors`, `handleSubmit`, JSX form với từng input + `{errors.email}` / `{errors.password}`. Toàn bộ code mẫu form nằm ở đây. |
| `src/utils/validation.ts` (tùy chọn) | `export function validateSignup(email: string, password: string): FormErrors { ... }` và `export type FormErrors = { email?: string; password?: string }`. |
| `src/App.tsx` | `import { SignupForm } from './components/SignupForm'` và `<SignupForm />`. |

**Gợi ý khi phỏng vấn:** Có thể nói "Em validate on submit; nếu cần em có thể thêm validate onBlur hoặc real-time khi user rời field."

---

## Cách xử lý

1. **State:** `email`, `password` (string); `errors` (object hoặc từng state `emailError`, `passwordError`).
2. **Validation:** Hàm nhận email/password, trả về object lỗi (vd `{ email?: string, password?: string }`).
   - Email: required + regex đơn giản (vd `/^[^\s@]+@[^\s@]+\.[^\s@]+$/).
   - Password: required + length >= 6.
3. **Submit handler:** `e.preventDefault()`. Gọi hàm validate, nếu có lỗi thì set errors và return. Nếu không lỗi thì alert/log và có thể clear form.
4. **Render:** Mỗi input có `value={state}`, `onChange`, và bên dưới `{errors.email && <span>...</span>}`.

---

## Code mẫu

**File: `src/components/SignupForm.tsx`**

```tsx
import { useState } from 'react'

interface FormErrors {
  email?: string
  password?: string
}

function validate(email: string, password: string): FormErrors {
  const err: FormErrors = {}
  if (!email.trim()) err.email = 'Email is required'
  else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) err.email = 'Invalid email format'
  if (!password) err.password = 'Password is required'
  else if (password.length < 6) err.password = 'Password must be at least 6 characters'
  return err
}

export function SignupForm() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [errors, setErrors] = useState<FormErrors>({})

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    const nextErrors = validate(email, password)
    setErrors(nextErrors)
    if (Object.keys(nextErrors).length > 0) return
    alert(JSON.stringify({ email, password }))
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      <div>
        <label htmlFor="pw">Password</label>
        <input
          id="pw"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      <button type="submit">Sign up</button>
    </form>
  )
}
```

**File: `src/App.tsx`** — thêm:

```tsx
import { SignupForm } from './components/SignupForm'
// in JSX: <SignupForm />
```

**Ghi chú:** Có thể validate onBlur hoặc onChange (real-time) tùy yêu cầu; trên đây validate khi submit.
