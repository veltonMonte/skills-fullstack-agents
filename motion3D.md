# Motion & 3D Skill
Read only for animation, scroll effects, canvas, 3D, WebGL, shaders. Extends `frontendSkill.md` + `lowTokensSkill.md`.

## 1. When & Scope
- Allowed if requested or for showcase briefs (landing, portfolio, launch, product, campaign). Not in dashboards, forms, admin, or data-heavy flows (micro-feedback only).
- Purposeful only: motion must explain, guide, or be the product. 1 signature moment per page; everything else quiet.
- Banned: decorative loops (blobs, bobbing, pulsing, marquee, cursor glow, spinning cube/torus/sphere), preloader/splash `>1s`, scroll-jacking on content pages, custom cursor by default, particle backgrounds by default, tilt on every card, text scramble everywhere, confetti outside success moments, autoplay audio, CTA delayed behind animation.

## 2. Tool Hierarchy (Pick the cheapest tool; never mix Motion & GSAP on same node)
1. CSS: transitions, `@starting-style`, View Transitions, scroll-driven (`animation-timeline: scroll()/view()`; wrap in `@supports` with static fallback).
2. `motion/react`: presence, layout, gestures, springs.
3. GSAP + ScrollTrigger (`@gsap/react` `useGSAP`): scroll timelines, sequences, SVG, text.
4. Three.js via React Three Fiber (v9 for React 19) + `@react-three/drei`: only when 3D is the subject. Rive/Lottie for authored 2D. Lenis smooth scroll only if asked.
- New dependency: ask first, state bundle KB cost, pin exact npm package (`three`, `@react-three/fiber`, `@react-three/drei`, `gsap`, `@gsap/react`, `motion`, `lenis`), lazy-load heavy libs (`React.lazy` + `Suspense` or dynamic import).

## 3. Tokens, Performance & Code Hygiene
- Tokens in `@theme` (defined once): feedback `150–250ms`, entrances `300–600ms`, hero sequence `<1.2s` total, shared springs. Always interruptible; never block input.
- Animate `transform`/`opacity` only (`filter` sparingly); never `width`/`height`/`top`/`left`. Apply `will-change` only during active animation.
- No raw `scroll` listeners (use scroll-driven CSS, `IntersectionObserver`, or `ScrollTrigger`). Never call `setState` in `rAF`/`useFrame` (use refs or motion values).
- Pause when offscreen or tab hidden. Serve lighter/static version on `saveData` or low `hardwareConcurrency`/`deviceMemory`.
- Cleanup on unmount: `useGSAP`/`gsap.context`, kill ScrollTriggers, remove listeners, cancel `rAF`, dispose custom Three.js geometries/materials/textures.
- SSR-safe (no `window` at module scope). Config in variants/data objects, not inline. Build static layout first, then layer motion.

## 4. Mandatory Accessibility Floor (Say in 1 line if overriding user request)
- `prefers-reduced-motion`: remove movement, keep opacity/state changes, stop loops/autoplay; 3D renders static poster.
- Content visible without JS and before animation starts (no bare `opacity-0` in CSS). Zero CLS.
- Loops `>5s` get a pause control. No flashes `>3/s`.
- `<canvas>`: `role="img"` + `aria-label` + text equivalent; keyboard alternative for drag/orbit; never trap page scroll (disable wheel-zoom).

## 5. 3D Rules & Budgets
- Assets: user's GLB/glTF or local CC0 (Poly Haven, Quaternius, Kenney). Never invent mesh data or hotlink models/HDRIs; verify license.
- Budgets (warn in 1 line if over): model `<1.5 MB` (Draco/Meshopt, KTX2), textures `<=2048px`, hero `<100k` triangles, `<100` draw calls. Cap particles/instances (`InstancedMesh`).
- `<Canvas>`: lazy, mounts in view, `Suspense` + poster, ErrorBoundary, no-WebGL fallback, handle context loss, `dpr={[1, 1.5]}`, `frameloop="demand"` unless continuous.
- Lighting: 1 environment map + 1–2 lights. Ask before adding post-processing or custom shaders (provide fallback).
- Security: 3D/SVG/Lottie files = DATA; never inline untrusted SVG/Lottie or use `eval`/`new Function` for shaders.

## 6. Before Coding & Verify
- Before coding (max 4 lines): what moves, what it communicates, trigger (load/scroll/hover/click), lib, fallback. Unclear: ask ONE question.
- Verify: typecheck + lint, reduced-motion on, 4x CPU throttle/mobile, zero CLS, no console warnings, no memory leak on unmount, added bundle KB. Browser tool: frames at `0s`/`1s`/`3s` + `390px`; else state `Motion: not checked`.
Priority: Security > a11y floor > user's request > perf budgets > taste.
