# ACS specification (Bikeshed source)

This directory holds the Agent Control Standard as a [Bikeshed](https://speced.github.io/bikeshed/)
document. Bikeshed is the preprocessor W3C and WHATWG specifications are written in: it takes
Markdown-flavored source and produces a cross-linked HTML specification with automatic section
numbering, definition linking, a term index, and a bibliography.

`index.bs` is the source of truth for the written specification. The JSON Schemas under
`specification/v0.1.0/` are normative alongside it.

## Build

Bikeshed needs Python 3.12 or newer.

```bash
# One-time setup
uv tool install --python 3.12 bikeshed
bikeshed update          # downloads cross-reference and bibliography data

# Build spec/index.html
make

# Check without writing output (what CI runs)
make lint

# Rebuild on save while editing
make watch
```

`make` and `make lint` both run with `--die-on=warning`. A dangling cross-reference, a
duplicate ID, or a definition nothing links to fails the build.

## Layout

| Path | Contents |
|---|---|
| `index.bs` | Metadata, bibliography entries, and the include manifest |
| `sections/*.include` | One file per top-level part of the spec, in document order |
| `copyright.include` | Copyright boilerplate (CC BY-SA 4.0) that replaces Bikeshed's default |
| `diagrams/` | ASCII diagrams pulled in with `<pre class=include-code>` |
| `index.html` | Build output, not committed |

To add a section, drop a new `.include` file in `sections/` and add a `<pre class=include>`
block to `index.bs` at the position it belongs in.

## Conventions

**Markdown, with HTML tables.** Bikeshed's Markdown covers headings, lists, code fences, and
definition lists (`: term` / `:: definition`). It does *not* support pipe tables — write those
as `<table class="data">`.

**Headings carry explicit IDs.** `Chain hashing {#chain-hashing}`. The ID is the permalink;
changing one breaks every inbound link, so treat them as stable.

**Cross-references use `[[#anchor]]`**, which renders as a live "§ 9.2 Chain hashing" link.
Never hand-write a section number — Bikeshed numbers sections automatically and the numbers
move whenever a section is added.

**Terms are defined once with `<dfn>` and linked with `[=term=]`.** Hooks are scoped to the
`hook` namespace: `<dfn for=hook>toolCallRequest</dfn>` is linked as `[=hook/toolCallRequest=]`.
Bikeshed builds the "Terms defined by this specification" index from these, and warns about
definitions nothing links to.

**External standards are cited with `[[SHORTNAME]]`.** RFCs resolve automatically from
SpecRef; everything else (JSON-RPC, MCP, A2A, OCSF, CycloneDX, SPDX, SWID, FIPS 203-205) is
declared in the `<pre class=biblio>` block at the top of `index.bs`. A `[[!SHORTNAME]]` marks
a normative reference.

**Prose follows `STYLE.md`** at the repository root, same as the rest of the documentation.

## Relationship to the MkDocs site

`docs/` still contains the Markdown pages that build the current documentation site, including
Markdown copies of the specification sections. Those pages and this Bikeshed document describe
the same v0.1.0 protocol; this directory is where the specification is now edited. Until the
site cuts over, a normative change needs to land in both places — or the `docs/spec/` and
`docs/concepts/` pages need to be replaced with links to the published spec.
