# HostelViet Design Guidelines

## Brand Identity

### Logo Concept

- **Mark:** Stylized wave + bed icon fusion representing ocean travel + accommodation
- **Wordmark:** "HostelViet" in Outfit Bold, "Viet" highlighted in gold
- **Usage:** Mark alone for favicons/apps, full logo for headers

### Tagline

- Primary: "Your Gateway to Vietnamese Adventures"
- Secondary: "Stay Local, Explore More"

### Brand Voice

- **Tone:** Friendly, adventurous, trustworthy, modern
- **B2B (Owners):** Professional, efficient, data-driven
- **B2C (Travelers):** Warm, exciting, welcoming

---

## Color System

### Primary Palette

| Token         | Light Mode | Dark Mode | Usage           |
| ------------- | ---------- | --------- | --------------- |
| `primary-50`  | #E6F0F7    | #0A2A3D   | Backgrounds     |
| `primary-100` | #CCE1EF    | #0D3548   | Hover states    |
| `primary-500` | #0F4C75    | #3D8AB8   | Primary actions |
| `primary-600` | #0D3F61    | #5BA3D0   | Hover primary   |
| `primary-700` | #0A324D    | #7BBCE3   | Active states   |
| `primary-900` | #051926    | #B3DCF5   | Text on light   |

### Secondary (Gold Accent)

| Token           | Light Mode | Dark Mode | Usage            |
| --------------- | ---------- | --------- | ---------------- |
| `secondary-400` | #F5A623    | #F5A623   | Accents          |
| `secondary-500` | #D4920F    | #FFBE3D   | CTAs, highlights |
| `secondary-600` | #B37D0D    | #FFD166   | Hover accent     |

### Neutrals

| Token         | Light Mode | Dark Mode | Usage          |
| ------------- | ---------- | --------- | -------------- |
| `neutral-0`   | #FFFFFF    | #0F1419   | Surface        |
| `neutral-50`  | #F8FAFC    | #1A2027   | Background     |
| `neutral-100` | #F1F5F9    | #242B33   | Cards          |
| `neutral-200` | #E2E8F0    | #2F3840   | Borders        |
| `neutral-400` | #94A3B8    | #64748B   | Muted text     |
| `neutral-600` | #475569    | #94A3B8   | Secondary text |
| `neutral-900` | #0F172A    | #F1F5F9   | Primary text   |

### Semantic Colors

| Token     | Color   | Usage                    |
| --------- | ------- | ------------------------ |
| `success` | #10B981 | Confirmations, available |
| `warning` | #F59E0B | Alerts, low availability |
| `error`   | #EF4444 | Errors, sold out         |
| `info`    | #3B82F6 | Information, links       |

---

## Typography

### Font Stack

```css
@import url("https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700&family=Inter:wght@400;500;600&display=swap");
```

| Role     | Font   | Weight  | Fallback   |
| -------- | ------ | ------- | ---------- |
| Headings | Outfit | 600-700 | system-ui  |
| Body     | Inter  | 400-500 | sans-serif |

### Type Scale

| Token       | Size | Line Height | Usage            |
| ----------- | ---- | ----------- | ---------------- |
| `text-xs`   | 12px | 1.5         | Labels, captions |
| `text-sm`   | 14px | 1.5         | Secondary text   |
| `text-base` | 16px | 1.6         | Body text        |
| `text-lg`   | 18px | 1.5         | Lead paragraphs  |
| `text-xl`   | 20px | 1.4         | H5               |
| `text-2xl`  | 24px | 1.3         | H4               |
| `text-3xl`  | 30px | 1.2         | H3               |
| `text-4xl`  | 36px | 1.2         | H2               |
| `text-5xl`  | 48px | 1.1         | H1               |
| `text-6xl`  | 64px | 1.0         | Hero headlines   |

---

## Spacing System

**Base Unit:** 4px

| Token      | Value | Usage           |
| ---------- | ----- | --------------- |
| `space-1`  | 4px   | Tight gaps      |
| `space-2`  | 8px   | Icon gaps       |
| `space-3`  | 12px  | Compact spacing |
| `space-4`  | 16px  | Default padding |
| `space-5`  | 20px  | Medium gaps     |
| `space-6`  | 24px  | Section padding |
| `space-8`  | 32px  | Card padding    |
| `space-10` | 40px  | Large gaps      |
| `space-12` | 48px  | Section margins |
| `space-16` | 64px  | Page sections   |

### Border Radius

| Token          | Value  | Usage                  |
| -------------- | ------ | ---------------------- |
| `rounded-sm`   | 4px    | Inputs, small elements |
| `rounded-md`   | 8px    | Buttons, cards         |
| `rounded-lg`   | 12px   | Modals, large cards    |
| `rounded-xl`   | 16px   | Feature cards          |
| `rounded-full` | 9999px | Avatars, pills         |

---

## Components

### Buttons

| Variant   | Background      | Text          | Border        |
| --------- | --------------- | ------------- | ------------- |
| Primary   | `primary-500`   | white         | none          |
| Secondary | transparent     | `primary-500` | `primary-500` |
| Ghost     | transparent     | `neutral-600` | none          |
| Accent    | `secondary-500` | `neutral-900` | none          |

**Sizes:**

- `sm`: 32px height, 12px padding, text-sm
- `md`: 40px height, 16px padding, text-base
- `lg`: 48px height, 24px padding, text-lg

**States:** Hover (-10% lightness), Focus (2px ring), Disabled (50% opacity)

### Cards

- **Hostel Card:** Image top, title, location, price, rating, CTA button
- **Booking Card:** Compact horizontal, date range, guest count, total price
- **Stats Card:** Icon left, metric large, label small, trend indicator

**Styling:** `rounded-lg`, `shadow-sm`, 1px border `neutral-200`, `space-4` padding

### Form Elements

- **Input:** 44px height, `rounded-sm`, 1px border, focus ring `primary-500`
- **Select:** Same as input, chevron icon right
- **Search Bar:** Icon left, clear button right, `rounded-full` for hero search
- **Checkbox/Radio:** 20x20px, `primary-500` when checked

### Navigation

- **Header (Public):** Logo left, search center, auth/user right, sticky
- **Sidebar (Dashboard):** 256px width, collapsible to 64px, icons + labels
- **Mobile Nav:** Bottom tab bar (5 items max), hamburger for more

### Tables

- Header: `neutral-100` background, `text-sm` uppercase, `font-medium`
- Rows: Alternating `neutral-50`/white, hover `primary-50`
- Pagination: Compact, show total count

### Modals

- Overlay: `neutral-900` at 50% opacity
- Container: `rounded-lg`, max-width 480px (sm), 640px (md), 960px (lg)
- Header: Title + close button, bottom border
- Footer: Action buttons right-aligned

---

## Page Layouts

### Public Portal (Travelers)

```
+----------------------------------+
| Header (sticky)                  |
+----------------------------------+
| Hero Search Section              |
+----------------------------------+
| Content Area                     |
| - Featured hostels               |
| - Destinations                   |
| - Testimonials                   |
+----------------------------------+
| Footer                           |
+----------------------------------+
```

### Owner Dashboard

```
+--------+-------------------------+
| Sidebar| Header                  |
| (256px)+-------------------------+
|        | Page Title + Actions   |
|        +-------------------------+
|        | Content Grid           |
|        | - Stats cards          |
|        | - Data tables          |
|        | - Charts               |
+--------+-------------------------+
```

### Breakpoints

| Token | Min Width | Target        |
| ----- | --------- | ------------- |
| `sm`  | 640px     | Large phones  |
| `md`  | 768px     | Tablets       |
| `lg`  | 1024px    | Laptops       |
| `xl`  | 1280px    | Desktops      |
| `2xl` | 1536px    | Large screens |

---

## Icons & Imagery

### Icon Library

**Primary:** Lucide React (MIT license, consistent stroke width)

- Size: 20px (default), 16px (small), 24px (large)
- Stroke: 1.5px

### Image Guidelines

- **Hostel Photos:** 16:9 aspect ratio, min 800x450px, WebP format
- **Thumbnails:** 4:3 ratio, lazy loaded
- **Avatars:** 1:1 ratio, 40px default, `rounded-full`
- **Maps:** Integrate Mapbox or Google Maps, custom markers in `primary-500`

---

## Accessibility

### Color Contrast

- Normal text: 4.5:1 minimum (WCAG AA)
- Large text (18px+): 3:1 minimum
- Interactive elements: 3:1 against adjacent colors

### Focus States

- Visible 2px ring in `primary-500`
- Offset 2px from element
- Never remove outline without replacement

### Screen Readers

- Semantic HTML (nav, main, aside, article)
- ARIA labels for icon-only buttons
- Alt text for all images
- Skip-to-content link

### Motion

```css
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
    }
}
```

---

## Tailwind Config Reference

```js
// tailwind.config.js
module.exports = {
    theme: {
        extend: {
            colors: {
                primary: {
                    50: "#E6F0F7",
                    100: "#CCE1EF",
                    500: "#0F4C75",
                    600: "#0D3F61",
                    700: "#0A324D",
                    900: "#051926",
                },
                secondary: { 400: "#F5A623", 500: "#D4920F", 600: "#B37D0D" },
            },
            fontFamily: {
                heading: ["Outfit", "system-ui", "sans-serif"],
                body: ["Inter", "sans-serif"],
            },
        },
    },
};
```

---

_Last updated: 2026-02-03_
