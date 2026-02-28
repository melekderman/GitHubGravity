# Codebase Health Map 🧬

A single-page interactive visualization that scans any public GitHub repository and turns its recent engineering activity into a **living health map**. Files become orbiting nodes — their color reflects recency, their size reflects churn, and their distance from the center reveals collaboration risk.

---

## What It Does

Enter any public GitHub repository in `owner/repo` format and the app builds a visual health profile of its codebase using recent repository activity.

Each file is rendered as a node in a force-directed orbital system:

- **Color → Recency** — Recently touched files glow green; aging files shift toward amber and red; untouched files fade into cold blue-gray.
- **Size → Churn** — Files that appear in more recent commits become larger, highlighting hotspots of change.
- **Orbit → Bus Factor Risk** — Files touched by only one contributor are pulled inward, making high-risk ownership concentration immediately visible.
- **Warning Rings → Maintenance Signals** — Pulsing rings mark files with bus-factor risk or bug-fix-heavy activity.
- **Links → Directory Structure** — Each file is tethered to the center through parent-path bonds, preserving a sense of repository anatomy.

The result is a compact, visual snapshot of codebase health: active files, fragile ownership zones, and potentially unstable hotspots stand out at a glance.

---

## How the Health Model Works

The visualization combines lightweight GitHub metadata with recent commit history to estimate file-level risk and activity.

### Signals

| Signal               | Source                                                                 |
| -------------------- | ---------------------------------------------------------------------- |
| **Recency**          | Most recent commit date associated with each file                      |
| **Churn**            | Frequency of a file appearing in recent commit diffs                   |
| **Contributor count** | Unique authors touching that file within the analyzed commit window    |
| **Bug-fix ratio**    | Commit messages containing keywords: `fix`, `bug`, `patch`, `error`, `hotfix`, `regression` |

### Risk Heuristics

- **Bus factor risk** — Flagged when a file has meaningful recent churn but only one author.
- **Bug-prone** — Flagged when a large share of recent changes come from bug-fix-like commits.
- **Untouched** — Files with no recent activity remain in the outer orbit with minimal visual emphasis.

> [!NOTE]
> This is intentionally heuristic, not a formal software quality metric. It is meant to reveal patterns quickly, not replace deeper engineering review.

---

## Visualization Model

Several D3 force components run together to create the map:

| Force             | Role                                                              |
| ----------------- | ----------------------------------------------------------------- |
| `forceLink`       | Connects each file to its parent structure, preserving repo shape |
| `forceManyBody`   | Repels nodes from one another to prevent overlap                  |
| `forceRadial`     | Places files at orbital distances based on collaboration risk     |
| `forceCollide`    | Enforces physical spacing using each node's actual radius         |

Additional visual layers:

- Canvas-based animated starfield background
- Glow filters for high-importance nodes
- Pulsing alert rings for risky files
- Hover tooltips with per-file health details
- Auto-fit zoom after simulation warmup

---

## Getting Started

No install required — open directly in a browser.

```bash
git clone https://github.com/your-username/codebase-health-map
open index.html
```

Or simply download the single HTML file and open it locally. No build step, no framework, no backend.

### Scanning a Repository

1. Enter a public repository in `owner/repo` format.
2. Click **SCAN** or press **Enter**.
3. Explore the generated health map.

### Optional: GitHub Token

For deeper analysis, paste a **GitHub Personal Access Token** into the token field.

| Mode            | Behavior                                                                 |
| --------------- | ------------------------------------------------------------------------ |
| **Without token** | Loads the repo and recent commits but skips detailed per-file inspection; falls back to lightweight heuristics |
| **With token**    | Fetches recent commit details and produces a much richer file-level health map |

---

## Controls

| Action                       | Result                              |
| ---------------------------- | ----------------------------------- |
| Type `owner/repo` + Enter    | Scan a repository                   |
| Click **SCAN**               | Start analysis                      |
| Scroll                       | Zoom in / out                       |
| Click + drag background      | Pan                                 |
| Click + drag node            | Reposition a file manually          |
| Hover a node                 | See file health metrics in tooltip  |

---

## Visual Legend

### Recency Colors

| Color           | Meaning                              |
| --------------- | ------------------------------------ |
| 🟢 Bright green | Updated within 7 days               |
| 🟩 Lime         | Updated within 7–30 days            |
| 🟡 Amber        | Updated within 1–3 months           |
| 🟠 Orange       | Updated within 3–12 months          |
| 🔴 Red          | Updated more than 1 year ago        |
| 🔵 Blue-gray    | Never touched in the analyzed window |

### Indicators

| Marker              | Meaning          |
| ------------------- | ---------------- |
| Red pulsing ring    | Bus factor risk  |
| Orange pulsing ring | Bug-prone file   |

### Encoding Summary

- **Node radius** → Commit churn
- **Orbit distance** → Inverse collaboration depth
- **Link line** → Structural bond to repository layout

---

## API Strategy

The app uses the **GitHub REST API** directly from the browser.

### Core Requests

- `GET /repos/{owner}/{repo}` — Repository metadata and default branch
- `GET /repos/{owner}/{repo}/git/trees/{branch}?recursive=1` — Recursive file tree
- `GET /repos/{owner}/{repo}/commits` — Recent commit list
- `GET /repos/{owner}/{repo}/commits/{sha}` — Commit details for selected recent commits

### Rate-Limit Considerations

- File scan is capped for performance.
- Recent commits are limited to a small window.
- Detailed commit inspection is batched with short delays between requests to reduce rate-limit pressure.

---

## Technical Stack

| Layer      | Technology                                        |
| ---------- | ------------------------------------------------- |
| Layout     | [D3.js v7](https://d3js.org/) — force simulation, drag, zoom, SVG rendering |
| Data       | GitHub REST API v3 — repository tree and commit data |
| Frontend   | Vanilla HTML / CSS / JS — single-file architecture |
| Background | Canvas — animated starfield                       |
| Graph      | SVG — interactive node graph and overlays         |

---

## Limitations

- The analysis is heuristic, not a formal code quality audit.
- Per-file health is only as good as the recent commit window being sampled.
- In limited mode (no token), file-level signals are partially approximated.
- Bug-prone detection relies on commit message keywords and can misclassify.
- Large repositories are capped for visual performance.
- Older or low-powered devices may experience slower force simulation.

---

## Why It Exists

Most repository tools tell you *what files exist*.
This one tries to show **how a codebase behaves**.

Instead of a static tree, you get a visual field of ownership concentration, activity hotspots, stale zones, and maintenance pressure — all in one screen. Designed for quick exploration, technical storytelling, and making hidden code health patterns visible.

---

## License

[MIT](LICENSE) — Use it freely, modify it, and adapt it to your own engineering workflows.
