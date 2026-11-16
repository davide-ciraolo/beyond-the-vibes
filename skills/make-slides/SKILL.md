---
name: make-slides
description: Build or edit slides for the Beyond the Vibes course in slides/ (Slidev, Editorial Paper theme). Use when asked to create, generate, update or fix course slides, a module page, a deck, or a layout example.
---

# Make slides

Slides follow the course framework. Each row of a module's slide outline in `docs/framework/<file>.md` becomes one slide in `slides/pages/<file>.md` (same file name). The two decks `slides/day1.md` and `slides/day2.md` import the pages with `src:`.

## Workflow

1. Read the module's framework file: objectives, slide outline, demos, lab, open items.
2. For each outline row:
   - copy `templates/<layout>.md` for the row's layout (see `layouts.md` for props and slots);
   - write the headline as the slide's `##` heading (`#` on `cover` and `section`);
   - fill in the content from the row and the research note it cites (`assets/research/NN-*.md`).
3. Run `python tools/check_slides.py slides/pages/<file>.md` and fix every error.
4. Build and look:
   ```bash
   cd slides
   npx slidev build day1.md --out dist/day1
   npx slidev export day1.md --format png --output dist/day1-png --timeout 60000
   npx slidev export day1.md --output dist/day1.pdf --timeout 60000
   ```
   Read the PNGs of the slides you touched. List any slide whose text overflows, wraps the headline to more than two lines, or looks crowded, and fix or split it.

## Rules

1. **One idea per slide.** The headline states the takeaway, not the topic. Use at most 6 bullets per column or slot, and at most 10 lines per code block (Mermaid is exempt).
2. **Named layouts only:** `cover`, `default`, `agenda`, `section`, `timeline`, `compare`, `diagram`, `code-split`, `lab`, `quote`, `image`, `stat`, `table`. No `style=` attributes, no `<style>` blocks. If a slide needs a new look, change the theme, not the slide.
3. **Images:**
   - copy each image from `assets/images/` to `slides/public/` and reference it as `/name.png`;
   - caption it with `<Source>…</Source>`, or `source:` on the `image` layout;
   - make sure it has a row in `assets/IMAGE-SOURCES.md`;
   - redraw © Anthropic diagrams as Mermaid in the theme style.
4. **Fast-changing facts** (versions, prices, flags, survey numbers, anything marked ⚠ in the research): add `<Verify note="NN §x" />` after the claim. `note` names the research note and section from the outline's Research column.
5. **Structure:** each page file starts with a `section` slide (`cover` for welcome and recap). Each lab starts with a `lab` slide whose frontmatter names `handout: labs/handouts/lab-N.md` and `tag: lab-N-start`.
6. **Emphasis:** `*one or two words*` in a headline renders terracotta italic. Use it once per headline at most. In a `table`, a bold first cell highlights the row; keep that for the Claude Code row.
7. **Finish** with the linter, `slidev build` and the PDF export for every deck you touched, and report any slides that exceed the density limits.

## Deck headmatter

`slides/day1.md` and `slides/day2.md` start like this, then list one `src:` block per page:

```md
---
theme: ./theme
title: Beyond the Vibes · Day 1
colorSchema: light
canvasWidth: 980
fonts:
  provider: none
src: ./pages/d1-00-welcome.md
---
```

## Don't

- Don't edit `slides/gallery.md` to change an example. Edit `templates/<layout>.md`.
- Don't add speaker notes (out of scope).
- Don't load web fonts or images from the internet: the deck must work offline.
- Don't write bare file names such as CLAUDE.md or SKILL.md in slide text: `.md` is a top-level domain, so Slidev turns them into links. Put them in backticks; in a headline's `*emphasis*`, write `SKILL&#46;md`.
