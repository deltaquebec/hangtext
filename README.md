# hangtext: hanging indentation for prose and verse

**Block-level hanging indentation with cumulative nesting and verse layout**

The **hangtext** package provides three environments for hanging-indent layouts: `hprose` for prose; `hverse` for verse with uniform block indentation; `hverse*` for verse with progressive per-stanza indentation. The hang is a property of the block: only the block's first line pulls back to the outer margin, and nested instances step inward cumulatively. The package has no dependencies, patches no kernel commands, and works identically on untagged engines and under `\DocumentMetadata{tagging=on}` (PDF/UA-2).

[Overleaf](https://www.overleaf.com/read/jfsdfxwkgdnc#e1f365)

## Background

My own handwriting tends to favor hanging indentation (see the bts folder in the repo for examples). This way, I can, at a glance, keep track of what information belongs where. I wanted a LaTeX implementation of this. Formerly `hind`, the name collides with the ITF/Google typeface of the same name, and its prose environment (formerly `hind`, now `hprose`) had the same flaw. Each environment reduces `\linewidth` by its hang and maintains `\@totalleftmargin`. Nested display material ends at the container's right edge on either path, and load order relative to other packages is immaterial. Consequently, this is a strictly left-to-right (sinistrodextral) functionality.

## Requirements

- an e-TeX capable engine (`\dimexpr`), which every current engine satisfies;
- nothing else: no package dependencies, no hooks, no class assumptions.

## Features

- **Three environments**: `hprose` (block-level prose hang); `hverse` (uniform verse hang under `\obeylines`); `hverse*` (progressive per-stanza indentation);
- **Cumulative nesting**: nested `hprose` and `hverse` instances accumulate their hangs; the ambient `\leftskip` is captured on entry, so the hang adds to indentation imposed by theorems, lists, or quotations rather than replacing it;
- **Block semantics**: subsequent paragraphs inside a block stay at the indented level, including paragraphs resuming after display math or lists;
- **Verse runover**: long verse lines wrap one level deeper via per-line `\hangindent`, marking runover as continuation;
- **Per-stanza progression** in `hverse*`: stanza N starts N hangs in, its lines hang one further; blank lines delimit stanzas;
- **Measure propagation**: lists, quotes, and (under tagging) every display environment opened inside a hang compute the correct width; `quote` and `quotation` keep their own right insets;
- **Width constraint** via `twidth`: register clamping between paragraphs, automatic minipage wrapping mid-paragraph;
- **Justification control** for `hprose`: `raggedright`, `raggedleft`, `centering`;
- **Direct-call forms** (`\hprose...\endhprose`) for use inside TikZ nodes and other environments otherwise hostile to `\begin`/`\end`;
- **Tagged PDF support**: no `\everypar`-based paragraph tagging is disturbed.

## Installation

```bash
git clone https://github.com/deltaquebec/hangtext.git
cp hangtext/src/hangtext.sty ~/texmf/tex/latex/hangtext/
texhash ~/texmf
```

Or place `hangtext.sty` in your project directory.

## Quick start

```latex
\documentclass{article}
\usepackage{hangtext}

\begin{document}

\begin{hprose}[hindent=2.5em, vspace=4pt]
The opening line sits flush with the outer margin; continuation
lines wrap at the indented level, and so do subsequent paragraphs
until the environment ends.

A second paragraph, fully at the indented level.
\end{hprose}

\begin{hverse*}[hindent=1.5em]
Stanza zero, flush.
Its lines hang one in.

Stanza one, one indent in.
Its lines hang two in.
\end{hverse*}

\end{document}
```

### Minimal examples

```latex
% nested prose hangs; the inner block steps further in
\begin{hprose}
Outer block.
\begin{hprose}[hindent=3em]
Inner block, three ems beyond the outer hang.
\end{hprose}
Back to the outer level.
\end{hprose}

% per cola et commata via nested hverse
\begin{hverse}
We the people of the United States,
    \begin{hverse}
        in order to form a more perfect union,
    \end{hverse}
do ordain and establish this Constitution.
\end{hverse}

% pull-quote at reduced width
\begin{hprose}[twidth=0.6\linewidth, hindent=1.5em]
A narrowed hanging block set between paragraphs.
\end{hprose}

% direct call inside a TikZ node
\node[text width=16em, align=left] {%
  \hprose[hindent=1.5em]\relax
  A hanging paragraph inside a TikZ node.%
  \endhprose
};
```

## Environments

| Environment | Layout |
|-------------|--------|
| `hprose` | block-level prose hang; first line out, block body in |
| `hverse` | verse under `\obeylines`; first line out, later lines in, runover one level deeper |
| `hverse*` | verse with progressive per-stanza indentation |

## Option reference

### `hprose`

| Option | Values | Default |
|--------|--------|---------|
| `hindent` | dimension | `2em` |
| `vspace` | dimension | `0pt` |
| `justification` | `raggedright`, `raggedleft`, `centering` | flush |
| `twidth` | dimension | unbounded |

### `hverse` and `hverse*`

| Option | Values | Default |
|--------|--------|---------|
| `hindent` | dimension | `\allttsubsequentindent` (`2em` fallback) |
| `vspace` | dimension | `0pt` |
| `twidth` | dimension | unbounded |

There is no `justification` key for verse; verse is set ragged-right automatically, since justification under `\obeylines` produces large inter-word gaps on almost every line.

## Commands and registers

| Name | Role |
|------|------|
| `\hprose...\endhprose` | direct-call form; safe in bare running text |
| `\hverse...\endhverse`, `\hverse*...\endhverse*` | direct-call forms; require an enclosing group (TikZ nodes supply one) |
| `\cumulativehang` | dimension; total hang accumulated by nesting |
| `\baseleftskip` | dimension; ambient `\leftskip` captured at outermost entry |

## Documentation

The repository includes `hangtext.tex`, a combined manual and example tour: prior work (`hanging`, `hang`, bare `\hangindent`), the manuscript-tradition background (ekthesis, *per cola et commata*), the positioning model, every environment with rendered examples, and design notes.

## Scope and limitations

- the hang is block-level by design; for per-paragraph hangs use the `hanging` package or bare `\hangindent`;
- on untagged engines, `center`, `flushleft`, `flushright`, and `verbatim` compute no measure of their own and ignore the hang; under `tagging=on` they are block environments and behave correctly;
- `\linewidth` inside the environments is genuinely narrower, by design: material sized to it (`\includegraphics[width=\linewidth]`, `tabular*`) aligns with the hanging block rather than the full column;
- the environments replace `\everypar` for their duration; packages that chain content through it (`lineno`) lose that content inside the environments;
- `hverse*` redefines `\par` to detect stanza breaks; constructs that call `\par` for other reasons can in principle miscount stanzas;
- `hverse*` does not advance `\cumulativehang`; a block nested inside it anchors from the pre-`hverse*` position, not the current stanza depth.

## Compatibility

**Engines**: pdfTeX, LuaTeX, XeTeX;
**Tagging**: tested under `\DocumentMetadata{tagging=on}` with the current tagged block implementation;
**Lists**: `enumitem` and similar list-reshaping packages work unmodified, since nothing is patched;
**Related packages**: `hanging` and `hang` address per-paragraph and labeled hangs respectively; the documentation directs users to them for those jobs.

## Contributing

The usual good practice:

1. check existing issues before opening new ones;
2. include minimal working examples for bug reports;
3. follow existing code style;
4. update the documentation for new features.

## Citation

```bibtex
@misc{hangtext2026,
  author = {Quigley, Daniel},
  title  = {hangtext: hanging indentation for prose and verse},
  year   = {2026},
  url    = {https://github.com/deltaquebec/hangtext},
  version = {3.0},
  note   = {LPPL 1.3c}
}
```

## License

**LaTeX Project Public License 1.3c.**

## Author

**Daniel Quigley**
- [dquigleydev@gmail.com](mailto:dquigleydev@gmail.com)
- [GitHub](https://github.com/deltaquebec)
- [Website](https://dquigley.dev)

## Acknowledgments

Conceptual grounding from the manuscript and typographic traditions discussed in the documentation, Codex Sinaiticus to Jerome to Harrison, and from the typographic cueing literature. Thanks to the maintainers of `hanging` and `hang` for the prior work, and to the LaTeX tagging project, whose block implementation shaped the 3.0 architecture.

## See also

- [hanging](https://ctan.org/pkg/hanging), per-paragraph hanging indentation
- [hang](https://ctan.org/pkg/hang), labeled hanging paragraphs and lists
- [memoir](https://ctan.org/pkg/memoir), comprehensive book production including verse
- [verse](https://ctan.org/pkg/verse), dedicated poetry typesetting

---

**Version**: 3.0
**Last updated**: July 2026
**Status**: Active development

---
