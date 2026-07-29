# HVL Quarto Theme — presentations and reports

[Quarto](https://quarto.org) themes for Høgskulen på Vestlandet (HVL) / Western
Norway University of Applied Sciences, based on the HVL visual identity guide
(Profilmanual 2017).

The repository ships three extensions:

| Extension | Provides |
|---|---|
| `hvl-brand` | HVL colours, fonts and logos as a [`brand.yml`](https://quarto.org/docs/authoring/brand.html) — shared by all formats |
| `hvl` | `hvl-revealjs` presentation format |
| `hvl-report` | `hvl-typst` PDF report format |

Requires Quarto 1.6 or newer.

## Quick start

### New project

```bash
quarto use template julienvollering/quarto-hvl
```

This creates two starter documents — `presentation.qmd` and `report.qmd` — and
installs all three extensions. Delete whichever you don't need.

### Add to an existing project

```bash
quarto add julienvollering/quarto-hvl
```

Then set your document format:

```yaml
format:
  hvl-revealjs: default   # presentation
```

```yaml
format:
  hvl-typst: default      # PDF report
```

Render either with `quarto render <file>.qmd`, or the Render button in RStudio
/ VS Code.

## Presentations

Open `presentation.qmd` and edit the YAML header:

```yaml
title: "Presentation Title"
subtitle: "Subtitle · Course or event name"
author: "Your Name"
institute: "Western Norway University of Applied Sciences"
date: today
```

### Title slide

The title slide background and logo are set via `title-slide-attributes`:

```yaml
title-slide-attributes:
  data-background-color: "#004357"
  data-background-image: "_extensions/julienvollering/hvl-brand/hvl_logo_en_neg.png"
  data-background-size: "180px"
  data-background-position: "top 1.5em right 1.5em"
```

### Custom classes

| Class | Purpose |
|---|---|
| `.ingress` | Lead sentence in teal DM Sans |
| `.hvl-accent` | Sea-green left-border accent bar |
| `.fact-box` | Highlighted information box |
| `.glass-box` | Semi-transparent overlay for background images |
| `.big-number` | Large teal statistic |
| `.big-label` | Small grey label beneath a big number |
| `.section-slide` | White text for teal section divider slides |
| `.dark-slide` | Teal/white text for dark background slides |
| `.no-rule` | Suppress the teal h2 border on full-bleed slides |
| `.brand-logo` | Sizes a `{{< brand logo >}}` shortcode to 55px tall |
| `.teal` `.blue` `.muted` | Colour helpers |
| `.vcenter` | Vertically centre a column |
| `.small` `.tiny` | Font-size helpers |

Fragment highlight classes (text stays visible; background animates in on click):

```markdown
[phrase]{.fragment .hl-teal}
[phrase]{.fragment .hl-yellow}
```

See `presentation.qmd` for working examples of all classes and layouts.

## Reports

`report.qmd` renders to PDF via Typst. Colours, fonts, headings, links and code
styling all come from the brand — `hvl-report` itself is deliberately almost
empty, with no page geometry, title page or headers imposed and no Typst
template to maintain.

Add a logo by naming one of the brand's logo resources:

```yaml
logo: hvl-en    # or hvl-nb, hvl-en-neg, hvl-nb-neg
```

The logo repeats on every page. Omit `logo:` for an unbranded page header.

Typst downloads DM Sans, Lora and JetBrains Mono from Google Fonts on first
render and caches them under `.quarto/typst/fonts`, so no fonts need to be
installed locally. The first render therefore needs network access.

## Using the palette elsewhere

`brand.yml` is plain YAML, so R can read the palette directly instead of
repeating hex codes — useful for keeping ggplot figures in step with the theme:

```r
brand_path <- Sys.glob(c(
  "_extensions/hvl-brand/brand.yml",     # rendering inside this repo
  "_extensions/*/hvl-brand/brand.yml"    # installed via quarto add
))[1]

hvl <- yaml::read_yaml(brand_path)$color$palette
hvl$`sea-green`   # "#00AFBA"
```

The full brand spec, including PMS and CMYK values and the official font
families, is in `HVLStyleGuideDictionary.R`.

## Bilingual use

Set `lang: en` or `lang: nb`, then pick the matching logo by hand — this is not
automatic. In presentations that means `title-slide-attributes` and the closing
slide; in reports it means the `logo:` key. English logos are `hvl-en` /
`hvl_logo_engelsk.png`, Bokmål are `hvl-nb` / `hvl_logo.png`, with `_neg`
variants for dark backgrounds.

## Fonts

The theme substitutes open-source fonts for the licensed originals:

| Role | Ships with theme | HVL profile font |
|---|---|---|
| Headings | DM Sans | NB Akademie Std |
| Body | Lora | Lyon Text |
| Code | JetBrains Mono | — |

For Word output HVL's manual specifies Arial and Georgia instead; see
`HVLStyleGuideDictionary.R`.

## Updating

```bash
quarto update julienvollering/quarto-hvl
```

This overwrites the installed extensions. Your `.qmd` files are not affected.
