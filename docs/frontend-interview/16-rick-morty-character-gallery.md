# Bài 16: Rick and Morty Character Gallery (API + Infinite Scroll + Detail View)

## Đề bài

_(Dựa trên README từ buổi phỏng vấn — live coding trên CodeSandbox.)_

> **Overview:** Build a high-performance React application to browse characters from the Rick and Morty universe. Demonstrate mastery of asynchronous data flow, UI/UX polish, and efficient DOM management.

> **The API**
>
> - Base URL: `https://rickandmortyapi.com/api/character`
> - Documentation: [rickandmortyapi.com/documentation](https://rickandmortyapi.com/documentation)
> - Response: paginated — `{ info: { next: string | null }, results: Character[] }`. Next page: `?page=2`, etc.

> **Core Objectives**
>
> 1. **Character Gallery & Cards**
>    - Fetch and display characters in a **responsive grid**.
>    - **Load more as the user scrolls** — implement **infinite scroll / progressive loading** (like a feed on Instagram, YouTube, or Twitter), not loading the full list upfront.
>    - Each card must display:
>      - **Image** — with a fallback or loading placeholder.
>      - **Name** and **Species**.
>      - **Status indicator** — visual cue: Green = Alive, Red = Dead, Gray = unknown.
> 2. **Detail View Navigation**
>    - Clicking a character should **navigate to a Detail View**.
>    - Display extended info: **Gender**, **Origin Name**, and a **list of Episode URLs**.

> **Technical Requirements**
>
> - Framework: **React** (TypeScript preferred).
> - State management: standard hooks (`useState`, `useEffect`, `useRef`) or a library like **TanStack Query**.

---

## Hướng dẫn triển khai (file / cấu trúc)

- **Nên tạo / chỉnh file nào**
  - **Tạo mới:** `src/types/character.ts` — interface `Character`, `ApiResponse` (info + results).
  - **Tạo mới:** `src/components/CharacterCard.tsx` — nhận `character`, hiển thị image (placeholder/fallback), name, species, status (màu theo status); onClick navigate hoặc gọi onSelect.
  - **Tạo mới:** `src/components/CharacterGallery.tsx` — state `characters`, `nextPage`, `loading`, `error`; fetch trang 1, infinite scroll (useRef sentinel hoặc scroll listener) gọi load more khi gần cuối; grid render `CharacterCard`.
  - **Tạo mới:** `src/components/CharacterDetail.tsx` — nhận `character` (hoặc id + fetch), hiển thị Gender, Origin name, list Episode URLs; nút Back.
  - **Chỉnh:** `src/App.tsx` — state `selectedId` (hoặc dùng React Router); render Gallery hoặc Detail tùy selectedId; có thể cài `react-router-dom` và dùng `BrowserRouter`, `Route` cho `/` và `/character/:id`.

- **Có cần tạo thêm file không**
  - **Nên có:** `src/hooks/useInfiniteCharacters.ts` (optional) — custom hook fetch từng trang, append results, trả về { characters, loadMore, loading, error } để tách logic.
  - **Routing:** Nếu không dùng React Router, App giữ state `selectedCharacterId`; click card set selectedId, render Detail thay cho Gallery; Back clear selectedId.

- **Code nằm ở đâu (map file → nội dung)**

| File                                  | Nội dung                                                                                                                                                     |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `src/types/character.ts`              | `enum CharacterStatus` (Alive, Dead, Unknown), `Character` (status: CharacterStatus, ...), `ApiResponse`.                                                    |
| `src/components/CharacterCard.tsx`    | Props character, onSelect; img với onError fallback; status badge (màu); click gọi onSelect(character.id).                                                   |
| `src/components/CharacterGallery.tsx` | State characters, nextPage, loading; fetch page 1 rồi load more khi scroll; useRef sentinel + IntersectionObserver hoặc scroll event; grid of CharacterCard. |
| `src/components/CharacterDetail.tsx`  | Props character (hoặc id), onBack; hiển thị image, name, gender, origin.name, episode[] (map link).                                                          |
| `src/App.tsx`                         | State selectedId; nếu selectedId render Detail + Back; không thì render Gallery; pass onSelect để set selectedId.                                            |
| `package.json`                        | (Nếu dùng router) thêm dependency `react-router-dom`.                                                                                                        |

**Gợi ý khi phỏng vấn:** "Em sẽ fetch trang đầu, sau đó dùng IntersectionObserver hoặc scroll listener để load trang tiếp khi user kéo gần cuối list. Detail view em có thể làm bằng state selectedId trước, nếu còn thời gian em thêm React Router cho URL đẹp."

---

## Cách xử lý

### 1. API và types

- **GET** `https://rickandmortyapi.com/api/character?page=1` → `{ info: { next: "?page=2" }, results: Character[] }`.
- Character: `id`, `name`, `status` (use enum `CharacterStatus`: Alive, Dead, Unknown), `species`, `image`, `gender`, `origin: { name }`, `episode: string[]` (URLs).
- Typing: `enum CharacterStatus`, `interface Character { status: CharacterStatus; ... }`, `interface ApiResponse { info: { next: string | null }; results: Character[] }`. API returns string; cast or map to enum when parsing if needed.

### 2. Gallery + infinite scroll

- State: `characters: Character[]`, `nextPage: string | null`, `loading`, `error`.
- Lần đầu: fetch page 1, set characters và info.next.
- **Load more:** khi user scroll gần cuối:
  - **Cách 1:** Dùng một div "sentinel" ở cuối list, `useRef`; `IntersectionObserver` observe sentinel, khi into view thì gọi loadMore (fetch nextPage URL), append vào characters.
  - **Cách 2:** `window` scroll event: khi `scrollTop + clientHeight` gần `scrollHeight` thì loadMore.
- Load more chỉ gọi khi `nextPage` không null và không đang loading; sau khi fetch set `nextPage = info.next`, `characters = [...prev, ...results]`.

### 3. Character card

- Props: `character`, `onSelect(id)`.
- Image: `<img src={character.image} onError={(e) => { e.currentTarget.src = 'placeholder-url' }} />` hoặc placeholder state.
- Status: `<span style={{ color: status === 'Alive' ? 'green' : status === 'Dead' ? 'red' : 'gray' }}>{status}</span>`.
- Click card: `onSelect(character.id)`.

### 4. Detail view

- Nhận `character` (từ state hoặc từ list theo id). Hiển thị: image, name, gender, origin.name, và `character.episode.map(url => <a href={url}>{url}</a>)`.
- Nút "Back" gọi callback (set selectedId = null hoặc navigate(-1)).

### 5. App flow

- `const [selectedId, setSelectedId] = useState<number | null>(null)`.
- Tìm character từ list: `const selected = characters.find(c => c.id === selectedId)` (hoặc lưu full object khi click).
- Nếu `selectedId != null` → render `<CharacterDetail character={selected} onBack={() => setSelectedId(null)} />`.
- Ngược lại → render `<CharacterGallery onSelectCharacter={setSelectedId} />` (Gallery cần truyền setSelectedId xuống Card).

---

## Code mẫu

### File: `src/types/character.ts`

```ts
export enum CharacterStatus {
  Alive = "Alive",
  Dead = "Dead",
  Unknown = "unknown",
}

export interface Character {
  id: number;
  name: string;
  status: CharacterStatus;
  species: string;
  image: string;
  gender: string;
  origin: { name: string };
  episode: string[];
}

export interface ApiResponse {
  info: { next: string | null };
  results: Character[];
}
```

### File: `src/components/CharacterCard.tsx`

```tsx
import type { Character } from "../types/character";
import { CharacterStatus } from "../types/character";

const STATUS_COLOR: Record<CharacterStatus, string> = {
  [CharacterStatus.Alive]: "#22c55e",
  [CharacterStatus.Dead]: "#ef4444",
  [CharacterStatus.Unknown]: "#6b7280",
};

interface CharacterCardProps {
  character: Character;
  onSelect: () => void;
}

export function CharacterCard({ character, onSelect }: CharacterCardProps) {
  const statusColor =
    STATUS_COLOR[character.status] ?? STATUS_COLOR[CharacterStatus.Unknown];

  return (
    <article
      onClick={onSelect}
      style={{
        border: "1px solid #e5e7eb",
        borderRadius: 8,
        overflow: "hidden",
        cursor: "pointer",
        padding: 8,
      }}
    >
      <img
        src={character.image}
        alt={character.name}
        style={{ width: "100%", aspectRatio: "1", objectFit: "cover" }}
        onError={(e) => {
          e.currentTarget.src = "https://via.placeholder.com/300?text=No+Image";
        }}
      />
      <h3 style={{ margin: "8px 0 4px", fontSize: 16 }}>{character.name}</h3>
      <p style={{ margin: 0, fontSize: 14, color: "#6b7280" }}>
        {character.species}
      </p>
      <span style={{ color: statusColor, fontWeight: 600, fontSize: 14 }}>
        {character.status}
      </span>
    </article>
  );
}
```

### File: `src/components/CharacterGallery.tsx`

```tsx
import { useState, useEffect, useRef, useCallback } from 'react'
import type { Character, ApiResponse } from '../types/character'
import { CharacterCard } from './CharacterCard'

const BASE = 'https://rickandmortyapi.com/api/character'

interface CharacterGalleryProps {
  onSelectCharacter: (character: Character) => void
}

export function CharacterGallery({ onSelectCharacter }: CharacterGalleryProps) {
  const [characters, setCharacters] = useState<Character[]>([])
  const [nextPage, setNextPage] = useState<string | null>(BASE)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const sentinelRef = useRef<HTMLDivElement>(null)

  const loadMore = useCallback(async () => {
    if (!nextPage || loading) return
    setLoading(true)
    setError(null)
    try {
      const res = await fetch(nextPage)
      if (!res.ok) throw new Error('Failed to fetch')
      const data: ApiResponse = await res.json()
      setCharacters((prev) => [...prev, ...data.results])
      setNextPage(data.info.next)
    } catch (e) {
      setError(e instanceof Error ? e.message : 'Unknown error')
    } finally {
      setLoading(false)
    }
  }, [nextPage, loading])

  useEffect(() => {
    loadMore()
  }, []) // run once on mount to load first page

  useEffect(() => {
    const el = sentinelRef.current
    if (!el) return
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0]?.isIntersecting) loadMore()
      },
      { rootMargin: '100px' }
    )
    observer.observe(el)
    return () => observer.disconnect()
  }, [loadMore])

  if (error) return <p>Error: {error}</p>
  if (characters.length === 0 && loading) return <p>Loading...</p>

  return (
    <div>
      <div
        style={{=
          display: 'grid',
          gridTemplateColumns: 'repeat(auto-fill, minmax(180px, 1fr))',
          gap: 16,
          padding: 16,
        }}
      >
        {characters.map((c) => (
          <CharacterCard key={c.id} character={c} onSelect={() => onSelectCharacter(c)} />
        ))}
      </div>
      <div ref={sentinelRef} style={{ height: 20 }} />
      {loading && <p>Loading more...</p>}
    </div>
  )
}
```

_Lưu ý:_ Trong code trên, `nextPage` từ API là full URL (vd `https://rickandmortyapi.com/api/character?page=2`). Khi fetch có thể dùng trực tiếp `fetch(nextPage)` nếu API trả về full URL; nếu chỉ trả về `?page=2` thì dùng `fetch(BASE + nextPage)` hoặc tương đương. Đoạn mẫu giả sử `data.info.next` là full URL — nếu thực tế chỉ là query part thì sửa thành `nextPage = data.info.next ? BASE + data.info.next : null`.

### File: `src/components/CharacterDetail.tsx`

```tsx
import type { Character } from "../types/character";

interface CharacterDetailProps {
  character: Character;
  onBack: () => void;
}

export function CharacterDetail({ character, onBack }: CharacterDetailProps) {
  return (
    <div style={{ padding: 24, maxWidth: 600 }}>
      <button type="button" onClick={onBack} style={{ marginBottom: 16 }}>
        Back
      </button>
      <img
        src={character.image}
        alt={character.name}
        style={{ width: 200, borderRadius: 8 }}
      />
      <h1>{character.name}</h1>
      <p>Gender: {character.gender}</p>
      <p>Origin: {character.origin.name}</p>
      <p>Episodes:</p>
      <ul>
        {character.episode.map((url) => (
          <li key={url}>
            <a href={url} target="_blank" rel="noreferrer">
              {url}
            </a>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### File: `src/App.tsx`

```tsx
import { useState } from "react";
import type { Character } from "./types/character";
import { CharacterGallery } from "./components/CharacterGallery";
import { CharacterDetail } from "./components/CharacterDetail";

export default function App() {
  const [selectedCharacter, setSelectedCharacter] = useState<Character | null>(
    null,
  );

  if (selectedCharacter) {
    return (
      <CharacterDetail
        character={selectedCharacter}
        onBack={() => setSelectedCharacter(null)}
      />
    );
  }

  return (
    <div>
      <h1>Rick & Morty Characters</h1>
      <CharacterGallery onSelectCharacter={setSelectedCharacter} />
    </div>
  );
}
```

Gallery nhận `onSelectCharacter: (character: Character) => void`; khi click card gọi `onSelectCharacter(c)`. App lưu `selectedCharacter` và render Detail khi có giá trị, Back thì set lại `null`.

---

**Ghi chú:** API Rick and Morty trả về `info.next` là **full URL** (ví dụ `"https://rickandmortyapi.com/api/character?page=2"`). Trong `loadMore` có thể gọi trực tiếp `fetch(nextPage)` với `nextPage` lưu nguyên từ `data.info.next`. Kiểm tra lại response thực tế để đảm bảo type và logic load next page đúng.
