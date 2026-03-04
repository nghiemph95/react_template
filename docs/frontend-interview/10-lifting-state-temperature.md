# Bài 10: Lifting State Up — Temperature converter

## Đề bài

> "Hai ô input: một nhập Celsius, một nhập Fahrenheit. Khi đổi giá trị ở ô nào thì ô kia tự cập nhật theo (C → F: F = C * 9/5 + 32; F → C: C = (F - 32) * 5/9). State chỉ lưu ở một nơi (vd temperature value + scale 'c' | 'f')."

*(Kiểm tra: lifting state, single source of truth, derived value.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/TemperatureConverter.tsx` — state temperature (number) và scale ('c' | 'f'); hai input controlled, value mỗi ô = convert từ temperature+scale; onChange mỗi ô: parse number, set temperature và scale tương ứng.
  - **Chỉnh:** `src/App.tsx` — render TemperatureConverter.

- **Có cần tạo thêm file không**
  - Có thể tách component con `TemperatureInput` nhận scale, value, onChange — không bắt buộc.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/TemperatureConverter.tsx` | useState temperature, scale; hàm convert C↔F; hai input: value từ temperature (convert nếu cần), onChange parse và set temperature + scale. |
| `src/App.tsx` | Import and render TemperatureConverter. |

---

## Cách xử lý

1. **State:** `temperature` (number), `scale` ('c' | 'f'). Luôn lưu "giá trị đang hiển thị ở ô được edit" — khi user đổi ô C, lưu temperature = số C và scale = 'c'; khi đổi ô F, lưu temperature = số F và scale = 'f'.
2. **Giá trị hiển thị:** Ô C = scale === 'c' ? temperature : (temperature - 32) * 5/9; ô F = scale === 'f' ? temperature : temperature * 9/5 + 32.
3. **onChange ô C:** setTemperature(Number(e.target.value)), setScale('c').
4. **onChange ô F:** setTemperature(Number(e.target.value)), setScale('f').

---

## Code mẫu

**File: `src/components/TemperatureConverter.tsx`**

```tsx
import { useState } from 'react'

function toCelsius(f: number) {
  return ((f - 32) * 5) / 9
}

function toFahrenheit(c: number) {
  return (c * 9) / 5 + 32
}

export function TemperatureConverter() {
  const [temperature, setTemperature] = useState<number>(0)
  const [scale, setScale] = useState<'c' | 'f'>('c')

  const celsius = scale === 'c' ? temperature : toCelsius(temperature)
  const fahrenheit = scale === 'f' ? temperature : toFahrenheit(temperature)

  return (
    <div>
      <div>
        <label>Celsius</label>
        <input
          type="number"
          value={celsius}
          onChange={(e) => {
            setTemperature(Number(e.target.value))
            setScale('c')
          }}
        />
      </div>
      <div>
        <label>Fahrenheit</label>
        <input
          type="number"
          value={fahrenheit}
          onChange={(e) => {
            setTemperature(Number(e.target.value))
            setScale('f')
          }}
        />
      </div>
    </div>
  )
}
```

**File: `src/App.tsx`** — add: `import { TemperatureConverter } from './components/TemperatureConverter'`, in JSX: `<TemperatureConverter />`.
