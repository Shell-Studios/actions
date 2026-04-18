# actions

> Centralised CI/CD system for Minecraft add-on repositories in this organisation.  
> **One change here → every add-on repo gets the update automatically.**

---

## Architecture

```
shell-studios/actions  (this repo)
│
├── .github/workflows/
│   ├── release.yml          ← Reusable Workflow  – full release pipeline
│   └── test-dispatch.yml    ← Reusable Workflow  – manual test runner
│
└── actions/
    ├── compile-ts/          ← Composite Action – compile TypeScript (if present)
    ├── package-addon/       ← Composite Action – detect BP/RP and create .mcaddon/.mcpack
    ├── create-release/      ← Composite Action – publish GitHub Release
    └── notify-discord/      ← Composite Action – send Discord embed
```

Each add-on repo only needs **one file**:

```
my-addon-repo/
└── .github/workflows/release.yml   ← calls actions with uses: shell-studios/actions/...@v1
```

---

## How versioning works

```
actions tags:   v1  →  v1.0.0  →  v1.1.0  →  v2.0.0
add-on repos:    uses: shell-studios/actions/.github/workflows/release.yml@v1
                                                                         ^^^
                  Pinned to major version – gets all v1.x.x fixes automatically.
                  Breaking changes get a new major tag (v2), opt-in per repo.
```

### Tagging `actions` after a change

```bash
git tag v1.2.0
git tag -f v1          # move the floating major tag
git push origin v1.2.0 v1 --force
```

---

## Expected add-on folder structure

```
my-addon-repo/
├── behavior_pack/      (or BP/)
├── resource_pack/      (or RP/)
├── release_notes.md    ← shown in GitHub Release + Discord embed
└── .github/
    └── workflows/
        └── release.yml
```

| Folders present                   | Output file                 |
| --------------------------------- | --------------------------- |
| `behavior_pack` + `resource_pack` | `addon-name-v1.2.3.mcaddon` |
| `behavior_pack` only              | `addon-name-v1.2.3.mcpack`  |
| `resource_pack` only              | `addon-name-v1.2.3.mcpack`  |

TypeScript is compiled automatically if `tsconfig.json` exists at the root.

---

## Required secrets (set per add-on repo or as org-level secrets)

| Secret                 | Required | Description                         |
| ---------------------- | -------- | ----------------------------------- |
| `DISCORD_WEBHOOK`      | ✅       | Webhook for stable releases         |
| `DISCORD_WEBHOOK_BETA` | ✅       | Webhook for pre-release/beta builds |

---

## Release tag conventions

| Tag pattern      | Result                                       |
| ---------------- | -------------------------------------------- |
| `v1.2.3`         | Stable release, notifies `DISCORD_WEBHOOK`   |
| `v1.2.3-beta.1`  | Pre-release, notifies `DISCORD_WEBHOOK_BETA` |
| `v1.2.3-alpha.1` | Pre-release, notifies `DISCORD_WEBHOOK_BETA` |

---

## Using the system in an add-on repo

Copy `ADDON-REPO-TEMPLATE/.github/workflows/release.yml` into your add-on repo and edit two lines:

```yaml
with:
  addon-name: "My Awesome Addon" # ← your display name
```

That's it.

---

## Manual test dispatch

From any add-on repo → **Actions** → **Release** → **Run workflow** → choose a test mode:

| Mode           | What it does                                                     |
| -------------- | ---------------------------------------------------------------- |
| `compile-only` | Compiles TS, nothing else                                        |
| `package-only` | Compiles + packages, uploads artifact for inspection, no release |
| `notify-test`  | Sends a test embed to Discord without creating any release       |
| `full-dry-run` | Compiles + packages, no release, no notify                       |

---

## Adding a new composite action

1. Create `actions/my-new-action/action.yml`
2. Reference it inside reusable workflows as:
   ```yaml
   uses: shell-studios/actions/actions/my-new-action@v1
   ```
3. Tag a new patch/minor version of `actions`.

---

## Local development tips

- Test a single composite action by referencing a branch:  
  `uses: shell-studios/actions/actions/compile-ts@my-feature-branch`
- Once stable, merge to `main`, tag, and move the floating major tag.
