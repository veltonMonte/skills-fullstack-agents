# Frontend Skill
UI work only. Extends `lowTokensSkill.md`. Goal: Apple-level polish, minimum tokens.
For new visual systems, landing pages from scratch, external inspiration, or full anti-vibe-coding audit: also read `docs/skills/references/frontend-vibe-ban-and-inspiration.md`.

## 1. Stack (package.json versions win; never assume APIs from memory)
React 19 (`ref` as prop, no `forwardRef`, no `React.FC`) | TS strict (no `any`, `import type`) | Tailwind v4 (`@theme` + CSS vars) | Base UI `@base-ui/react` (headless Dialog, Tabs, Select, Menu) | `tailwind-variants` (`tv`) | `tailwind-merge` (`twMerge`) | Lucide or Phosphor.
Files `kebab-case`. Named exports only; no default exports, no barrel files (`index.ts`). Props extend `ComponentProps<'el'>` + `VariantProps`.

## 2. Component Pattern
```tsx
export const buttonVariants = tv({
  base: ['inline-flex items-center justify-center rounded-lg border font-medium transition-colors pointer-coarse:min-h-11',
    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring',
    'data-[disabled]:pointer-events-none data-[disabled]:opacity-50'],
  variants: {
    variant: { primary: 'border-primary bg-primary text-primary-foreground hover:bg-primary-hover',
               ghost: 'border-transparent text-muted-foreground hover:text-foreground' },
    size: { sm: 'h-6 px-2 gap-1.5 text-xs [&_svg]:size-3', md: 'h-7 px-3 gap-2 text-sm [&_svg]:size-3.5' },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
})
export interface ButtonProps extends ComponentProps<'button'>, VariantProps<typeof buttonVariants> {}
export function Button({ className, variant, size, disabled, ...props }: ButtonProps) {
  return <button type="button" data-slot="button" data-disabled={disabled ? '' : undefined}
    className={twMerge(buttonVariants({ variant, size }), className)} disabled={disabled} {...props} />
}
```
Compound parts (`CardHeader`, `CardTitle`...) = same pattern, 1 function each, `data-slot="card-header"`.
Always: `twMerge(base, className)`, `data-slot`, state via `data-[state]:`, `{...props}` last, `aria-label` on icon-only buttons. Focus trap/keyboard/ARIA from Base UI, never hand-rolled.

## 3. Tokens (No hardcoded hex or arbitrary px)
- `bg`: `surface`, `surface-raised`, `primary`, `secondary`, `muted`, `destructive`
- `text`: `foreground`, `foreground-subtle`, `muted-foreground`, `primary-foreground`
- `border`: `border`, `input`, `primary`, `destructive` | `ring`: `ring`
- Missing token: ask before adding to `@theme`.

## 4. Before Coding
1. Search `components/` and theme first. Reuse or extend; never duplicate.
2. From screenshot/Figma: map every color, size, radius to existing tokens. Report only real mismatches (max 3 lines). Unclear behavior (hover, empty, mobile): ask ONE question or assume and state it in 1 line.
3. Inside an existing design system: no plan needed. No existing visual system: read `docs/skills/references/frontend-vibe-ban-and-inspiration.md` and plan in max 8 lines.

## 5. Quality Bar & Code Rules
- Visual hierarchy: 1 focal point per view; whitespace over borders; max 3 font weights; 4px spacing scale; concentric nested radii (`inner = outer - padding`).
- Typography: `text-balance` headings, `text-pretty` body, `tabular-nums` numbers, lines `<75ch`. Sentence case, active voice, exact CTA verbs ("Save changes" -> toast "Saved").
- States: every interactive has `hover`, `focus-visible`, `active`, `disabled`, `loading`. Every data view has `loading` (skeleton, never spinner), `empty` (with next action), `error` (what failed + how to fix, no apology).
- Motion: 150–250ms `ease-out`, `transform`/`opacity` only, respect `prefers-reduced-motion`.
- Responsive & A11y: mobile-first (`320px+`), no horizontal scroll, `min-h-dvh`, safe areas, `44px+` touch targets. Semantic elements, 4.5:1+ contrast, full keyboard path, labeled inputs, `alt` text, dark mode via tokens.
- Perf & Hygiene: images with `width`/`height` + `lazy`, zero CLS, no `useMemo`/`useCallback` unless measured, no `<div onClick>`, no `useEffect` for derived state, no 400+ line components, no arbitrary values (`w-[437px]`, `z-[9999]`, `!important`), no `h-screen` (use `dvh`), no `alert()`, no unrequested popups/toggles.
- Security: no `dangerouslySetInnerHTML` without `DOMPurify`; validate dynamic URLs (`new URL()`, block `javascript:`/`data:`); `rel="noopener noreferrer"` on `target="_blank"`; auth tokens in `httpOnly` cookies only.

## 6. Verify
Typecheck + lint on changed files. Scan output against banned tells and fix silently. Check states, tokens, a11y, mobile. With screenshot tool: 1 capture at `390px` and 1 at `1280px` (max 1 fix round); else state `Visual: not checked`.
Priority: Security > user's request > existing design system > taste rules > brevity.
