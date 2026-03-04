# Bài tập phỏng vấn Frontend (React)

**Thư mục này nằm ngoài project `react-demo`** — dùng để ôn trước khi phỏng vấn. Khi demo project `react-demo` trong buổi phỏng vấn, interviewer sẽ không thấy nội dung ở đây.

Các file trong `frontend-interview/` mô phỏng **đề bài phỏng vấn frontend thực tế**. Mỗi file gồm:

- **Đề bài** — Câu hỏi / yêu cầu như interviewer đưa ra.
- **Hướng dẫn triển khai** — Nên tạo/chỉnh file nào, có cần tạo thêm file không, code nằm ở file nào (để làm đúng ngữ cảnh khi phỏng vấn).
- **Cách xử lý** — Ý tưởng và các bước xử lý.
- **Code mẫu** — Đoạn code tham khảo (toàn bộ bằng tiếng Anh).

## Cách dùng

1. Ôn và làm bài trong project **react-demo** (trong `react_template/react-demo/`).
2. Đọc tài liệu tại đây (**react_template/docs/**) trên máy mình — không cần mở folder này khi share màn hình hoặc gửi repo demo.
3. Đọc **Đề bài** → **Hướng dẫn triển khai** → tự làm trong react-demo → so **Cách xử lý** và **Code mẫu**.

## Cấu trúc project khi làm bài (react-demo)

```
react_template/react-demo/
  src/
    main.tsx
    App.tsx        ← import và render component bài tập
    App.css
    style.css
    components/    ← tạo Counter.tsx, UserList.tsx, ... theo từng bài
    hooks/         ← custom hooks (useLocalStorage, ...)
    contexts/      ← React Context (ThemeContext, ...)
```

Sau khi tạo component mới, **import vào `App.tsx`** và render để xem kết quả.

## Danh sách bài

| File | Chủ đề |
|------|--------|
| [01-counter-useState.md](./01-counter-useState.md) | Counter với useState, nút tăng/giảm |
| [02-fetch-list-loading-error.md](./02-fetch-list-loading-error.md) | Gọi API, hiển thị list, xử lý loading & error |
| [03-form-validation.md](./03-form-validation.md) | Form có validation (required, email) |
| [04-debounced-search.md](./04-debounced-search.md) | Ô search debounce trước khi gọi API |
| [05-tabs-component.md](./05-tabs-component.md) | Component Tabs chuyển nội dung theo tab |
| [06-todo-list.md](./06-todo-list.md) | Todo list: CRUD, filter All/Active/Completed |
| [07-modal-dialog.md](./07-modal-dialog.md) | Modal: open/onClose, overlay click |
| [08-useRef-focus-previous.md](./08-useRef-focus-previous.md) | useRef: focus input, lưu previous value |
| [09-custom-hook-useLocalStorage.md](./09-custom-hook-useLocalStorage.md) | Custom hook useLocalStorage |
| [10-lifting-state-temperature.md](./10-lifting-state-temperature.md) | Lifting state: converter Celsius ↔ Fahrenheit |
| [11-useCallback-useMemo.md](./11-useCallback-useMemo.md) | useCallback, useMemo (tránh re-render / cache tính toán) |
| [12-react-context.md](./12-react-context.md) | React Context (theme, Provider, useContext) |
| [13-pagination.md](./13-pagination.md) | Pagination: Previous/Next, slice list theo trang |
| [14-accordion-collapse.md](./14-accordion-collapse.md) | Accordion: mở/đóng từng section, một mở tại một thời điểm |
| [15-star-rating.md](./15-star-rating.md) | Star rating: click + hover, controlled component |
| [16-rick-morty-character-gallery.md](./16-rick-morty-character-gallery.md) | Rick and Morty API: gallery grid, infinite scroll, detail view (live coding) |
