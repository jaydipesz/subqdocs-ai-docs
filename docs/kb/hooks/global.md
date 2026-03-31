# Global frontend hooks (`src/hooks/`)

One-line inventory for hooks shared across domains. Prefer documenting **domain-specific** hooks in [`../subqdocs-frontend/domains/<name>.md`](../subqdocs-frontend/domains/).

| File | Purpose |
|------|---------|
| `useDebounce.ts` | Debounced value for search/input |
| `useInquireInfoChatbotRedux.ts` | Chatbot inquire flow + Redux bridge |
| `useMicrophone.ts` | Microphone access / state for recording |
| `useOutsideClick.ts` | Detect clicks outside a ref (dropdowns, popovers) |
| `useSquareCard.ts` | Square card / payment UI integration |

Tests live under `src/hooks/__tests__/` where present.
