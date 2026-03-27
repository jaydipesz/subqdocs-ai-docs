# Skill: Add an SVG Icon

**When NOT to use this:** Using an MUI `<SvgIcon>` wrapper that wraps a custom path — still add the path to `Svg.tsx` and pass it as children. Using a `<img src="...">` for raster images (not SVGs).

---

## The rule

All SVG icons in the entire frontend are defined in one file:

```
subqdocs-frontend/src/components/common/svg/Svg.tsx
```

Never define an SVG inline in a component file. Never import from an icon library (MUI Icons, Heroicons, etc.) without first checking if the icon already exists in `Svg.tsx`.

---

## Step 1 — Add to Svg.tsx

Open `src/components/common/svg/Svg.tsx` and append the new component. Group it near semantically related icons:

```tsx
export const MyNewIconSVG = ({
    className = '',
    onClick,
}: {
    className?: string;
    onClick?: () => void;
}) => (
    <svg
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        fill="none"
        className={className}
        onClick={onClick}
    >
        <path d="..." fill="currentColor" />
    </svg>
);
```

From `Svg.tsx` (exact):
```tsx
export const DeleteIconSVG = ({ className = '', onClick }: { className?: string; onClick?: () => void }) => (
    <svg
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        fill="none"
        className={className}
        onClick={onClick}
    >
        <path
            d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z"
            fill="currentColor"
        />
    </svg>
);
```

---

## Step 2 — Import in the consuming component

```tsx
import { MyNewIconSVG } from '../../../components/common/svg/Svg';

<MyNewIconSVG className="w-5 h-5 text-gray-500" onClick={handleClick} />
```

---

## Naming convention

| Pattern | Examples |
|---|---|
| `PascalCaseIconSVG` or `PascalCaseSVG` | `DeleteIconSVG`, `CameraIconSVG`, `ChevronDownSVG` |
| Feature-grouped | `EmaIconSVG`, `EfaxIconSVG`, `CalendarIconSVG` |

Never name it `Icon` or `SVG` alone — always include the descriptive noun.

---

## Props to always include

- `className?: string` — required for Tailwind sizing (`w-5 h-5`) and color (`text-gray-500`)
- `onClick?: () => void` — if the icon is ever used as a button

Only add `fill`, `stroke`, `width`, `height` as separate props if the icon needs them for design reasons (e.g., two-tone icons, icons with variable stroke width).

---

## fill="currentColor"

Use `fill="currentColor"` on paths rather than hardcoding a hex color. This lets the parent control color via Tailwind's `text-*` class:

```tsx
<MyNewIconSVG className="text-red-500" />   // icon becomes red
```

---

## Checklist
- [ ] Icon added to `src/components/common/svg/Svg.tsx` only
- [ ] Named `PascalCaseIconSVG` or `PascalCaseSVG`
- [ ] Accepts `className` and `onClick` props
- [ ] Paths use `fill="currentColor"` (not hardcoded hex)
- [ ] Imported from `Svg.tsx` in the consuming component
