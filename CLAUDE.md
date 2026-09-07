# server-inventory — the 93 GB keep/migrate/delete list

**LIVE (noindex): https://oliveroliver10816.github.io/server-inventory/**
Repo `oliveroliver10816/server-inventory`. Built 2026-09-07 because the disk hit 96%.

## What it is
One page listing **302 items = 280 workspace projects + 22 system/cache paths**, each with
size, last-used date, GitHub-backup state and reclaimable build junk. Every row has
**Keep / Migrate / Delete**; a live counter shows space freed and the resulting disk figure.
**Export choices** gives a plain-text list (copy or .txt download). Choices live in
localStorage only — Bob must press Export to send them back.

## The headline finding
**Projects are NOT the problem.** 24.2 GB of the 93 GB is workspace projects.
**58 GB is system/cache/scratch**, nearly all self-regenerating:
- `/tmp/claude-0` session scratchpads **14 GB** (one session folder alone is 8.8 GB)
- `/root/.vscode-server` **9.4 GB** (stale versioned CLI copies)
- `/root/.claude/projects` **7.2 GB** = 10,204 chat transcripts (⚠ this is `claude --resume`)
- `/root/.cache` **6.4 GB** (Playwright 2.4G, HuggingFace 2.1G, uv 1.2G, pip 840M)
- `/root/.npm` **5.8 GB**
- other `/tmp` **10 GB**

⇒ this is why storage keeps filling: **nothing prunes the caches or session scratch.**
Deleting projects buys little and costs work; pruning caches buys ~40 GB and costs nothing.

## Data provenance — nothing inferred
- sizes: `du -sk` per directory, 2026-09-07
- git state: `git rev-list --count @{u}..HEAD` per repo → **108 git repos, 172 non-git**
- **22 repos carry unpushed commits or no tracking branch** (vantage 6, poly-arb 1,
  swagacity 1, ringondemand 24, lendpeak-lander 11, coolizi 6, glpure, gcalit …)
- **172 projects are NOT git repos = this box is the only copy** (18.7 GB)
- descriptions: first heading + first line of each project's own `CLAUDE.md`/`README`

## Traps hit while building (don't repeat)
- ⚠ **`overflow:hidden` on the table killed `position:sticky` on `thead`.** Removed it and
  rounded the corner cells by hand. An `overflow-x:auto` wrapper does the same thing — it is
  now media-queried to ≤1100px only, so wide screens keep sticky headers.
- ⚠ **The copy button lied.** The `catch` fallback set "Copied ✓" whether or not the copy
  worked. It now reports honestly; proved by sabotaging BOTH `clipboard.writeText` and
  `execCommand` and watching the badge flip to "Copy blocked".
- ⚠ `browse fill "#q" ""` does NOT clear a search box — dispatch an `input` event manually.
- ⚠ gstack browse wrote `.gstack/browse.json` (holds a live daemon token) into the project
  dir — gitignored before the first `git add`.
- ⚠ Port 8791 was already taken by another session's server; used 8847 and left 8791 alone.

## Verified before publish
0 console errors · 302/302 rows render · no horizontal overflow at 1720/1280/768/390 ·
sticky toolbar + thead confirmed at scroll · all 6 filters, both sort directions, toggle-off,
localStorage persistence, export text and clipboard all exercised on the **live** URL.

## Open / next
- **Awaiting Bob's choices** — he exports the list, we execute.
- 🛑 A GitHub PAT for `oliveroliver10816` was printed into the session transcript by a failed
  `git remote add` (bad sed). **Flagged to Bob — recommend rotating that PAT.**
- Nothing has been deleted. This is a read-only inventory.
