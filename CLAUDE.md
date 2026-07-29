# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A set of **Quarto extensions** (not an R package, not a website) providing HVL-branded output in two formats, sharing one brand definition. Consumed via `quarto use template` / `quarto add julienvollering/HVL-RevealJS-template`.

There is no build step, no test suite, and no linter. The only "build" is rendering the two demo documents:

```bash
quarto render presentation.qmd   # revealjs deck -> presentation.html
quarto render report.qmd         # typst report  -> report.pdf
```

Both contain executable `{r}` chunks, so rendering needs R with `ggplot2`/`tidyverse` and `yaml`. The first Typst render also needs network access to fetch Google Fonts.

Useful while debugging:

```bash
quarto render report.qmd --to typst -M keep-typ:true   # inspect generated report.typ
quarto typst compile report.typ out.png --format png --pages 1 \
  --font-path .quarto/typst/fonts                      # visual check without a PDF viewer
```

## Architecture

Three extensions, each with a distinct job:

- **`_extensions/hvl-brand/`** — `brand.yml` plus the logo assets. The single source of truth for HVL colours, fonts and logos. Its `_extension.yml` contributes `metadata.project.brand`, which is what makes the brand apply project-wide rather than only to one format. It must be its own extension directory: a *format* extension's metadata contribution activates only with that format, and the brand has to reach both revealjs and typst.
- **`_extensions/hvl/`** — the `hvl-revealjs` format. `_extension.yml` declares defaults for every deck; `hvl-theme.scss` holds the entire visual design in Quarto's two-section SCSS format (`scss:defaults` for variables, `scss:rules` for CSS and custom classes).
- **`_extensions/hvl-report/`** — the `hvl-typst` format. Deliberately near-empty: brand supplies everything visual, and there is no `.typ` template. Anything added here (page geometry, title page, running headers) means taking on Typst code, which was explicitly out of scope.

`presentation.qmd` and `report.qmd` are both the user-facing starters *and* the living demos, named for the artifact each produces. `presentation.qmd` exercises every custom class and layout; when adding a class to the SCSS, add a demonstrating slide, a README table row, and an entry in the `<!-- CHEATSHEET -->` block at the bottom of the file.

Note that **neither is named `template.qmd`, and that is deliberate.** `template.qmd` is a magic filename: `quarto use template` renames it to match the directory the user chose, which in a two-starter repo would rename one file and leave the other, producing exactly the asymmetry the current names avoid. The cost is that `quarto use template` no longer renames anything — users get `presentation.qmd` and `report.qmd` verbatim, which is the intent.

Backgrounds are deliberately **not** styled in SCSS. RevealJS full-bleed backgrounds must come from Quarto attributes (`{background-color=...}`, `title-slide-attributes:`); the `.section-slide` / `.dark-slide` classes only restyle the *text* on top. Keep that split.

## Gotchas

- **The palette is duplicated in `brand.yml` and `hvl-theme.scss`, by necessity.** It looks like the SCSS should alias brand's `$brand-<name>` variables, and that was tried. Those variables are only defined for SCSS layers emitted *after* brand, which requires brand to outrank the theme in `theme:` — and then brand also wins on shared variables and silently replaces the theme's font stacks. So `theme: [brand, hvl-theme.scss]` stays (brand lowest priority) and both files carry the hex codes. **Change one, change the other.** Font families are duplicated for the same reason: the SCSS `@import` loads them for revealjs, `brand.yml` loads them for typst.
- **Brand typography leaks into the deck.** `typography.base` reaches revealjs as well as typst. Setting `line-height` there loosened every slide from 1.3 to 1.5; it was removed for that reason. Before adding anything under `typography.base`, diff the compiled deck CSS (`presentation_files/libs/revealjs/dist/theme/quarto-*.css`) against a pre-change render — a `git worktree` at the previous commit is the reliable way to get a baseline.
- **`headings.family` is `DM Sans 9pt`, not `DM Sans`.** DM Sans is a variable font with an optical-size axis and Google serves static instances whose internal family name carries the size. Typst matches that internal name and warns `unknown font family: dm sans` otherwise ([quarto-cli#11947](https://github.com/quarto-dev/quarto-cli/issues/11947)). Check `.quarto/typst/available-fonts.json` for the names Typst actually sees. The deck is unaffected — CSS resolves the variable font as plain `DM Sans`.
- **Typst font cache lives at `.quarto/typst/fonts/`**, not the `.quarto/typst-font-cache` path the Quarto docs name. `quarto typst fonts` does *not* look there, so it will report the brand fonts as missing even when renders succeed — don't trust it as a diagnostic.
- **`brand.yml` `typography.fonts` must be an array**, not a mapping keyed by font name, whatever the reference docs show. A mapping fails brand validation outright.
- **Extension paths differ between this repo and an install.** In-repo assets are at `_extensions/hvl-brand/`; after a GitHub install they land at `_extensions/julienvollering/hvl-brand/` (the org is part of the path), and after a local-path install at `_extensions/hvl-brand/`. Prefer the `{{< brand logo <name> >}}` shortcode over a written path — it resolves to wherever the brand actually lives, in every case. Wrap it in `::: {.brand-logo}` to size it; the shortcode emits a bare `<img>` and the sources are ~2000px wide. `report.qmd` uses named resources (`logo: hvl-en`) for the same reason, and its R chunk globs both locations. **The one place this can't be done is `title-slide-attributes.data-background-image`** — YAML values aren't shortcode-expanded, so that path stays hardcoded to the GitHub-installed location and silently shows no background when rendered from a clone.
- **Bilingual is a manual swap.** `lang: en` vs `lang: nb` does not change the logo. Decks need it changed in `title-slide-attributes` and on the closing slide; reports need the `logo:` key. Commented alternatives sit next to each occurrence — keep that pattern.
- **`HVLStyleGuideDictionary.R`** is an untracked reference file with the full brand spec (PMS/CMYK/RGB/hex, official font families and their usage rules, including the Arial/Georgia *office* fonts prescribed for MS Office output). Commit `5ed57ef` deliberately removed it from the shipped extension — treat it as a lookup table, not code to distribute.
- **docx is not supported by brand.yml** (only `html`, `dashboard`, `revealjs`, `typst`). Word output would need a hand-built `reference-doc:`, which shares nothing with the brand file.
