# time-picker

A styled replacement for `<input type="time">` built on [shadcn/ui](https://ui.shadcn.com).

The native time input's `step` attribute only affects validation and the
spinner — the picker UI it pops up ignores it in most browsers, so you can't
actually force a 15-minute grid. This component is a two-column hour/minute
wheel popover (iOS-style), plus free-form typing that snaps to the grid on
commit. Built for a tablet-first admin UI, so it's touch-first: drag-to-scroll
wheels, a confirm button instead of auto-closing on scroll, large tap targets.

- Self-contained — the only files it needs are shadcn's own `input`,
  `button` and `popover`, pulled in automatically as registry dependencies.
- Built against **Radix** (the `-b radix` shadcn base), not Base UI or React
  Aria. If your project was initialized with a different base, the `Popover`/
  `PopoverAnchor` API won't line up — you'd need to adapt the popover usage.

## Install

```bash
npx shadcn add https://raw.githubusercontent.com/michroucka/time-picker/main/public/r/time-picker.json
```

This also installs `input`, `button` and `popover` if your project doesn't
have them yet, and adds `class-variance-authority` / `lucide-react` if
missing.

## Usage

```tsx
import { TimePicker } from "@/components/ui/time-picker";

function Example() {
    const [value, setValue] = useState("");
    return <TimePicker value={value} onChange={setValue} />;
}
```

### Props

| Prop           | Type                                 | Default        | Notes                                                                                                         |
| -------------- | ------------------------------------ | --------------- | --------------------------------------------------------------------------------------------------------------- |
| `value`        | `string`                             | —               | `"HH:mm"`, required.                                                                                            |
| `onChange`     | `(value: string) => void`            | —               | Called with a `"HH:mm"` string, already snapped to `step` and clamped to `[min, max]`.                          |
| `step`         | `number`                             | `15`            | Minute grid granularity.                                                                                        |
| `min` / `max`  | `string`                             | `"00:00"` / `"23:45"` | `"HH:mm"`. The hour wheel is clipped to this range; the minute wheel is additionally clipped on the boundary hour(s). |
| `defaultValue` | `string`                             | `min`           | Where the wheel opens/nudges from while `value` is still empty. Doesn't set `value` or fire `onChange` by itself. |
| `allowTyping`  | `boolean`                            | `false`         | When `false` the field is `readOnly` — the wheel popover is the only way to change it. When `true`, free typing (any digits) is parsed and snapped to `step` on blur/Enter, and live-previews on the wheel as you type. |
| `icon`         | `React.ReactNode \| null`            | a clock icon    | Leading icon in the trigger. Pass `null` for none.                                                              |
| `variant`      | `"default" \| "ghost"`               | `"default"`     | `"default"` is the regular boxed Input look. `"ghost"` is compact/borderless/muted — for a small inline time label rather than a standalone field. |
| `className`    | `string`                             | —               | A one-off tweak on top of `variant` (e.g. `"text-right"`) — not meant to rebuild the whole look from scratch.   |
| `placeholder`, `disabled`, `id`, `name`, `aria-label`, `aria-invalid` | | | Passed straight through to the underlying input. |

### Behavior notes

- **Wheel vs. typing are independent.** Scrolling/tapping the wheel only
  commits when you tap the checkmark (so a user can "dial in" a value and
  back out via Escape/outside-click without committing). Typing commits on
  blur/Enter, independent of whether the wheel is open.
- **Arrow Up/Down** on the input nudge the value by `step`, immediately
  (no confirm step) — a quick desktop/keyboard shortcut, separate from the
  wheel's touch-first flow.
- **Scrolling the wheel never relies on native browser scroll.** It's driven
  entirely by pointer/wheel event handlers writing `scrollTop` directly. This
  is deliberate: if this component ends up nested inside a modal/dialog that
  does its own scroll-locking (e.g. Radix `Dialog`, which only whitelists
  its own content subtree), native scroll gestures on a portaled popover can
  get silently blocked. Driving it manually sidesteps that regardless of
  what's around it.
- **Outside-click dismissal is also handled manually** (not via the
  underlying Popover's own logic) for the same reason — nested dismissable
  layers (e.g. a Dialog's) can fight over "is this click outside me", and
  the manual version only checks the two DOM nodes this component actually
  owns.

## Development

Edit `components/ui/time-picker.tsx`, then regenerate the served registry
JSON before committing/pushing:

```bash
npm install
npm run build
```

This writes `public/r/registry.json` and `public/r/time-picker.json` — the
latter embeds the full, current file content, which is what `shadcn add`
actually fetches. The two can drift if you edit the `.tsx` and forget to
rebuild, so always rebuild before pushing.
