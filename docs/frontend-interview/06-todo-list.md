# Bài 6: Todo List (CRUD + filter)

## Đề bài

> "Làm một Todo list: input + nút Add để thêm todo; mỗi item có checkbox (toggle done), nút Delete; filter All / Active / Completed. Dùng state để lưu list, mỗi item có id, text, done."

*(Kiểm tra: state với array of objects, add/update/delete, filter, key khi map.)*

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/components/TodoList.tsx` — state todos, filter; input + Add; list với checkbox + Delete; filter buttons.
  - **Tùy chọn:** `src/types/todo.ts` — type `Todo`, `FilterType`.
  - **Chỉnh:** `src/App.tsx` — import và render `<TodoList />`.

- **Có cần tạo thêm file không**
  - Không bắt buộc. Type có thể khai báo trong cùng file.

- **Code nằm ở đâu (map file → nội dung)**

| File | Nội dung |
|------|----------|
| `src/components/TodoList.tsx` | Interface Todo, state `todos`, `filter`; addTodo, toggleTodo(id), deleteTodo(id); filtered list; JSX form + ul + filter buttons. |
| `src/App.tsx` | `import { TodoList } from './components/TodoList'`, `<TodoList />`. |

---

## Cách xử lý

1. **State:** `todos: { id: string; text: string; done: boolean }[]`, `filter: 'all' | 'active' | 'completed'`, `inputValue: string`.
2. **Add:** trim input, nếu có thì `setTodos(prev => [...prev, { id: crypto.randomUUID() hoặc Date.now(), text, done: false }])`, clear input.
3. **Toggle:** `setTodos(prev => prev.map(t => t.id === id ? { ...t, done: !t.done } : t))`.
4. **Delete:** `setTodos(prev => prev.filter(t => t.id !== id))`.
5. **Filtered:** `todos.filter(t => filter === 'all' || (filter === 'active' && !t.done) || (filter === 'completed' && t.done))`.
6. Render list với `key={item.id}`, checkbox checked={item.done} onChange toggle, button Delete.

---

## Code mẫu

**File: `src/components/TodoList.tsx`**

```tsx
import { useState } from 'react'

interface Todo {
  id: string
  text: string
  done: boolean
}

type FilterType = 'all' | 'active' | 'completed'

export function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([])
  const [inputValue, setInputValue] = useState('')
  const [filter, setFilter] = useState<FilterType>('all')

  const addTodo = () => {
    const text = inputValue.trim()
    if (!text) return
    setTodos((prev) => [
      ...prev,
      { id: crypto.randomUUID(), text, done: false },
    ])
    setInputValue('')
  }

  const toggleTodo = (id: string) => {
    setTodos((prev) =>
      prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t))
    )
  }

  const deleteTodo = (id: string) => {
    setTodos((prev) => prev.filter((t) => t.id !== id))
  }

  const filtered =
    filter === 'all'
      ? todos
      : filter === 'active'
        ? todos.filter((t) => !t.done)
        : todos.filter((t) => t.done)

  return (
    <div>
      <div>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyDown={(e) => e.key === 'Enter' && addTodo()}
          placeholder="What to do?"
        />
        <button type="button" onClick={addTodo}>Add</button>
      </div>
      <div>
        {(['all', 'active', 'completed'] as const).map((f) => (
          <button
            key={f}
            type="button"
            onClick={() => setFilter(f)}
            style={{ fontWeight: filter === f ? 'bold' : 'normal' }}
          >
            {f.charAt(0).toUpperCase() + f.slice(1)}
          </button>
        ))}
      </div>
      <ul>
        {filtered.map((t) => (
          <li key={t.id} style={{ textDecoration: t.done ? 'line-through' : 'none' }}>
            <input
              type="checkbox"
              checked={t.done}
              onChange={() => toggleTodo(t.id)}
            />
            {t.text}
            <button type="button" onClick={() => deleteTodo(t.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

**File: `src/App.tsx`** — add: `import { TodoList } from './components/TodoList'`, in JSX: `<TodoList />`.
