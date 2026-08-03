# Coding Interview Mastery — Theory Notes

👉 **Live site:** **[kid1995.github.io/coding-interview-mastery](https://kid1995.github.io/coding-interview-mastery/)**

A study vault covering the foundational theory needed to solve:
- **Elevator Saga**-style simulation/scheduling problems ([play.elevatorsaga.com](https://play.elevatorsaga.com/))
- **LeetCode**-style algorithm exercises
- **System design** interview questions
- **Software resilience** (surviving real-world failure)
- Writing **easily-testable code**

## Structure

```
docs/
└── Mindmap/
    ├── 00 - Coding Interview Mastery Mind Map (Index).md   ← start here
    ├── 01 - Algorithms & Data Structures/
    ├── 02 - Simulation & Scheduling/
    ├── 03 - System Design Fundamentals/
    ├── 04 - Software Resilience/
    └── 05 - Testability & Clean Code/
```

Each category folder has an `00 - <Category> (Overview).md` plus one note per leaf topic. Every
note follows the same template: Definition & Core Concepts → Best Practices → Real-World Use Case
→ Hands-On Practice → Exam Tips → References → Related.

`docs/` is a versioned snapshot of the canonical vault, which lives in this user's iCloud Obsidian
vault at `EXAMS/CODING-INTERVIEW-MASTERY/Mindmap/` (kept in sync across devices). Edit the vault
copy for day-to-day study; re-copy into `docs/` when you want a new versioned snapshot here.

## Origin

Built with the `study-mindmap-vault` Claude Code skill — no pre-existing syllabus, the topic tree
was designed from scratch across the 5 theory areas above, with every reference link verified live
via web search before being cited.

## Status

Docs-only for now. Practice-code folders (Elevator Saga solutions, LeetCode solutions) can be added
later once you're actively solving specific problems.

## Publishing (Quartz)

The site is generated from `docs/Mindmap/` using [Quartz](https://quartz.jzhao.xyz/), which turns
Obsidian-style wikilinks into a real linked website (graph view, search, backlinks included).
`content/` is a symlink to `docs/Mindmap/` — there's only one copy of the notes, Quartz just reads
them from that path instead of the default `content/` folder name.

- **Preview locally:** `npm install --legacy-peer-deps && npx quartz build --serve`, then open
  `http://localhost:8080`.
- **Deploy:** automatic — `.github/workflows/deploy.yml` builds and publishes to GitHub Pages on
  every push to `main`.
- **`--legacy-peer-deps` is required**: Quartz's own dependency tree has a peer-dependency conflict
  between `@quartz-community/latex` and `@myriaddreamin/rehype-typst` (unrelated to this repo's
  content) that plain `npm install` refuses to resolve.
- **No lockfile is committed** (this account's global `.gitignore` excludes `package-lock.json`
  across all projects), so CI uses `npm install` rather than `npm ci`.
- **`public/CNAME` is stripped after every build**: Quartz auto-generates it from
  `configuration.baseUrl` in `quartz.config.yaml`, but since this is a GitHub Pages *project* page
  (`kid1995.github.io/coding-interview-mastery`) rather than a custom domain, that file would
  incorrectly claim the bare `kid1995.github.io` apex domain. Project pages don't need a CNAME.

Re-running the `study-mindmap-vault` skill to add/update notes refreshes `docs/Mindmap/` as usual —
no extra step needed, since `content/` just symlinks to it.
