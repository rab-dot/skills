---
name: code-quality
description: Code quality with eslint 8.2 + airbnb, prettier 2.8, husky 8, lint-staged 13, commitlint 17 and steiger 0.5 FSD guard. Use when fixing lint errors, formatting, pre-commit hooks, commit messages, or FSD layer violations.
---

# Code Quality (pinned: eslint@8.2, prettier@2.8, husky@8, lint-staged@13, commitlint@17, steiger@0.5)

Offline-first. Lint gates run locally — same commands work with no network.

## 1. Commands

- `lint: eslint . --fix`
- FSD guard: `steiger src` (config `steiger.config.ts`; rule `fsd/insignificant-slice` disabled)
- Staged gate (`lint-staged`): `*.{js,jsx,ts,tsx}` → `yarn run lint` → `yarn run test` → `git add`
- Commits: conventional (`commitlint/config-conventional`), `prepare: husky install`

## 2. ESLint (airbnb + typescript + prettier, eslint 8)

`.eslintrc` is eslintrc-format (v8), not flat config. Resolution order for a red file:

1. `import` order/cycles (`eslint-plugin-import`): fix alias usage (`@/`), kill cycles.
2. `react-hooks`: exhaustive-deps — fix the dep (memoize/split), do not blanket-disable.
3. `jsx-a11y`: label controls, alt text, focusable interactive elements.
4. `@typescript-eslint`: no-explicit-any → narrow or `unknown` + guard; consistent type imports.

Disable inline only with reason comment + narrowest scope. Prettier owns formatting — never hand-format against it; run `--fix`.

## 3. Prettier 2.8 baseline

Single/double quotes, trailing commas, print width per repo config — do not reformat whole files outside your diff. Editor: format-on-save on; CI check fails on drift.

## 4. FSD via steiger (layers enforced)

```
app → pages → widgets → features → entities → shared   (+ src/auth as entities)
```

Bans: no same-layer cross-imports (`features/a` ↛ `features/b`); `entities` → `shared` only; `features` → `entities/shared` only; `shared` → `shared` only; slice imports via `index.ts` public API (`@/entities/user`, never `@/entities/user/ui/...`); logic in `model/`, not `ui/`. Aliases `@/* → src/*`.

Before finishing any structural change: `npx steiger src` green.

## 5. Commits

```
feat(docs): add synonym filter
fix(map): reset selection on unmount
```

Types: `feat|fix|refactor|test|chore|docs`. Scope = feature slice. Husky runs lint+tests on commit — a failing commit means fix code, never `--no-verify`.

## Checklist

- [ ] `eslint . --fix` clean on touched files.
- [ ] `steiger src` green after moves/new slices.
- [ ] No unrelated reformats; commit message conventional.
