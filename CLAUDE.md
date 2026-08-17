# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A French "Première" (11th grade, lycée) mathematics course pack, written in LaTeX. It is the sibling project to the "Seconde" course pack (`Cours 2NDE 2026-2027/Polycopié et Exercices`), started from the same house style, and shares the same overall conventions. There are two parallel deliverables sharing the same `CLT2627.sty`:

- `PolycopiePremiere2627.pdf` (the course handout), built from `PolycopiePremiere2627.tex` (repo root), which pulls in chapter files from `Chapitres/`.
- `LivretExercicesPremiere2627.pdf` (the companion exercise booklet), built from `LivretExercicesPremiere2627.tex` (repo root), which pulls in worksheet files from `Exercices/`.

`CartesMentales/`, `Annexes/`, and `Miscellaneous/` hold everything else (see Architecture below). All shared macros, environments, and colors live in one custom package, `CLT2627.sty` (at the root) — genuinely shared with the Seconde project (`Cours 2NDE 2026-2027/Polycopié et Exercices`), not a copy: `CLT2627.sty` here is a **symlink** to the real file living in the Seconde repo (`../../Cours 2NDE 2026-2027/Polycopié et Exercices/CLT2627.sty`). Editing it from either project edits the exact same file on disk, and a git commit of the symlink change in this repo only records the link, not the content — the content itself is version-controlled in the Seconde repo. This relies on the two repos staying siblings under `2026 2027/` at their current relative depth; if this repo is ever cloned or moved on its own without the Seconde repo alongside it at that same relative path, the symlink will dangle and compilation will fail to find `CLT2627.sty` until it's fixed (re-point the symlink, or drop in a real copy).

## Build

Compile with **LuaLaTeX** (required — the package uses `fontspec` and other LuaLaTeX-specific features), with shell-escape enabled, **from the repo root**:

```bash
lualatex -synctex=1 -interaction=nonstopmode -file-line-error -shell-escape PolycopiePremiere2627.tex
```

The exercise booklet compiles the same way, swapping the master file:

```bash
lualatex -synctex=1 -interaction=nonstopmode -file-line-error -shell-escape LivretExercicesPremiere2627.tex
```

This matches the recipe configured in `.vscode/settings.json` for the LaTeX Workshop extension (its default recipe is named `lualatex`). Run it twice (or via `latexmk`) to resolve the table of contents and cross-references. Every file in `Chapitres/`, `Exercices/`, and `CartesMentales/` should carry a `% !TEX root = ../PolycopiePremiere2627.tex` (or `../LivretExercicesPremiere2627.tex` for `Exercices/`) magic comment, so opening one directly in VS Code/LaTeX Workshop and building still compiles the whole master document with the correct working directory — never compile a chapter, worksheet, or mind-map fragment as its own root.

Other documents compile standalone, each **from its own directory**, following the same pattern as the Seconde project's `Annexes/` (e.g. a syllabus or planning doc placed in `Annexes/`, if added later).

### Compiling only one chapter while editing

`PolycopiePremiere2627.tex` has a place near the top for a block of commented-out `\includeonly{...}` lines, one per chapter file (paths prefixed with `Chapitres/`), exactly like the Seconde project. Uncomment the one(s) you're actively editing before compiling — LaTeX will skip laying out the other chapters and reuse their existing `.aux` files, which is much faster. **Always re-comment all of them before producing the final full PDF.** `LivretExercicesPremiere2627.tex` has the same mechanism for `Exercices/` files.

## Architecture

This mirrors the Seconde project's layout:

- **`Chapitres/`** — every file `\include`d by the master document, grouped into `\part`s matching the Première program (not yet populated — add `\part`s and chapters as content is written, following the Seconde project's part ordering as a model: e.g. Nombres et calculs → Géométrie → Fonctions - Analyse → Probabilités/Stats, adapted to the actual Première syllabus). Chapter files are never compiled as the "main" file — always compile through the master document, or via `\includeonly` as above, so counters and cross-references resolve correctly. `\include`/`\input` paths inside these files are relative to the **repo root** (the master's own directory), not to `Chapitres/` itself.
  - Once chapters exist, follow the Seconde project's `<Thème><N>_<Sujet>.tex` naming convention (e.g. `NombresCalcul1_Sujet.tex`), and reserve the `0`-numbered file in each theme for a mind-map / overview chapter, not a lesson.
- **`Exercices/`** — every worksheet file `\include`d by `LivretExercicesPremiere2627.tex`, flat (no per-theme subfolders, matching the Seconde project). Unlike the Seconde `Exercices/` folder (seeded from a large batch of legacy files with unconverted names), this one starts empty — new worksheets can go straight to the `<Thème><N>_<Sujet>.tex` naming convention from the start, no legacy-name transition needed.
  - Follow the Seconde project's convention of giving each worksheet its own `\chapter` number via `\setcounter{chapter}{<n-1>}` immediately before `\chapter{...}` (whose title is just the topic), displayed as "Feuille `<n>`" via the `\titleformat{\chapter}` override already carried over into `LivretExercicesPremiere2627.tex` (local to that file, never in `CLT2627.sty`, since the two documents share that file).
- **`CartesMentales/`** — every mind-map asset referenced from a `Chapitres/*_CartesMentales.tex` file: standalone-fragment `.tex` files pulled in with `\input{CartesMentales/CarteMentaleXxx.tex}`, and the rendered `.pdf` mind maps placed with `\overpic{CartesMentales/CarteMentaleXxx.pdf}`. Anything in here is live content in the compiled handout — if a mind-map file isn't referenced from a chapter, it belongs in `Miscellaneous/` instead, not here.
- **`Annexes/`** — standalone reference documents that aren't part of the handout (syllabus, teaching-progression planning doc, etc.), following the Seconde project's pattern.
- **`Miscellaneous/`** — drafts, work-in-progress, and editable non-LaTeX sources not referenced by the compiled handout.
- `CLT2627.sty` (symlinked from the Seconde repo, see above) is the single source of truth for every custom macro, environment, and color used across chapters — check here first before adding a new one, since an equivalent likely already exists. Because it's shared, any change made here (or in the Seconde repo) affects **both** projects immediately — double-check a tweak doesn't only make sense for one grade level before making it. Its major building blocks:
  - **Pedagogical boxes**, each a colored `tcolorbox` with its own counter (reset per chapter) and color: `\Dfn[nom]{...}` (définition, green), `\Prp[nom]{...}` (propriété, blue), `\Thm[nom]{...}` (théorème, bordeaux), `\Dem[nom]{...}` (démonstration, violet), `\Exemple[nom]{...}` (gris), `\ANoter[nom]{...}` (à noter, ambre).
  - **The `Algo` environment**: a two-column box for presenting an algorithm, built from two manually-placed minipages (not tcolorbox's `sidebyside`, which mis-measures height against `lstlisting`). Left column via `\AlgoScratch{...}` renders **Scratch 3 blocks** (via the `scratch3` package) — never prose pseudo-code. Right column opens with `\begin{AlgoScript}\begin{AlgoCode}...\end{AlgoCode}\end{AlgoScript}` and renders Python styled to look like Capytale's editor (blue "Script" tab, dark code background, Scratch-matched syntax colors). See usage template in `CLT2627.sty` just above the `Algo` tcolorbox definition.
  - **`\CodeCapytale{code}{url}`**: renders a Capytale access-code/QR-code card (QR generated in-LaTeX, no external image needed).
  - Exercise-difficulty markers `\ExoVert{}{}`, `\ExoOrange{}{}`, `\ExoRouge{}{}` (1/2/3 stars) — legacy, kept for compatibility but superseded by the `Exercice` environment below for anything in `Exercices/`.
  - **The `Exercice` environment** (for `Exercices/` worksheets): `\begin{Exercice}[difficulté 1-3][titre facultatif] ... \end{Exercice}` — both arguments are optional square brackets (never braces; an omitted title must not swallow the first word of the statement). Renders a dark (`sombreExercice`) bold title bar with the exercise number (counter reset per chapter, like the pedagogical boxes) and color-coded difficulty stars (green/orange/red, same coding as `\ExoVert/Orange/Rouge`) right-aligned in the title.
  - **Hand-drawn axis helpers** `\axeX[y0]{xmin}{xmax}{ticks}`, `\axeY[x0]{ymin}{ymax}{ticks}`, `\origine`, `\pointGraphique{x}{y}{Nom}{position}`, and the TikZ styles `axe`/`general`/`quadrillageNIV2`/`quadrillage55`/`epais` — for sketched function-graph figures in `Exercices/` worksheets.
  - A large custom color palette, including graded families `A1`–`H4` (darkest to lightest per hue) for general use, and specific named colors per box type.
- `.aux`, `.log`, `.fls`, `.fdb_latexmk`, `.toc`, `.xdv`, and the compiled `.pdf` are build artifacts and are (for the most part) committed alongside the source in this repo — don't treat their presence as untracked cruft, but don't hand-edit them either.

## Content conventions

These are established house-style rules, carried over from the Seconde project — follow them for any new or edited content:

- **Completions**: any content that fills in a blank meant for the student to complete by hand (a missing word, a table entry, a sign-table cell, etc.) is wrapped in `\textcolor{J1}{...}` (or `\color{J1}` inside math mode already wrapped in `$...$`), so all completions across the document look consistent. Surrounding instructions/scaffolding stay in normal black text.
- **Worked equation/inequation solutions**: always open the derivation with "Soit $x$ un réel." before the `\Longleftrightarrow`/`align*` chain. Lay the example out as two side-by-side `[c]`-aligned minipages: the derivation (~0.6\linewidth) on the left, the concluding "La solution est $S = ...$" sentence (~0.35\linewidth) on the right — not stacked vertically, to keep examples from spilling onto the next page. When an `\item` immediately precedes such a block, end the item line with `\\` first or the minipage produces an overfull hbox.
- **Vocabulary**: describe the method for solving an equation/inequation as "isoler l'inconnue dans un membre", not "avoir l'inconnue d'un seul côté" (considered imprecise).
- **Scratch block identifiers**: inside `\AlgoScratch{...}`, never use an underscore (raw `_` is a fatal error there; escaped `\_` compiles but silently corrupts nearby oval/boolean macros) — use camelCase (`borneInf`, `premierePuissance`) instead, even though the parallel Python code on the right keeps `snake_case`.
