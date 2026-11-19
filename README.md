# tinycoder-web

The marketing site for the [TinyCoder inference engine](https://github.com/mgorshkov/tinycoder-inference) — a
from-scratch C++20/CUDA engine for GGUF and EXL3 checkpoints.

This repository is **only** the static site. It has no build step, no framework, no package manager and no
external requests: `index.html` and `media/` are the whole thing, and the page works offline.

## Preview locally

```bash
cd tinycoder-web
python3 -m http.server
```

Open <http://localhost:8000>.

Any static file server works. Opening `index.html` directly with `file://` also renders, but serving it is
closer to production and avoids browser restrictions on local assets.

## Layout

| Path | What it is |
|---|---|
| `index.html` | The whole page — hand-written HTML, CSS in a single `<style>` block, no JavaScript. |
| `media/tinycoder.jpg` | Logo, used in the nav and hero. |
| `media/icon.svg` | Favicon and footer mark. |
| `media/icon.png` | `apple-touch-icon`. |
| `LICENSE` | MIT. |

Do not delete or rename anything in `media/`; the page references those exact filenames.

## Repository map

Every outbound link on the page points at one of the three product repositories below — including the deep
links to `BENCHMARKS.md` inside the engine repository — plus the local `LICENSE` file. The page never links
to its own repository: a site does not need to cite itself, and the footer points outward only.

| Repo | URL | Role on the page |
|---|---|---|
| `tinycoder-inference` | <https://github.com/mgorshkov/tinycoder-inference> | Hero primary CTA; engine links in "What it is" and EXL3; the full benchmark table. |
| `tinycoder-openai` | <https://github.com/mgorshkov/tinycoder-openai> | The OpenAI-compatible gateway: the API section's link and the endpoint table's source of truth. |
| `tinycoder` | <https://github.com/mgorshkov/tinycoder> | The VS Code extension — a downstream consumer, deliberately secondary. |

The page does not link to this repository (`tinycoder-web`): it is where the site lives, not something the
site advertises.

Link, don't embed: no badges, no remote images, no scripts that fetch at view time. Every URL above was
checked with `curl -sI` and returns `200`.

The "complete table" links point at the engine's default branch (`main`). The `1.0.0` branch that
previously carried `BENCHMARKS.md` has been merged into `main` and removed, so all links must use
`main`. Verify with:

```bash
curl -s -o /dev/null -w "%{http_code}\n" \
  https://raw.githubusercontent.com/mgorshkov/tinycoder-inference/main/BENCHMARKS.md
```

A `200` means the link target exists; a `404` means the branch or path changed again.

## Updating the benchmark numbers

**The single source of truth is `tinycoder-inference/BENCHMARKS.md`.** Never edit a benchmark number in
`index.html` by hand, and never retype one from memory or from a chat transcript.

1. Read the current table in `tinycoder-inference/BENCHMARKS.md`.
2. Copy the figures **verbatim** — same digits, same decimal places, same thousands separators. `15.24` is
   not `15.2`; `10,149.8` is not `10150`.
3. Keep the selection rule below, and keep the protocol line in sync with `BENCHMARKS.md`.
4. Check that every figure on the page still appears verbatim in `BENCHMARKS.md`.

### Selection rule

The benchmark table on the page is titled **"Where TinyCoder is faster"** and is explicitly a *selection*,
not the full survey. A row is included only if TinyCoder is faster in **both** columns (pp512 and tg128).
As of the current `BENCHMARKS.md`, that yields exactly four rows:

| Row | Why it qualifies |
|---|---|
| Qwen2.5-Coder-1.5B `q2_k` | faster in both columns |
| Qwen2.5-Coder-1.5B `iq3_xxs` | faster in both columns |
| Ornith-1.5-35B-A3B | faster in both columns |
| Qwen3.8-27B EXL3 `1.60bpw` | faster in both columns |

Rows where the engine loses a column are deliberately excluded (`Qwen2.5-Coder-7B iq2_s` wins prefill only;
`Qwen2.5-Coder-7B iq3_xxs` and `Ornith-1.5-9B` lose both). That is allowed *only* because the page says it is
a selection and links the complete table directly under it. If that pointer is ever removed, the filtered
table becomes misleading — do not remove it.

### Formats without a baseline

`Ternary-Bonsai-2-27B pq2_0` has no *upstream* llama.cpp baseline: llama.cpp has no loader for the Prism
ternary format. Prism's own llama.cpp fork does load it, and `BENCHMARKS.md` publishes that run (three
alternating interleaved pairs, means reported): 878.5 tok/s pp512 and 50.36 tok/s tg128 against TinyCoder's
880.6 and 51.73. So the row is a real comparison, but the baseline is a fork rather than upstream llama.cpp
and must be labelled as one wherever it is shown — the wins table therefore carries a **Baseline engine**
column, so every row names the engine it is measured against instead of saying it once in a caption.

The row does belong in the wins table: TinyCoder leads decode by 2.7% (51.73 against 50.36) and prefill by
2.1 tok/s, and that prefill margin is inside the fork's 1.5 tok/s spread, which the row's sub-line says
plainly rather than hiding behind a bold number. The "Formats the other engines can't run" block keeps the
format-breadth argument, with the fork numbers cited instead of dashes.

The same block must keep the EXL3 distinction: EXL3 is also a format llama.cpp does not load, but it *does*
have a baseline (exllamav3 1.6.6). TinyCoder leads it on prefill (+1.1%) and trails on decode (-0.7%, both
measured interleaved, `scripts/exl3_ab_bench.sh`); state both directions, not the favourable half. A measured win must not be lumped in with an
unmeasured capability.

## Copy rules

- **No claim of a win the benchmark table does not show.** If `BENCHMARKS.md` does not have the row, the page
  does not have the claim. This applies to speed, memory, tokens-per-second and "runs on X" statements.
- **No invented API surface.** The engine repository ships a library (`tinycoder_core`), not an HTTP server.
  The OpenAI-compatible endpoints documented on the page are the ones implemented in the separate
  `tinycoder-openai` gateway — verify against that source before changing the endpoint table or the
  "what it is not" list.
- **No unverified model names, flags, versions or hardware claims.** Take them from the engine's README,
  `BENCHMARKS.md` and source, not from memory.
- Keep the honest notes: the exllamav3 decode-margin caveat, the text-gate rule, and the fact that llama.cpp
  wins configurations in the full table.

## Constraints

- Static HTML only. No build step, no JavaScript framework, no bundler.
- No external CDN dependencies — no web fonts, no analytics, no remote scripts or styles. Everything must
  render offline from this directory.
- Real headings, alt text on images, visible focus styles, and a colour palette with sufficient contrast.
- The page currently contains no JavaScript at all. Keep it that way unless there is a compelling reason;
  content must never depend on script execution to become visible.

## License

MIT — see [LICENSE](LICENSE).
