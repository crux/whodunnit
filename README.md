# whodunnit

A small standalone tool that generates a git contributor dashboard for any repository — commits, code/markup/config splits, contributor lifecycle, file hotspots, bus-factor risk analysis.

Single Python script, single HTML template, no dependencies beyond Python 3.10+ and `git`. Output is one self-contained HTML file you can open, share, or archive.

## Install

```sh
git clone https://github.com/crux/whodunnit.git
~/path/to/whodunnit/whodunnit /path/to/some/repo --open
```

Optionally symlink it onto your `$PATH`.

## Usage

```
whodunnit [REPO] [--since DATE] [--until DATE] [-o OUT] [--open]
```

- `REPO` — path to a git repository (default: current directory)
- `--since` / `--until` — passed straight through to `git log` (e.g. `2025-01-01`)
- `-o` — output HTML path (default: `./whodunnit.html`)
- `--open` — open the result in your browser

### Examples

```sh
whodunnit ~/code/myrepo --open
whodunnit ~/code/myrepo --since 2025-01-01 -o report-2025.html
whodunnit ~/code/myrepo --since 2026-01-01 -o ytd.html --open
```

## What it shows

The dashboard is a single HTML file with four sections:

- **Who** — commits per author, share, lines added by category (code / markup / config / data / lockfile / other), avg lines per commit, tenure timeline with active/hiatus/gone status
- **When** — cumulative commits, cumulative code-only lines, monthly velocity stacked by author, per-year leaderboard, codebase composition over time, per-author punch cards (day × hour)
- **What** — file change hotspots, extension breakdown
- **Risk** — bus factor (greedy removal), per-author solo/dominant file counts, onboarding hit list grouped by directory, per-directory bus factor, ownership-concentration histogram

## Author identity merging

`whodunnit` uses `git log %aN`, which respects the target repo's `.mailmap`. To merge duplicate identities, add a `.mailmap` file at the target repo's root. Email-based matching is the most robust:

```
# Rename all commits with this email
Canonical Name <canonical@email>

# Rewrite name and email based on commit email
Canonical Name <canonical@email> <commit@email>
```

This benefits every other git tool too (log, blame, shortlog, GitHub UI).

## Marking generated / vendored files

`whodunnit` respects the target repo's `.gitattributes` for `linguist-generated` and `linguist-vendored` markers — the same conventions GitHub's linguist library uses. Marked files are routed into the `vendored` category so they don't inflate author attribution.

```
# in your repo's .gitattributes
src/data/big-reference-dump.json linguist-generated=true
vendor/**                        linguist-vendored=true
docs/CHANGELOG.md                linguist-generated=true
```

This is the right way to handle bulk reference data, auto-generated lockfiles, or third-party content that one person happened to commit. It benefits GitHub's contributor graph too.

`whodunnit` also has a built-in path heuristic (`/node_modules/`, `/vendor/`, `/pdfjs/`, `/bootstrap/`, etc.) so the common cases work out of the box without any setup.

## Notes on metrics

- **Lines added/removed** is noisy. Lockfiles, generated code, vendored deps, and formatter sweeps inflate it. The category split exists so you can see what's actually in there.
- **Bus factor** is the smallest number of contributors whose departure would leave more than 50% of files with no remaining author. Computed via greedy removal.
- **File-level analyses** (hotspots, bus factor, ownership) are restricted to files that still exist in the tree at the end of the window. Historical activity (commits, lines added) uses the full window.
- **Merges** are excluded from line counts but included in commit counts.
- **Renames** are normalized to the post-rename path via git's `{old => new}` notation, so the same file is counted as one entity.
