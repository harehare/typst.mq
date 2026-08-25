<h1 align="center">typst.mq</h1>

A [Typst](https://typst.app) markup parser implemented as an [mq](https://github.com/harehare/mq) module.

Parses a practical subset of Typst syntax — headings and single-line
statements such as `#figure(...)` — into structured mq values, so a Typst
document can be inspected and transformed with mq's selector language.

## Features

- Headings (`= Title`, `== Section`, ...) with level and text
- Single-line function-call statements (`#figure(...)`, `#image(...)`, ...), including:
  - Positional and named (`key: value`) arguments
  - Nested calls (`#figure(image("a.png"))`)
  - Trailing content blocks (`#figure(...)[Caption]`)
  - Arrays (`(1, 2, 3)`) and dicts (`(r: 1, g: 0, b: 0)`, `(:)`)
  - Strings with escapes, including `\u{XXXX}` Unicode escapes
  - Numbers, including units (`10pt`, `50%`) and `true`/`false`/`none`
- Paragraphs, coalesced across non-blank lines and split on blank lines
- Anything unsupported (`#let`, `#show`, `#import`, multi-line statements,
  code blocks, math mode, ...) is preserved verbatim as a `raw` node instead
  of raising an error

## Installation

Copy `typst.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp typst.mq ~/.local/mq/config/
```

### HTTP Import (no local installation needed)

If `mq` was built with the `http-import` feature, you can import directly from GitHub without any local setup. This requires the `--allow-http-import` flag, which is disabled by default:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/typst.mq" | let result = typst::typst_parse(.) | result[0]["text"]' report.typ
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/typst.mq@v0.1.0" | typst::typst_parse(.)' report.typ
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "typst" | typst::typst_parse(.)' input.typ
```

If you copied it to the mq built-in module directory:

```sh
mq -I raw 'import "typst" | typst::typst_parse(.)' input.typ
```

## API

### `typst_parse(input)`

Parses a Typst string and returns an array of structured nodes.

| Input type | Output |
|---|---|
| String | Array of nodes (see below) |

| Node | Shape |
|---|---|
| Heading | `{"type": "heading", "level": N, "text": "..."}` |
| Call | `{"type": "call", "name": "...", "args": [...], "named": {...}, "content": [...]}` |
| Paragraph | `{"type": "text", "text": "..."}` |
| Unsupported statement | `{"type": "raw", "text": "..."}` |

Values inside `args`/`named`/`content` are plain mq values (string, number, bool, `None`), or one of:

| Value | Shape |
|---|---|
| Nested call | `{"type": "call", ...}` |
| Content block | `{"type": "content", "text": "..."}` |
| Array | `{"type": "array", "items": [...]}` |
| Dict | `{"type": "dict", "entries": {...}}` |
| Length with unit | `{"type": "length", "value": N, "unit": "pt"}` |
| Bare identifier | `{"type": "ident", "name": "..."}` |

## Example

Given `report.typ`:

```typst
= Report

Intro paragraph.

#figure(image("chart.png"), caption: [Sales])[Q1 results]

== Details
More text.
```

```sh
mq -L . -I raw 'import "typst" | let result = typst::typst_parse(.) | result[0]["text"]' report.typ
# => "Report"

mq -L . -I raw 'import "typst" | let result = typst::typst_parse(.) | let calls = filter(result, fn(n): get(n, "type") == "call";) | calls[0]["named"]["caption"]["text"]' report.typ
# => "Sales"
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.8 or later.

## License

MIT
