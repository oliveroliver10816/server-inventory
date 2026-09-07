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

## 🛑 2026-09-07 — THE CLICK TRAP. Bob's selection was WRONG and it was my bug.

He exported **72 deletes; 58 of them had been worked on within 30 days**, one edited that same
day (`altvaulz-x`). He stopped the run: *"Projects that were recently worked on shouldn't be
deleted. This page you created gave out some MORE pages / project names than I selected."*
**He was right.**

**Cause:** every Keep/Migrate/Delete click called a full `draw()`. With the **Undecided** filter
on, the decided row stopped matching, vanished, and the list **shifted up under the cursor** —
so each next click at the same screen position hit a *different* project. Proven: 5 clicks at a
fixed position marked **5 different projects**; after the fix, **1**.

**Fixed (live, re-verified on the published URL):**
- `applyRow()` updates the clicked row **in place** — no re-render, the row never moves.
  Filter/sort/search re-render only when the user changes them.
- **Undo** button, stepping back through decisions one at a time (disabled when empty).
- Age shown in **red** for anything ≤30 days, **TODAY** called out explicitly (120 of 280 rows).
- New stat tile: **"Deletes worked on <30d"**.
- The export now **leads with a WARNING block** naming every recent project in the delete set.

⚠ **The test that would have caught it:** click the SAME coordinates N times and assert you hit
the same row. My original tests clicked `rows[0]`, `rows[1]`, `rows[2]` **by index**, which can
never expose this. See memory [[re-render-on-click-moves-the-next-target]].

## State after the aborted deletion run — NOTHING WAS DELETED
The safety scan ran before any `rm`, which is what left room to stop. All 72 folders intact,
disk unchanged at 93 GB / 96%. Only additive changes were made:
- `/root/backups/pre-delete-20260907/` (1.1 MB, 24 files) — every `CLAUDE.md`/`README` from the
  72, the `openmontage` Pexels key + `blastup` server `.env`, and **newsradar's code**
  (236 KB, media excluded — it is inside `moneyprinterturbo`, is not a git repo, and its cron is
  only commented out, so deleting that folder would have destroyed it).
- `poly-arb` working tree committed and pushed to a **new branch `archive-20260907-local`**
  (remote `main` untouched — it had diverged).
- `snow-safari` — his 2 unpushed commits pushed to main.
- `abilene-teardown` — `.gitignore` change committed and pushed.
- `altvaulz-x` left completely untouched, still holding its **24 untracked files including the
  live `tracker.py` the cron runs** — "ON GITHUB" did NOT cover them.

⭐ **Findings worth keeping from the scan:** `poly-arb`'s wallet key lives in
`/root/.config/poly-arb-live/`, **outside** the folder, so the ~$13.68 was never at risk.
🛑 **Two GitHub PATs were printed into the session transcript** by failed `git remote` commands
whose sed mis-parsed the URL — **`oliveroliver10816` and `melvingoodman7507`. Rotate both.**
A remote can be either `https://TOKEN@github.com/…` or `https://user:TOKEN@github.com/…`;
match `^https://([^@/]+)@github\.com/` and never let git echo a URL it failed to resolve.
