# Central releases on the CAL GitHub Pages site

One public repo (**`stafne/cal`**) can list every app **and** host all macOS installers and version tags. You do **not** need separate `hp_sleep_mac`, `hp_merge_mac`, … Pages repos unless you want them for other reasons.

Private **source** stays in **`hp_py_<app>`** repos. Only **binaries and the splash** live in **`cal`**.

---

## Architecture

```text
hp_py_sleep (private)  ──release script──►  stafne/cal  Releases
hp_py_merge (private)  ──release script──►     tag: hp_merge-v1.0.5
hp_py_controller …     ──release script──►     assets: hp_py_merge.dmg, .zip, …

https://stafne.github.io/cal/  (Pages from cal repo)
    ├── index.html          ← app cards + Download DMG/ZIP
    ├── icons/*.png
    └── (optional) releases.json   ← manifest written by release scripts
```

| Piece | Repo | Visibility |
|-------|------|------------|
| Source | `hp_py_<app>` | Private |
| Downloads + versions | **`cal`** | Public |
| Lab splash | **`cal`** (same repo) | Public |

---

## Tag and asset naming (required for per-app versions)

GitHub allows **many releases** in one repo. Each app needs its **own tag prefix** so versions do not collide.

| App slug | Release tag example | Typical DMG asset |
|----------|---------------------|-------------------|
| sleep | `hp_sleep-v1.0.81` | `hp_py_sleep_mac.dmg` |
| merge | `hp_merge-v1.0.5` | `hp_py_merge.dmg` |
| controller | `hp_controller-v1.0.3` | `hp_py_controller.dmg` |
| attention | `hp_attention-v1.0.2` | `hp_py_attention_mac.dmg` |
| anesthesia | `hp_anesthesia-v1.0.0` | (match your PyInstaller output name) |
| icu | `hp_icu-v1.0.0` | (match your PyInstaller output name) |

**Rules:**

- Tag format: **`hp_<slug>-v<major>.<minor>.<build>`** (same version string you already use in `version.py`).
- Asset names stay **stable** per app (same basename every release) so `cal/index.html` can link predictably.
- Download URL pattern:  
  `https://github.com/stafne/cal/releases/download/<tag>/<asset>.dmg`

`releases/latest` on a repo points to **one** release only, so the splash page uses **per-app tags** (via the Releases API or `releases.json`), not a single shared `latest` for everything.

- **Cleanup:** each app’s `release_app_complete.py` must prune only releases whose `tag_name` starts with that app’s `RELEASE_TAG_PREFIX-`. Never delete unrelated tags on `cal`.

---

## Release script changes (per app)

In each app’s `release_app_complete.py`, point uploads at **`cal`** instead of `hp_<app>_mac`:

```python
REPO_OWNER = "stafne"
REPO_NAME = "cal"                    # was hp_sleep_mac, etc.
RELEASE_TAG_PREFIX = "hp_sleep"      # tag becomes hp_sleep-v1.0.81
```

When creating the GitHub Release:

- `tag_name` = `f"{RELEASE_TAG_PREFIX}-{get_version_string()}"`  → `hp_sleep-v1.0.81`
- Upload `dist/<APP_NAME>.dmg` and `.zip` as release assets (names unchanged).

Source commits still push to the **private** `hp_py_<app>` repo; only the **release API target** changes to `cal`.

**In-app auto-updater:** set `GITHUB_REPO = "stafne/cal"` and resolve the newest release whose `tag_name` starts with `hp_sleep-`, then download the `.zip` asset (same basename as today).

---

## Optional manifest: `cal/releases.json`

Release scripts can also write a small JSON file committed to **`cal`**:

```json
{
  "sleep": {
    "tag": "hp_sleep-v1.0.81",
    "dmg": "hp_py_sleep_mac.dmg",
    "zip": "hp_py_sleep_mac.zip",
    "published": "2025-05-14"
  }
}
```

`index.html` can read **`releases.json`** first (fast, no API rate limits), then fall back to the GitHub API.

---

## What you can drop

After all apps upload to **`cal`**:

- Separate **`hp_<app>_mac`** GitHub Pages sites (optional legacy).
- Per-app `index.html` download pages, unless you still want deep install docs there.

Keep **`hp_<app>_mac`** repos only if you want separate issue trackers or old links; they are not required for versioning.

---

## Migration checklist

1. Enable Pages on **`cal`** (already at `stafne.github.io/cal/`).
2. Pick tag prefix + asset names per app (table above).
3. Change each **`release_app_complete.py`** → `REPO_NAME = "cal"` + prefixed tags.
4. Run one release per app; confirm assets under  
   `https://github.com/stafne/cal/releases`.
5. Push updated **`cal/index.html`** (central download buttons).
6. Update each app’s **`auto_updater.py`** to use `stafne/cal` + tag prefix.
7. (Optional) Retire `hp_*_mac` Pages or leave redirects in those `index.html` files pointing to `/cal`.

---

## References

- **`cal/index.html`** — hub UI and JavaScript for per-app releases on `stafne/cal`.
- **`github_versioning.md`** — PyInstaller release flow (change upload target to `cal`).
- **`update_cal_site.py`** — refreshes `cal/icons/` from `icons/<app>.icns`.
