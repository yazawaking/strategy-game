# AGENTS.md

## 開発方針 (Development policy)

- リアルタイムの臨場感を維持する
- 素早いクリックや細かい指操作ではなく、戦略的判断を重視する
- PCとスマートフォンの両方で快適に遊べるようにする
- 既存機能を壊さずに改良する
- ゲームロジックの変更時はテストを追加する
- 大きな仕様変更は実装前に確認を求める
- mainブランチに直接コミットしない
- APIキーなどの秘密情報をコードに含めない

## What this repo is

Single-file browser game (POCKET BLITZ, a real-time strategy game). Everything — CSS, markup, and both JS layers — lives in `index.html`. There is **no package manager, no build step, no test suite, no lint config, and no CI**. Do not invent npm/build commands.

## How to run / verify

Open `index.html` in a browser (or serve the directory). There is no other verification path; changes are validated by playing the game. A deterministic seed ("作戦コード") + same settings reproduces the same map and deployment, which makes manual regression checks repeatable.

## Architecture: one file, two layers

- `<script id="engine">` — the simulation module (`PocketBlitz`, IIFE). **DOM-free and deterministic**: no DOM access, no `Math.random` in simulation code. Fixed tick order: snapshot intentions → movement → simultaneous damage → retreat → supply → victory.
- `<script id="interface">` — canvas rendering, input, UI, localStorage. Consumes the engine via `const B = PocketBlitz`.

When editing, preserve this boundary: keep the engine pure and put DOM/rendering/input in the interface layer. Keep randomness seeded via the operation code.

## Non-obvious conventions

- Hex grid geometry is hardcoded in the engine: `COLS=13, ROWS=9`, cells addressed as `r * COLS + c`. Paths contain cell IDs, never object references.
- Save format is versioned localStorage keys (`pb_operation_v6`, `pb_records_v6`, `pb_sound_v4`). When the save format changes, bump the key version; the interface tries previous versions (`v6` → `v5` → `v4`) when restoring.
- All user-facing text is **Japanese**; UI copy (unit stats, terrain, missions) lives in constants (`T`, `U`, `GROUPS`, `PLANS`…) at the top of each script.
