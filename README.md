# GitHub Gravity 🌌

A single-page interactive visualization that turns any GitHub repository into a living solar system. Files become planets, their size defines their mass, and contributor activity bends the gravity pulling them toward the center.

![GitHub Gravity](https://img.shields.io/badge/built_with-D3.js-f97316?style=flat-square) ![License](https://img.shields.io/badge/license-MIT-4dffa8?style=flat-square) ![Zero dependencies](https://img.shields.io/badge/zero-backend-a78bfa?style=flat-square)

---

## What it does

Enter any public GitHub repository (`owner/repo`) and watch its entire file tree materialize as a force-directed particle system:

- **File size → mass** — larger files render as bigger nodes and exert stronger repulsion on their neighbors
- **Contributor activity → gravity** — files touched by more commits get pulled toward the center star; rarely-touched files drift to the outer orbit
- **Directory structure → link bonds** — each file is tethered to its parent directory, with bond strength proportional to commit weight
- **File type → color** — at a glance you can see whether a repo is source-heavy, doc-heavy, or config-heavy

The result is an honest, physical portrait of a codebase: the files that matter most cluster at the core; the forgotten ones float at the edge.

---

## Physics model

Three D3 force components run simultaneously:

| Force | Role |
|---|---|
| `forceRadial` | Sets each node's target orbital radius based on its gravity weight — heavier files orbit closer to the center |
| `forceManyBody` | N-body repulsion between all nodes; larger nodes repel more strongly, preventing visual clutter |
| `forceLink` | Spring bonds from each file to its parent directory; strength scales with normalized commit activity |
| `forceCollide` | Collision detection using each node's actual radius so nodes never overlap |

Gravity weight is computed from file size and path depth as a proxy for importance, distributed proportionally across total contributor commit counts. Per-file commit history is intentionally skipped to stay within GitHub's rate limits without authentication.

---

## Usage

### No install — open in browser

```bash
git clone https://github.com/your-username/github-gravity
open github-gravity.html
```

Or just download `github-gravity.html` and open it directly. No server, no build step, no dependencies to install.

### Authenticate for larger repos

The GitHub API allows **60 unauthenticated requests/hour**. For big repositories like `torvalds/linux` or `microsoft/vscode`, paste a [Personal Access Token](https://github.com/settings/tokens) (no scopes needed for public repos) into the token field. This raises the limit to **5,000 requests/hour**.

### Controls

| Action | Result |
|---|---|
| Type `owner/repo` + Enter | Load a repository |
| Scroll | Zoom in / out |
| Click + drag background | Pan |
| Click + drag a node | Reposition that node |
| Hover a node | Tooltip with file path, size, type, and commit weight |

---

## Rate limit strategy

The app makes exactly **3 API calls** per repository load:

1. `GET /repos/{owner}/{repo}` — fetch default branch name
2. `GET /repos/{owner}/{repo}/git/trees/{branch}?recursive=1` — full file tree in one shot (avoids paginated directory traversal)
3. `GET /repos/{owner}/{repo}/contributors?per_page=100` — up to 200 contributors across 2 pages

The remaining rate limit is displayed live in the header after every request. The file tree is capped at **300 nodes** to keep the simulation smooth even on low-end hardware.

---

## Color legend

| Color | File type |
|---|---|
| 🟢 Green | Source code (`.js`, `.py`, `.c`, `.rs`, `.go`, …) |
| 🟡 Yellow | Config & data (`.json`, `.yaml`, `.toml`, `.sh`, …) |
| 🔴 Red | Docs & markup (`.md`, `.rst`, `.tex`, …) |
| 🔵 Blue | Binary & media (`.png`, `.woff`, `.zip`, …) |
| 🟣 Purple | Directories |

---

## Technical stack

- **[D3.js v7](https://d3js.org/)** — force simulation, scales, zoom/pan, SVG rendering
- **GitHub REST API v3** — file tree and contributor data
- **Vanilla HTML/CSS/JS** — no framework, no bundler, no runtime dependencies
- Canvas-based starfield rendered independently via `requestAnimationFrame`

---

## Limitations

- **Private repos** require a token with `repo` scope
- **Very large repos** (50k+ files) will be truncated to the first 300 entries returned by the tree API; recursive trees over GitHub's size limit may return `truncated: true`
- Per-file commit history is approximated rather than fetched directly — precise per-file attribution would require one API call per file, which would exhaust rate limits instantly
- The simulation is CPU-bound; older devices may see frame drops with 300+ nodes

---

## License

MIT — do whatever you want with it.
