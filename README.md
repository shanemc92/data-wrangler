# data-wrangler

Single-file, client-side tool for turning messy tabular text into something usable. Paste or drop CSV, TSV, any delimiter, fixed width columns, JSON or JSON lines, Markdown tables or key=value log lines, run them through a list of steps, and export CSV, TSV, JSON, JSON lines or Markdown. No build step, no dependencies, no network calls. Open `index.html` and it works, including from `file://`.

## Input

![screenshot-input](/docs/screenshot-input.png)

| Format | Notes |
|---|---|
| Delimited | Comma, tab, semicolon, pipe, runs of whitespace, any custom string (`::`, `\t`), or a set of characters any one of which splits (`,;\|`). Repeated delimiters can be collapsed. Quote aware, handles quoted newlines and `""` escapes |
| Fixed width | Cut positions (`6,19,25`, 0-based, the last column runs to the end of the line) or field widths counted from the left. Cells are trimmed |
| Logs with a message | A max column count keeps the rest of the line, spaces and all, in the last column, so `03-17 16:13:38.839  1702  2113 V WindowManager: Skipping ...` becomes six fields plus one message |
| JSON | Array of objects, array of arrays, a wrapper object with one array in it, or a JSON path (`data.items`). JSON lines when the text is not one document. Nested objects flatten to `a.b` |
| Markdown | Pipe tables, `\|` treated as a literal pipe |
| key=value | Firewall and syslog style. An unquoted value runs to the next `key=`, so `timestamp=2026-05-20 20:05 src=1.2.3.4` keeps the space. Quoted values, glued syslog prefixes (`<189>date=...`) and missing or reordered fields are handled; columns line up by key |
| Lines | One column per line |

Auto detect reads a line range you choose, so a file with a preamble can be pointed at the lines that matter. As well as the formats above it recognises two log shapes: fixed width columns, found from character positions that are blank on every sampled line (a boundary needs a two character gap, so word spaces in free text are not mistaken for one), and lines whose leading whitespace fields have the same shape on every line, where the rest of the line is one message. Everything it works out is written into the fields, so it can be corrected by hand. Skip first N lines and ignore comment lines are available per source.

Several sources can be appended (columns matched by name, optional `_source` column) or joined on one or more key columns (left, inner, full).

## Transform

Run top to bottom, each can be switched off, moved or removed, and removal asks first. Each step reports what it did and the size of the table it left behind. "hide" collapses a step to its header and that result line, and "Collapse all" does the lot, so a long pipeline stays readable. A failing step is skipped and says why.

Columns (keep, drop, rename, reorder), filter rows, sort (auto, text, number, date, IPv4), remove duplicates on chosen columns, key=value cells to columns, split a column, merge columns, extract with regex (named groups become column names), find and replace, clean text, keep a row range, transpose.

![screenshot-transform](/docs/screenshot-transform.png)

## Output

CSV, TSV, JSON (objects or arrays, optional typed numbers), JSON lines, Markdown. Split into files every N rows, by percentage, or one file per value of a column; each file keeps the header. Optional formula guard for spreadsheets and UTF-8 BOM for Excel.

![screenshot-output](/docs/screenshot-output.png)

## Configs

A config is parse settings, steps and output settings without the data. Save to a file or to named configs in the browser, then load it and drop the next batch of files in. Loaded configs are validated field by field before use.

## Security

Strict CSP, no remote resources, no `innerHTML` with data, prototype-safe object building, configs sanitised on load. Data and settings autosave to localStorage until Reset; inputs too large for localStorage are not saved. Regex steps run on the main thread, so a catastrophic pattern can hang the tab.

## Appearance

Three independent controls in the top bar, each saved separately, so changing
one never resets the others.

**Design** sets the shape language, surface hue and backdrop texture:

| Design | Shape | Backdrop |
|---|---|---|
| `cobalt` | Slight radius, navy | Blue glow from the top, blue edge rules. The default |
| `chamfer` | Cut corners | Instrument grid |
| `console` | Square, graphite | Horizontal scan rules |
| `contour` | Soft radii, violet | A single accent wash |

**Mode** sets the lightness ramp only, and every design supports every mode:

| Mode | Base |
|---|---|
| `dark` | Lights off. The default |
| `dusk` | Dark, lifted off black, for long sessions |
| `sepia` | Warm paper, bright but low glare |
| `light` | Cool white, closest to print |

**Accent** is any hue. Eight presets are offered, plus a hue/chroma wheel and a
hex field for matching a brand colour exactly.

That is sixteen design-and-mode combinations, on any accent.

Geometry is deliberately shared: every design uses the same bar height, panel
padding, control padding, type sizes and grid, so switching design changes
colour, radius and ornament without reflowing the page.

The accent's lightness is not taken from the picker. The mode proposes a
starting lightness, then the accent is walked away from the surface it sits on
until it clears a measured contrast ratio. HSL lightness is not perceptual, so
a fixed value would leave some hues washed out on white and others muddy on
black; measuring instead is what lets any hue, including a near-white or
near-black pick, stay readable. Status colours (`--ok`, `--warn`, `--bad`)
stay semantic and are kept clear of the accent range, so the accent never
reads as state.

Printing forces one light palette regardless of design and mode, drops the
backdrop texture and the bar controls, and keeps the bar as a masthead so the
logo and tool name land on the page.

## Deployment

`_headers` (Netlify, Cloudflare Pages) and `.htaccess` (Apache) carry the same
CSP and hardening headers. Neither is needed to run the file locally, including
from `file://`.

## Licence

MIT. See `LICENSE`.
