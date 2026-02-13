# Frontend Quality Checks

## Activation

Conditional: Active only when React, Vue, Svelte, Angular, or similar frontend framework is detected in dependencies (`package.json`, `import` statements) or framework-specific file extensions (`.jsx`, `.tsx`, `.vue`, `.svelte`).

---

## Check Items

### No Accessibility

- **Severity:** error
- **Detect:** Grep for `<img` without `alt`, `<button` without `aria-label` or visible text content, `<input` without associated `<label` or `aria-label`, interactive elements missing `role` attributes. Also flag missing `lang` attribute on `<html>` elements.
- **Deduction:** -10 points per violation
- **Roast (en):** "Zero accessibility. Your UI is invisible to screen readers -- literally. You built an app that a significant chunk of the population cannot use."
  - **Roast (ja):** "アクセシビリティゼロですか。スクリーンリーダーからしたらこのUI、存在してないのと同じなんですよね。人口の結構な割合を最初から切り捨ててるの、すごい割り切りですよね。"
- **Fix:** Add `alt` to every image, `aria-label` to icon-only buttons, associate labels with inputs. Run `eslint-plugin-jsx-a11y` or `axe-core` and fix everything it finds.

### Inline Styles Overuse

- **Severity:** warning
- **Detect:** Count occurrences of `style={{`, `style={`, or `:style="` within component files. Flag when more than 10 inline style declarations are found across the project.
- **Deduction:** -4 points
- **Roast (en):** "`style={{` forty-seven times. CSS modules are right there, crying in the corner, completely abandoned."
  - **Roast (ja):** "`style={{`が47回って、なんだろう、せっかくCSSフレームワーク入れてるのに完全に無視するの、やめてもらっていいですか。"
- **Fix:** Extract styles into CSS modules, styled-components, Tailwind classes, or a dedicated stylesheet. Inline styles should be reserved for truly dynamic values only.

### Prop Drilling

- **Severity:** error
- **Detect:** Trace prop names through component hierarchies. Flag when the same prop is passed through 3+ component levels without being used in intermediate components (only forwarded via `props.x` or destructured and spread).
- **Deduction:** -10 points per chain
- **Roast (en):** "This prop passes through 5 components like luggage on a connecting flight. It deserves frequent flyer miles. Use Context."
  - **Roast (ja):** "このprops、5階層もバケツリレーされてるんですよね。乗り継ぎ便の荷物かってくらいたらい回しにされてて、マイル貯まりそうですよね。"
- **Fix:** Use React Context, a state management library (Zustand, Jotai, Redux), or component composition to avoid threading props through uninvolved intermediaries.

### Missing Key Props

- **Severity:** error
- **Detect:** Grep for `.map(` followed by JSX return without a `key=` prop on the root element. In Vue, check for `v-for` without `:key`. Pattern: `\.map\(.*=>\s*(<\w+(?!.*key=))` and `v-for=.*(?!.*:key)`.
- **Deduction:** -10 points per occurrence
- **Roast (en):** "`.map()` without `key` props. React's reconciler is having a full existential crisis every re-render. Chaos."
  - **Roast (ja):** "`.map()`にkey付けないの、Reactのreconcilerが毎レンダリングごとに実存的危機に陥ってるんですよ。かわいそうだと思わないんですか。"
- **Fix:** Add a unique, stable `key` prop to every element rendered in a loop. Use item IDs, not array indices (indices cause bugs on reorder/delete).

### Massive Components

- **Severity:** error
- **Detect:** Component files (`.jsx`, `.tsx`, `.vue`, `.svelte`) exceeding 300 lines of code. Count non-blank, non-comment lines.
- **Deduction:** -10 points per file
- **Roast (en):** "This component has {n} lines. That's not a component -- that's a monolith cosplaying as React."
  - **Roast (ja):** "このコンポーネント{n}行あるんですけど、それってコンポーネントじゃなくてモノリスがReactのコスプレしてるだけですよね。"
- **Fix:** Extract subcomponents, custom hooks, and utility functions. A component should do one thing. If you can't describe it in one sentence, split it.

### useEffect Abuse

- **Severity:** warning
- **Detect:** Grep for `useEffect\(\s*\(\)\s*=>` without a dependency array (no `[]` or `[deps]` as second argument). Also flag `useEffect` that sets state derived from other state/props (should be `useMemo` or computed inline). Pattern: `useEffect\([^)]+\)\s*$` without trailing `, [`.
- **Deduction:** -4 points per occurrence
- **Roast (en):** "`useEffect` with no dependency array runs on every single render. Your component is a caffeinated hamster on a wheel -- busy doing nothing useful."
  - **Roast (ja):** "依存配列なしの`useEffect`って、毎レンダリングで走るの知ってます？ カフェイン漬けのハムスターが回し車で空回りしてるのと同じなんですよ。"
- **Fix:** Always provide a dependency array. Use `useMemo` or `useCallback` for derived values. If you need `useEffect` for synchronization, make sure deps are correct. Consider if the effect is even necessary -- the React docs have a whole page called 'You Might Not Need an Effect.'

### No Loading States

- **Severity:** warning
- **Detect:** Grep for `fetch(`, `axios.`, `useQuery`, `useSWR`, or async data calls in components. Then check if the surrounding component handles loading and error states. Flag when async data is used directly without conditional rendering for `isLoading`, `isError`, `loading`, `error`, or equivalent.
- **Deduction:** -4 points per occurrence
- **Roast (en):** "You `fetch` data and render it immediately. Users get a blank screen or a crash. Suspenseful, but not the good kind."
  - **Roast (ja):** "fetchしていきなり描画するの、ユーザーには真っ白な画面かクラッシュが表示されるんですよね。サスペンスはサスペンスでも、ホラーの方ですよね、それ。"
- **Fix:** Always handle loading, error, and empty states for async data. Use Suspense boundaries, skeleton loaders, or conditional rendering. Your users deserve to know something is happening.

### Hardcoded Strings

- **Severity:** warning
- **Detect:** Grep for UI-facing text strings hardcoded directly in JSX/templates: `<h1>`, `<p>`, `<button>`, `<label>` containing raw text instead of i18n function calls (`t()`, `intl.formatMessage`, `$t()`). Flag projects with no i18n library in dependencies that contain user-visible strings in 3+ languages or plan to internationalize.
- **Deduction:** -4 points
- **Roast (en):** "Every UI string is hardcoded in English. Want to add another language? Hope you enjoy a 500-file find-and-replace marathon."
  - **Roast (ja):** "UI文字列が全部ハードコードされてますけど、多言語対応が必要になったら500ファイルの置換マラソンが始まるんですよね。楽しそうですね。"
- **Fix:** Use an i18n framework (react-intl, i18next, vue-i18n). Extract all user-facing strings to translation files. Even if you only support one language today, the abstraction pays off.

### No Error Boundaries

- **Severity:** error
- **Detect:** In React projects, grep for `ErrorBoundary`, `componentDidCatch`, `getDerivedStateFromError`, or error boundary libraries (`react-error-boundary`). If none are found in a project with 5+ components, flag it. Vue equivalent: check for `errorCaptured` hook or `onErrorCaptured`.
- **Deduction:** -10 points
- **Roast (en):** "No error boundaries. One unhandled exception and your entire app goes white-screen-of-death. Your users see... nothing. Just the void."
  - **Roast (ja):** "Error Boundaryなしって、例外1個でアプリ全体が真っ白になるんですよ。ユーザーに見えるの、虚無だけですよね。"
- **Fix:** Wrap major UI sections with `ErrorBoundary` components. Use `react-error-boundary` for a clean API. Show a fallback UI instead of crashing the entire tree. Log errors to a monitoring service.

### Direct DOM Manipulation

- **Severity:** warning
- **Detect:** Grep for `document\.getElementById`, `document\.querySelector`, `document\.getElementsBy`, `document\.createElement`, `\.innerHTML\s*=`, `\.innerText\s*=`, `\.classList\.` inside React/Vue/Svelte component files. Exclude test files and setup scripts.
- **Deduction:** -4 points per occurrence
- **Roast (en):** "`document.querySelector` in a React component. That's like bringing a horse to a Tesla factory. The virtual DOM exists for a reason."
  - **Roast (ja):** "Reactコンポーネントで`document.querySelector`使うの、テスラの工場に馬を連れてくるようなものなんですよね。仮想DOMが泣いてますよ。"
- **Fix:** Use refs (`useRef`, `ref()`, `bind:this`) for direct element access. Use state for dynamic content. Let the framework manage the DOM -- that's the entire point of using one.

### Missing Semantic HTML

- **Severity:** info
- **Detect:** Analyze component templates for `<div>` and `<span>` density without semantic elements. Flag when a component has more than 10 nested `<div>` elements with no `<section>`, `<article>`, `<nav>`, `<main>`, `<header>`, `<footer>`, `<aside>`, or `<figure>` tags.
- **Deduction:** -1 point per component
- **Roast (en):** "Divs all the way down. Your HTML is a stack of meaningless boxes. Even screen readers gave up trying to understand the structure."
  - **Roast (ja):** "divの入れ子地獄ですよね、これ。意味のない箱が無限に積み重なってて、スクリーンリーダーも構造を理解するのを諦めてますよ。"
- **Fix:** Replace structural `<div>` elements with semantic alternatives: `<nav>` for navigation, `<main>` for primary content, `<article>` for self-contained content, `<section>` for thematic groupings, `<header>`/`<footer>` for top/bottom sections.

---

## Scoring

| Step | Detail |
|------|--------|
| Start | 100 points |
| Deductions | Apply per finding using the values above |
| Floor | Minimum score is 0 (no negatives) |
| Weight | Multiply final score by **1.0x** for category contribution |

Example: 2 missing keys (-20) + no error boundaries (-10) + 3 useEffect abuses (-12) + 1 massive component (-10) = 100 - 52 = 48. Weighted: 48 * 1.0 = 48.
