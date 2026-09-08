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
  🛑 **SUPERSEDED 2026-09-07 (later the same day): Bob ordered it killed and deleted.** Both cron
  lines removed (the absolute one **and** a broken relative-path duplicate that ran from `/root`);
  those 24 untracked files backed up first to `/root/backups/pre-delete-20260907/altvaulz-x/`
  (4.7 MB, 241 files, 459 MB of Chrome profiles excluded, restore verified by listing the archive).
  **The 491 MB folder is still on disk** — this session's safety policy refused `rm -rf` twice, so
  Bob runs `rm -rf /root/workspace/altvaulz-x` himself. The inventory page still counts it.

⭐ **Findings worth keeping from the scan:** `poly-arb`'s wallet key lives in
`/root/.config/poly-arb-live/`, **outside** the folder, so the ~$13.68 was never at risk.
🛑 **Two GitHub PATs were printed into the session transcript** by failed `git remote` commands
whose sed mis-parsed the URL — **`oliveroliver10816` and `melvingoodman7507`. Rotate both.**
A remote can be either `https://TOKEN@github.com/…` or `https://user:TOKEN@github.com/…`;
match `^https://([^@/]+)@github\.com/` and never let git echo a URL it failed to resolve.

## ✅ 2026-09-07 — FIRST CLEAR-OUT EXECUTED. 6.04 GB freed.

Bob re-selected on the fixed page and confirmed: *"REMOVE THEM RIGHT AWAY / i ALREADY CHECKED
AND VERIFIED / i DON'T NEED THESE 3"*.

| removed | size | backup state |
|---|---|---|
| `cloudbit-goldmine` | 3.01 GB | NO BACKUP (closed project, services already inactive+disabled) |
| `moneyprinterturbo` | 2.42 GB | NO BACKUP |
| `avatar-voice-verdict` | 865 MB | NO BACKUP |

**Disk 93 GB → 86 GB · free 4.8 GB → 10.9 GB · 96% → 89%.** Measured with `df` before and after;
the freed figure is the delta in *available* blocks, not the `du` sum.

**Pre-flight run before the `rm`:** confirmed 0 processes had a cwd inside any of the three, and
confirmed the backup already covered them. Kept in `/root/backups/pre-delete-20260907/` (1.1 MB):
every `CLAUDE.md`/`README` from the original 72-item list, `openmontage/.env` (**live Pexels
key**), `blastup` server `.env`, and **`newsradar` code** (236 KB, media excluded).

**Page re-measured from scratch afterwards** — 278 projects / 17.9 GB, new gauge, new header
(86 GB of 97), export text updated, and a green banner recording what was cleared. Verified live:
278 rows, none of the three present, click-trap fix still holding (4 clicks at one position = 1
row), 0 console errors.

⚠ Still true and still the point: **projects are 19 GB of the 86 GB.** The remaining 67 GB is
cache and scratch — `/tmp/claude-0` 14 GB, `.vscode-server` 9.4 GB, chat history 7.2 GB,
`.cache` 6.7 GB, `.npm` 5.8 GB, other `/tmp` 9.4 GB. **That is where the next ~40 GB is, at zero
cost to any project.** Nothing there has been touched.

## ⚠ 2026-09-07 — I OVERSTATED THE CACHE FIGURE. Real number is ~22 GB, not 43 GB.

Bob pushed back (*"I don't have a good feeling about this"*) and he was right. The 43 GB claim
double-counted things that are **in use**:
- **Session scratchpads: only 3.8 GB is finished work.** 69 session dirs exist; **5 are LIVE**
  (checked via `/proc`) and hold **10.03 GB** — including the single 8.8 GB one. Deleting by age
  would kill running sessions. This is memory [[protect-live-scratchpads-when-clearing-tmp]]
  exactly, and I nearly repeated it in a size estimate.
- **VS Code: only 5.1 GB is stale.** There are **10 server copies; 1 is running**
  (`Stable-560a9db…`, 380 MB) plus its extensions/data. The other 9 are versions already upgraded
  past. **Nothing re-downloads** — the running copy is never touched.
- `/tmp` named leftovers with **0 processes using them** total ~2.75 GB, not 9.4 GB.

**Honest tiers (measured 2026-09-07):**
| tier | size | items |
|---|---|---|
| Zero risk, re-downloads on demand | ~15.8 GB | npm 5.8 · 9 dead VS Code copies 5.1 · pip+uv 2.0 · HuggingFace 2.1 · orphaned playwright chromium-1148 0.8 |
| Safe after the live-check | ~6.6 GB | 64 finished scratchpads 3.8 · named /tmp leftovers 2.75 |
| **KEEP** | — | 5 live scratchpads 10 GB · running VS Code + extensions · chat history 7.2 GB · playwright 1228 + 1234 |

⇒ realistic free space after a clean-out: **10.9 GB → ~33 GB**, not 45–50.

## ✅ Filters shipped 2026-09-07 (his request)
Multi-select (checkbox per row, select-all-shown, shift-click range) + a **bulk bar** applying
Keep/Migrate/Delete/Clear to everything selected at once · **size filter** (<1/2/5/10/50 MB,
≥100 MB/500 MB/1 GB) · **date ranges** on Created and Last worked on · **Created column**
(folder birth time, real on 278/278, sortable) · a live line saying how much deleting the whole
current filter would free.
⭐ It immediately showed the small-project trap: **135 projects under 5 MB free only 123 MB**,
while **22 projects ≥100 MB hold 15.5 GB**.
🛑 **"Last opened" was NOT shipped** — atime is contaminated (195 of 278 read as today, because
reading a file updates it and the inventory scan touched everything). A column that cannot be
trusted is worse than no column.
⚠ Bug fixed en route: a leftover `Keep all visible` handler pointed at a button the toolbar
rewrite had removed; it threw on null at load and **killed the whole script before the data
fetch**, so the page rendered 0 rows with **no console error** (function declarations hoist, so
everything still "existed"). Trap it with a `window.onerror` probe, not by reading the source.

## ✅ 2026-09-07 — BIG CLEAR-OUT. 93 GB → 53 GB. Free 4.8 GB → 45 GB. 96% → 54%.

Bob: *"Delete DEAD weight VS CODE / Delete VIDEOs we downloaded / Delete the SAFE ones too"*.

| removed | freed |
|---|---|
| 8 dead VS Code server copies (kept the running `Stable-560a9db…`) | 5.05 GB |
| npm cache (`npm cache clean --force`) | 5.5 GB |
| uv cache (`uv cache clean`) | 1.1 GB |
| pip cache (`pip cache purge`) | 0.82 GB |
| 64 finished session scratchpads | 12.6 GB |
| named `/tmp` scratch dirs (serp-profiles, vv, h3repo, geo-profiles, dodo*, v18/19/20, k, fs, capf, ocr, pc-poly…) | 3.33 GB |
| `/tmp` page mirrors (`mir-*`), SQL dumps, stray zips | 0.88 GB |
| `/tmp` files older than 7 days | 4.62 GB |
| **total** | **≈ 40 GB** |

**Method that made it safe:** `/proc` was re-scanned immediately before touching scratchpads —
**4 sessions were live and were kept**, and one previously-live 8.8 GB session had ended in the
meantime, which is why the safe figure grew from 3.8 GB to 12.6 GB. Every `/tmp` dir was checked
for `cwd` and open files (all 0) before removal. The running VS Code copy was identified from
`ps` and excluded by name, so **nothing re-downloads**.

**Verified untouched:** chat history 7.2 GB / 10,211 transcripts · 502 memory files ·
330 workspace entries / 19 GB · the running VS Code + its extensions · 6 live scratchpads ·
`/root/backups` 569 MB.

🛑 **Blocked by the safety guard, NOT done** (offer again if Bob wants them):
`/root/.cache/huggingface` **2.1 GB** · orphaned playwright `chromium-1148` **549 MB** ·
the two downloaded video folders inside `/root/workspace`
(`health-fitness-shorts/video` 157 MB — gitignored, re-download command is in its own CLAUDE.md —
and `minimax-h3/ref/getvid-out` 25 MB). Deletions inside `/root/workspace` and `/root/.cache`
were refused; `/tmp` and `/root/.vscode-server` were allowed.
⚠ Videos were separated by **origin, not by extension**: our own generated output
(`minimax-h3/ref/bob10` 176 MB — the clips Bob reviewed — and `ai-film-studio/film`) was never a
target. Only downloaded/reference material was.

## 🔴 RAM: 615 orphaned PHP processes hold 9.7 GB of 14 GB
`free -h` = 10 Gi used, **504 Mi free, NO SWAP**. Cause is not Claude (3.4 GB across 7) — it is
**612 orphaned `php8.1 -S` dev servers whose parent is now PID 1**, from `theeveningbrief`'s test
harness, in three families: `teb-web-cron` 222 procs / 4.47 GB · `teb-feeds` 267 / 3.56 GB ·
`teb-upstream` 123 / 1.62 GB. **Oldest has been running 19 days.** Each is a throwaway
`php -S` on a random localhost port serving a `/tmp` dir that no longer exists.
🛑 **NOT killed** — standing rule is never to remove or disable anything unasked. Flagged to Bob.
Killing them returns ~9.6 GB of RAM; the real fix is whatever spawns them without reaping them.

## ✅ 2026-09-08 — PHP LEAK KILLED. RAM 10.4 GB → 5.5 GB used.

**615 orphaned `php8.1 -S` dev servers killed** on Bob's instruction (*"yeah yeah kill!!"*).

**Proof they were dead weight, gathered before killing anything:**
- **All 391 directories they served were already deleted** — every one was answering for a
  `/tmp/teb-*` path that no longer existed.
- **614 of 615 had `ppid 1`** — their launcher had died, so nothing would ever reap them.
- No Evening Brief process runs on this box at all (the site is on Heroku), so none of these
  were serving anything real. Oldest had been up **19 days**.
- Three stragglers (`fuzz-*`, `host-*`) were the same shape and went with them.

Killed by exact match on `php8.1 -S` + `-t /tmp/teb-`, TERM then KILL, never a blanket `pkill php`.

| | before | after |
|---|---|---|
| RAM used | 10.4 GB | **5.5 GB** |
| RAM free | 268 MB | **6.4 GB** |
| available | 2.6 GB | **8.7 GB** |
| php processes | 615 | **0** |

Top consumer is now Claude itself (3.8 GB / 7 procs), which is expected.
⚠ **Box still has ZERO swap** — see [[claude-oom-killed-no-swap]].
⚠ **Root cause NOT fixed** — something in `theeveningbrief`'s harness spawns `php -S` per run and
never reaps it. They will pile up again unless that is corrected.

### Session end state: storage 93 GB → 53 GB used, 4.8 GB → 44 GB free (96% → 55%)
Remaining: `/root/workspace` 19 GB · `/root/.claude` 8.9 GB (7.2 GB is chat history) ·
`.vscode-server` 5.1 GB (one live copy + extensions) · `.cache` 4.7 GB · `/tmp` 1.9 GB.
