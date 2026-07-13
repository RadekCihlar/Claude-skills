---
name: frontend-standards
description: Use BEFORE writing any JSX/HTML/CSS — new component, page, layout, styling change, or design-system work. Anti-AI-slop catalog (forbidden tells), positive design defaults, visual-polish craft (hierarchy, spacing scale, shades, shadows — Refactoring UI distilled), interaction feel (Web Interface Guidelines distilled), stack fallbacks, and the mandatory pre-done AI-tell scan. Skip only for trivial 1-line fixes (label typo, single className tweak).
---

# Frontend Standards

Act as an opinionated senior designer with taste. Output must feel human-crafted and specific to the product — never a generic AI/SaaS template. Project CLAUDE.md design tokens override anything here.

## Process

1. Read 2-3 neighbor components first; match their style.
2. Vague brief → propose 2-3 distinct named directions (e.g. "Rams + modern brutalism", "a24 meets Linear") via the brainstorming skill before building. Aesthetic direction needed → ALSO load `frontend-design:frontend-design`. Archetype starters by product: dev tool/admin → precision & density (tight, technical, monochrome); consumer/collab → warmth (generous space, soft shadows); finance/B2B → sophistication & trust (cool tones, layered depth); analytics → data-first (chart-optimized, numbers lead).
3. Check for an existing design-system record before inventing tokens: `.interface-design/system.md`, `design-tokens.*`, theme files, project CLAUDE.md. Found → follow it. Greenfield → after first component, persist chosen direction + tokens (spacing base, accent, radius, elevation strategy) to project CLAUDE.md so later sessions don't drift — consistency beats perfection.
4. Check for existing primitives BEFORE creating any `<Button>/<Card>/<Input>/<Dialog>`: look in `components/ui/`, `src/components/`, design-system packages. shadcn/MUI/Chakra/Radix installed → use theirs.
5. Quality bar = typography + spacing + restraint > flashy effects.
6. When in doubt → ask for reference screenshot, Figma link, or component to mirror.

## Stack defaults

Read `package.json` first; match existing framework/test runner/linter. FE fallback when greenfield: React 19 + TS strict + Vite + Tailwind + Vitest + TanStack Query/Router; `cn()` + CVA. Next.js: App Router, Server Components first. Zod at boundaries. Framework currency (verify installed major first — these rot): React 19 → ref is a prop, no forwardRef; Next 15 → await `params`/`searchParams`; Tailwind v4 → `@theme` in CSS, not tailwind.config.js.

## Forbidden AI-tells

Don't default to these unprompted. Drop unless user explicitly asks or the project already uses them.

**Visual/color** — no purple→pink / indigo→cyan / violet→fuchsia gradient backgrounds or `bg-clip-text` gradient headings (solid surface tokens instead); no glassmorphism (`backdrop-blur` translucent cards) unless design system has it; no neumorphism / `shadow-2xl` glow on every card; no default `rounded-2xl/3xl` everywhere (project radius tokens); no inflated type scale (`text-7xl` H1 + `text-xs` body); no arbitrary Tailwind values (`w-[427px]`, `text-[13.5px]`) — use scale tokens.

**Decoration/icons** — no emoji (🚀✨🎉⚡🔥💡) in headings/buttons/CTAs/toasts/empty states — use the project icon set; icons carry meaning, one per action max — no ✓-list spam, no decorative icon next to every heading; no pseudo-element rainbow borders.

**Motion** — no `animate-pulse/bounce/spin` on idle elements; no fade-in on every mount; no default `hover:scale-105` / `hover:-translate-y-1`; no marquee/ticker/parallax unprompted.

**Layout clichés** — no lone centered card (except auth/error/404); no centered hero + floating mockup unless brief says "landing"; no bento grids, three-column icon+title+desc feature grids, or 3-tier pricing unless asked; no generic dashboard (uniform rounded stat tiles, every number same weight); no sticky-everything; no cookie-cutter Problem→Solution→Features→Testimonials→FAQ→CTA or Linear/Vercel/Framer clone; no metronome spacing (`space-y-4`/`gap-4` on every stack — rhythm should vary with content hierarchy: related things closer, sections farther); no reflex `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8` container boilerplate — pick widths per content (prose ~65ch, data wider).

**Content/copy** — no "Supercharge your workflow" / "Get started in seconds" / "Unlock your potential" / "Built with ❤️" / "Powered by [stack]" — domain copy or `TODO: copy`; no lorem ipsum or invented names/quotes/prices; no fake testimonials, ever; no faux stats ("10,000+ users", "99.9% uptime") without real source; no pravatar/dicebear/ui-avatars/Unsplash placeholders — initials or empty state; no fake "As featured in" logo strips; no over-explained microcopy — helper text under every field, tooltips restating the label, "Click the button below to continue" — label things once, explain only the non-obvious.

**Feature creep** — no surprise dark-mode toggle, i18n switcher, or "Beta/New" badges; toasts only for async/non-obvious outcomes.

**A11y theatre** — no `aria-label` spam on self-labeled buttons; but never zero a11y either: real labels on icon-only buttons, alt text on meaningful images, focus-visible rings preserved.

**Escape hatch:** user says "make it pop" / "landing page" / "marketing site" / "hero section" → gradients, animation, big type fair game IF they match brand. Fake content + faux stats stay banned.

## Positive defaults

- **Typography first.** Distinctive face, real scale, strong hierarchy. Wire a real typeface (`next/font` etc.) — never ship default sans. `text-balance`/`text-pretty` on headings.
- **Asymmetric, intentional layout.** Break the grid for emphasis; negative space is a feature. Pick a lane (brutalist/minimalist/editorial/retro/experimental) matched to brand.
- **Color = semantic CSS-var tokens.** One accent + neutrals; never hardcode hex; no second accent without reason.
- **Subtle, high-quality motion.** Micro-interactions on hover/press; scroll-driven only where earned; respect `prefers-reduced-motion`.
- **Modern CSS over JS** where it wins: container queries, `:has()`, scroll-driven animations.
- **Semantic HTML first.** `<button>` over `<div onClick>`; `<nav>/<main>/<article>` over div-soup; ARIA only when semantics run out.
- **Four-state async.** Every async surface: idle / loading / empty / error.
- **Images.** next/image with explicit width/height, else `<img width height loading="lazy">`. No CLS.
- **Fonts.** next/font or `font-display: swap`. No render-blocking webfont.
- **Responsive for real.** Mobile-first, thumb reach, safe-area insets; verify ≤375px and ≥1280px.
- **Keyboard nav.** Modal = focus trap + ESC; dropdown = arrows + Home/End + ESC. Test with Tab before reporting done.
- **Contrast.** Text ≥ WCAG AA 4.5:1 (3:1 large/UI). No light-gray-on-white body copy.
- **Server components default** (Next app router); `"use client"` only for state/effects/browser APIs.
- **Hydration.** No `Date.now()` / `Math.random()` / `window` in render path — guard or useEffect.
- **Forms.** react-hook-form if in deps, else native `<form>` + FormData; validation = zod/yup if present, else HTML5 attrs.
- **State scope.** Local state stays local; no global store for a single-component toggle.
- **Component API.** Compound components/children over boolean-prop explosions: `<Card><Card.Header>` not `<Card withHeader showIcon hideFooter>`.

## Craft — visual polish (Refactoring UI distilled)

What separates "clean" from "amazing". Apply while designing, not as afterthought.

- **Hierarchy by de-emphasis.** Primary element not popping → mute its competitors, don't inflate it. Weight + color beat size: 3 text colors (dark primary / grey secondary / lighter-grey tertiary), 2 weights (400–500 normal, 600–700 bold), never below 400.
- **Kill labels.** Merge into the value ("12 left in stock", not "In stock: 12"); surviving labels small + muted.
- **One primary action per view.** Solid high-contrast primary; outline/soft secondary; link-style tertiary. Destructive ≠ big red button — secondary style until the confirmation step.
- **Non-linear spacing scale.** Adjacent steps ≥25% apart (4 8 12 16 24 32 48 64 96). Start with too much whitespace, remove until right. More space between groups than within them.
- **Don't fill the width.** Size to content: prose 45–75ch, forms stay narrow on wide screens. Empty margin never hurts.
- **Type mechanics.** Line-height inverse to size — headlines ~1.1–1.2, body ≥1.5. Tighten letter-spacing on large headlines; widen on ALL-CAPS. Align mixed sizes by baseline, not center.
- **Shades, not single hex.** 8–10 greys + 5–10 shades per accent, defined up front. Text on colored background: hand-pick same-hue lighter shade — never grey or white-with-alpha. Raise saturation as lightness leaves 50% or shades wash out.
- **Depth = two shadows** (soft ambient + tight direct). Elevation tiers: subtle → buttons, medium → dropdowns, large → modals. Light comes from above. Child radius ≤ parent radius — concentric corners.
- **Borders last resort.** Try shadow, background-color shift, or more spacing first.
- **Cheap polish that reads expensive:** accent border (card top / active nav / alert side), designed empty state with CTA, custom bullets/blockquotes, slight element overlap across background transitions.

## Interaction feel (Web Interface Guidelines distilled)

- Hit targets ≥24px (44px touch); label + control share one target, no dead zones.
- Inputs: font ≥16px on mobile (blocks iOS auto-zoom); correct `type`/`inputmode`/`autocomplete`; `spellcheck={false}` on emails/codes/usernames; never block paste or zoom.
- Forms: Enter submits (Cmd/Ctrl+Enter in textarea); errors inline next to field, focus first error on submit; submit stays enabled until request in flight — then spinner + keep the label; warn before nav with unsaved input.
- Loading: spinner appears after 150–300ms delay and shows ≥300ms — no flash; skeletons mirror final layout exactly.
- Optimistic UI; reconcile with rollback/undo on failure. Destructive → confirm or undo window.
- URL is state: filters, tabs, pagination, expanded panels deep-linkable; Back restores scroll.
- Motion: only `transform`/`opacity`; never `transition: all`; cancelable by input; triggered by action, not autoplay; `transform-origin` where motion "physically" starts.
- Micro-typography: curly quotes, real `…` char (also on follow-up menu items "Rename…" and progress "Saving…"), `tabular-nums` on comparable numbers, non-breaking space in `10 MB`.
- Hover/active/focus states carry MORE contrast than rest state, never less.

## Pre-done AI-tell + safety scan (mandatory)

Before reporting any FE task done, scan the diff; every hit must be justified or removed:

- **Classes:** `gradient-to`, `backdrop-blur`, `text-7xl`, `text-6xl`, `rounded-3xl`, `rounded-2xl` (non-button), `shadow-2xl`, `animate-pulse`, `animate-bounce`, `hover:scale-1`, `bg-clip-text`, `transition-all`.
- **Arbitrary values:** `-\[\d+px\]` in className.
- **Emoji in JSX strings** (U+1F300–U+1FAFF).
- **Copy:** "Supercharge", "Unlock", "Seamlessly", "Effortlessly", "Get started in seconds", "Welcome to your".
- **Image URLs:** unsplash.com, pravatar, dicebear, ui-avatars.
- **Safety:** `dangerouslySetInnerHTML` (justify), eslint-disable on a11y rules (justify), any-typed props, hardcoded hex outside design tokens.

Report one line: `AI-tell scan: clean` or `AI-tell scan: N hits — [list], kept because [reason]`.

## Done criteria

FE task is NOT done until visually verified in browser (golden path + 1 edge case) via /run or /verify, OR the handoff explicitly states: visual verification skipped — [reason]. Typecheck/unit green ≠ feature works.
