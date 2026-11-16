# Layout reference

Copy `templates/<layout>.md`, then change the content. Frontmatter props are YAML; slots use Slidev's `::name::` markers.

| Layout | Use for | Frontmatter | Slots |
|---|---|---|---|
| `cover` | Deck title (welcome, recap) | `day`, `subtitle`, `date`, `instructor` | default: `#` title |
| `section` | First slide of every module (ink) | `kicker`, `subtitle`, `day`, `time` | default: `#` title |
| `default` | Headline + up to 6 bullets | `kicker` | default |
| `agenda` | Schedules | `kicker`, `days: [{label, items: [{time, title, hot?}]}]` (1 or 2 days) | default: headline |
| `timeline` | History, release sequences (≤ 12 items) | `kicker`, `items: [{when, label, hot?}]` | default, `caption` |
| `compare` | Before/after, do/don't (right column is the recommended one) | `kicker` | default, `left`, `right`, `bottom` |
| `diagram` | Mermaid flowcharts or a redrawn SVG | `kicker` | default (headline + diagram), `caption` |
| `code-split` | Code left, numbered notes right | `kicker` | default, `left` (one code block ≤ 10 lines), `right` (ordered list) |
| `lab` | First slide of a lab (terracotta) | `lab`, `minutes`, `handout`, `tag`, `done` | default: headline + ordered steps |
| `quote` | One pull-quote | `author`, `cite` | default: quote text, `source` |
| `image` | One image with explanation | `kicker`, `image: /file.png`, `alt`, `source`, `portrait: true` (150px round photo; `source` shown without the "Source:" prefix; optional `links: [{label, url}]` rendered as clickable links under it) | default: headline + ≤ 3 short paragraphs |
| `stat` | One to three big numbers | `kicker`, `stats: [{value, label, tone?: ink}]` | default: headline, `caption` (put `<Source>` here) |
| `table` | Comparisons with ≤ 7 rows | `kicker` | default: headline + Markdown table, `caption` |

`kicker` is the module label, e.g. `03 · Claude Code Fundamentals`.

Components you can use inside any slot:
- `<Source>text</Source>`: caption line for images and data.
- `<Verify note="NN §x" />`: ⚠ marker for a fast-changing fact.
- `<Chip>text</Chip>`: small pill.
- `<Stat value="48%" label="…" />`: a big number.
- `<Timeline :items="[…]" />`: the timeline.

In Mermaid, highlight a node with `classDef ink fill:#1D1B18,color:#F5F1EA,stroke:#1D1B18` and `class <id> ink`.
