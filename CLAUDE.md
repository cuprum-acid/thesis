# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

LaTeX source for a Computer Science Bachelor's thesis: *"Exploring Kubernetes as a Platform for Business Logic Using Custom Resource Definitions and the Operator Pattern."* The repo contains only the thesis manuscript — the two systems it evaluates (`kube-billing` and `backend-billing`) live elsewhere; Chapters 3–5 describe and benchmark them.

## Build and lint

There is no Makefile; CI is the source of truth.

- **Build (CI):** `.github/workflows/build.yml` uses `xu-cheng/latex-action@v4` with `root_file: thesis.tex`. It runs `latexmk` under the hood, which handles the `biber` + `pdflatex` passes needed for the bibliography.
- **Local build:** `latexmk -pdf thesis.tex` (requires a full TeX Live install with `biber`, plus the packages loaded in `thesis.tex` — notably `tempora`, `biblatex`, `zed-csp`, `pdfpages`).
- **Lint:** `chktex thesis.tex` (or `chktex chapters/*.tex`). Warnings `n1`, `n35`, `n44` are globally suppressed via `.chktexrc`. The CI lint job (`.github/workflows/lint.yml`) runs with `continue-on-error: true` — failures do not block.
- **Bibliography:** entries live in `ref.bib`. Style is IEEE via `biblatex` with `backend=biber` and `autocite=inline`. Re-run the build twice (or use `latexmk`) after editing `ref.bib` so citations resolve.

## Document structure

- `thesis.tex` — preamble + `\include` orchestration only. All prose lives in `chapters/`.
- `chapters/` — one file per chapter (`abstract`, `chapter1`–`chapter6`, `appex`). Each chapter file starts with `\chapter{...}\label{chap:...}`. Add new chapters by creating a new file and adding an `\include{chapters/chapterN}` line to `thesis.tex`.
- `figs/` — image assets. `\graphicspath{{figs/}}` is set, so reference images by basename (e.g. `\includegraphics{images.png}`).
- `title.pdf` — pre-rendered title page, embedded via `\includepdf` at the start of the document. Do not regenerate it from LaTeX source — edit and re-export the original PDF.
- `ref.bib` — single BibTeX file for all citations.

## Conventions and quirks to preserve

- **Manual page counter after the abstract.** `thesis.tex` contains `\setcounter{page}{12}` immediately after `\include{chapters/abstract}` with a comment block explaining the page accounting (title page, contents, abstract, etc.). If you add or remove front matter, recount manually and update this number — the build will not warn you.
- **Encoding is T2A + UTF-8.** The preamble loads `\usepackage[T1,T2A]{fontenc}` and `\usepackage[utf8]{inputenc}` to support Cyrillic glyphs even though `babel` is set to `english`. Don't strip T2A — institutional title page / references may contain Cyrillic.
- **Custom listings languages.** The preamble defines `Go`, `bash`, and `yaml` listing styles via `\lstdefinelanguage`, with `mystyle` as the default (`\lstset{style=mystyle}`). For Go code samples use `\begin{lstlisting}[language=Go]`. Don't introduce a new listings package or override the style globally.
- **Custom helpers:** `\pic{label}` → "(Fig. X)", `\tab{label}` → "(Tab. X)", `inlinelist` (from `enumitem`) for `(1) (2) (3)`-style inline enumerations.
- **Theorem environments** are pre-defined: `theorem`, `corollary`, `lemma`, `proposition`, `definition`, `remark`, `example`.

## Working on the manuscript

- Chapter cross-references use the `chap:` label prefix (`\ref{chap:impl}`, `\ref{chap:eval}`). Stick with this prefix for new chapter labels.
- The bibliography heading is renamed to "References" via `\DefineBibliographyStrings` but printed under the title "Bibliography cited" via the `\printbibliography` call — change both if you rename it.
- `figs/` currently has a single placeholder `images.png`; chapters 3–5 reference figures that will need to be added.
