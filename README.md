# Git Quest

Learn Git **branching**, **merging**, **rebasing** and **conflict resolution** by
playing 20 puzzles against an animated commit graph. In the spirit of
*Learn Git Branching* and *Oh My Git!*, built for a classroom: bilingual
(English / 中文), no build step, no server, no network, no account.

**▶ Play: https://sclim999.github.io/git-quest/**

用关卡和动画学 Git 分支：把 **合并 (merge)**、**变基 (rebase)**、**冲突解决**
做成 20 关解谜。中英双语，纯静态网页 —— 打开就能玩，不需要安装、不需要联网、
不需要注册。老师可以改关卡，也可以选择把学生的星星记录到 Google 表格。

---

## What is in the box

* **A real Git simulator** — not a slideshow. `js/git-engine.js` keeps a genuine
  commit DAG and implements three-way merges, fast-forward vs. merge commits,
  `rebase` / `rebase --onto` / `cherry-pick` replay (copies are marked `C3'`,
  as in Git's own documentation), conflict markers with staged resolution,
  `--continue` / `--abort` / `--skip`, `reset` vs. `revert`, tags, detached
  HEAD, and relative refs like `main~2` and `HEAD^2`.
* **An animated graph** — `js/git-viz.js` tweens the SVG graph after every
  command, so a rebase visibly replays commit by commit and branch labels glide
  to their new commit. A second, smaller graph always shows the **target**
  state, so the puzzle is never a guessing game.
* **20 levels in 5 worlds** — commits & branches → merging → rebase &
  cherry-pick → conflicts → undo & a final boss.
* **Game layer** — stars (3 if you match the reference solution's command
  count), XP and ranks, 11 badges, sequential unlocking, hints, a step-by-step
  "show solution" playback, and a free-play **Sandbox**. Progress is saved in
  the browser's `localStorage`.
* **A conflict editor** — when a merge or rebase stops, the conflicted file
  appears with the real `<<<<<<< / ======= / >>>>>>>` markers plus
  *keep ours / keep theirs / keep both* buttons. The student still has to
  `git add` the file and finish the operation, exactly as in real Git.

## Files

| File | Purpose |
|---|---|
| `index.html` | The game page |
| `css/git-game.css` | Styling (dark console theme, single-theme by design) |
| `js/git-engine.js` | The Git simulator and the goal comparison |
| `js/git-viz.js` | Animated SVG commit-graph renderer |
| `js/git-levels.js` | **Level pack — edit this to add or change puzzles** |
| `js/git-portal.js` | Game loop: XP, stars, badges, terminal, hints |
| `js/config.js` | Optional spreadsheet URL for progress reporting |
| `tools/build-standalone.py` | Bundles everything into one HTML file |
| `google-apps-script/Code.gs` | Optional Google Sheets recorder |

## Running it

Open `index.html` in any modern browser. That is all — it also works straight
from `file://`, from a USB stick, or from any static host.

To publish it for a class on **GitHub Pages**: *Settings → Pages → Deploy from
branch → `main` / `/ (root)`*. The game is then served at
`https://<user>.github.io/git-quest/`.

## Handing it out as a single file

```
python3 tools/build-standalone.py git-quest.html
```

The CSS and all four scripts are inlined, so the result is one file you can
e-mail or hand out offline. The *Report to lecturer* panel is left out by
default, because sandboxed hosts block its request; add `--keep-report` when
you are publishing to a normal web host. `--artifact` emits the same page
without the `<html>` / `<head>` / `<body>` wrapper, for hosts that supply their
own document skeleton.

## Adding or editing a level

A level is two command scripts, so you never hand-draw a target graph — the
game builds it by running your own solution:

```js
{
  id: "3-5", world: 3,
  name:  { en: "…", zh: "…" },
  brief: { en: "…", zh: "…" },           // what the student reads
  goal:  { en: "…", zh: "…" },           // one-line objective
  setup:    ["git commit -m Base", "git checkout -b feature", "git commit -m Work"],
  solution: ["git rebase main"],         // reference answer -> target graph + par
  compare:  { files: true, head: true }, // also pin file contents / where HEAD ends
  hints: [{ en: "…", zh: "…" }]
}
```

Setup and solution scripts may use every simulator command plus two helpers:
`edit <file> "content"` (change a file — `\n` becomes a newline) and
`resolve <file> --ours|--theirs|--both` (the stand-in for opening an editor on a
conflict).

**How a level is judged.** The player passes when the *shape* of their graph
matches the target: every branch's and tag's ancestry structure, the total
number of reachable commits, plus file contents at each branch tip when
`compare.files` is set and HEAD's location when `compare.head` is set. Commit
ids and commit messages are deliberately ignored, so any route that builds the
right history counts — a student who gets there a different way still wins.
Only commands that change history count towards the star rating, so
`git status`, `git log` and editing files are free.

Two teaching conveniences worth knowing about:

* `git commit` with nothing staged invents a small change (a uniquely named
  `fN.txt`) so the first levels can be played by typing `git commit` alone.
  These placeholder files are collapsed in the working-tree panel and never
  take part in goal checking.
* Conflicts are whole-file rather than line-by-line: if both sides changed a
  file differently, the whole file conflicts. It keeps the markers readable for
  beginners.

## Recording stars to a spreadsheet (optional)

1. Create a Google Spreadsheet, then **Extensions → Apps Script**.
2. Paste `google-apps-script/Code.gs`, save, then **Deploy → New deployment →
   Web app** with *Execute as: Me* and *Who has access: Anyone*.
3. Copy the `/exec` URL into `SHEETS_WEBAPP_URL` in `js/config.js`.

Students then get a *Report to lecturer* panel: name, ID, class and a
*Send progress* button. Nothing is ever sent automatically, and no data leaves
the browser until that button is pressed. One row per student, one column per
level (stars out of 3).
