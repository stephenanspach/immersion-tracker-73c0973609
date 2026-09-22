# Immersion Tracker

A static dashboard for tracking Japanese immersion. This repository holds only the
**published output** — a single self-contained `index.html` with all data baked in at
generation time (no JavaScript storage, no network calls at runtime).

The generator script and the source data live outside this repository, in iCloud Drive.

## Viewing it

**https://stephenanspach.github.io/immersion-tracker-73c0973609/**

Served by GitHub Pages from the root of the `main` branch. Nothing needs to be installed
to read the dashboard — any browser on any device works, including phones. Pushing to
`main` triggers a Pages rebuild automatically; the new version is live within a minute.

The repository is **public**. `robots.txt` and the `noindex` meta tag keep it out of
search engines, but anyone who has the URL can open it. Keep that in mind before adding
anything sensitive to the tracked data.

## Repository contents

| File | Purpose |
| --- | --- |
| `index.html` | The entire dashboard — markup, styles, data, and sort/filter script |
| `robots.txt` | `Disallow: /` — keeps crawlers out |
| `.nojekyll` | Tells GitHub Pages to serve files as-is, skipping Jekyll |
| `README.md` | This file |

## Setting up a second machine

iCloud syncs the data and the generator, but a working setup needs several things iCloud
does **not** carry across. Work through these on the new Mac.

### 1. Clone the repository outside iCloud Drive

```sh
git clone https://github.com/stephenanspach/immersion-tracker-73c0973609.git ~/code/immersion-tracker
```

Do not put the clone inside iCloud Drive. iCloud can evict or conflict-copy files inside
`.git/`, which corrupts the repository. Each machine gets its own clone in local storage;
only the data and generator folders belong in iCloud.

### 2. Set up GitHub authentication

Keychain entries and `~/.ssh` keys do not sync through iCloud Drive, so pushing will fail
on a fresh machine until this is done. Either:

- **SSH** — generate a key with `ssh-keygen -t ed25519`, add the public key at
  https://github.com/settings/keys, then point the clone at the SSH remote:
  `git remote set-url origin git@github.com:stephenanspach/immersion-tracker-73c0973609.git`
- **HTTPS** — install the `gh` CLI and run `gh auth login`, or create a personal access
  token with `repo` scope and let the macOS credential helper store it on first push.

Verify with `git push` on a trivial commit before relying on it.

### 3. Install the generator's runtime

<!-- TODO: fill in for the actual generator -->
The generator is not checked into this repository. Record here what it needs so the next
setup is mechanical:

- **Location in iCloud:** `~/Library/Mobile Documents/com~apple~CloudDocs/<path/to/generator>`
- **Language and version:** `<e.g. Python 3.12>`
- **Dependencies:** `<e.g. pip install -r requirements.txt, or "none — stdlib only">`
- **Command to regenerate:** `<e.g. python3 build.py>`
- **Where it writes output:** the clone's `index.html`

### 4. Check for hardcoded paths

If the generator has an absolute path baked in — typically
`/Users/<shortname>/Library/Mobile Documents/com~apple~CloudDocs/...` — it breaks when the
two Macs have different account short names. Check with:

```sh
id -un   # run on both machines and compare
```

If they differ, switch the script to build paths from `$HOME` rather than hardcoding
`/Users/<shortname>`.

### 5. Make sure iCloud files are actually downloaded

With **Optimize Mac Storage** enabled, iCloud can leave dataless placeholders in place of
real files, which scripts fail on. In Finder, right-click the tracker's data folder and
choose *Download Now*, or *Keep Downloaded* to pin it permanently.

### 6. Recreate any automation

`launchd` jobs, `cron` entries, Shortcuts, and text-expander or macro triggers are all
per-machine. If regeneration is scheduled on the first Mac, set it up again here — or
deliberately leave it running on one machine only.

## Publishing a change

```sh
cd ~/code/immersion-tracker
git pull                      # always pull first — the other Mac may have pushed
<run the generator>           # rewrites index.html
git add index.html
git commit -m "Update dashboard $(date '+%Y-%m-%d %H:%M:%S')"
git push
```

Pages redeploys on push. Check the run at
https://github.com/stephenanspach/immersion-tracker-73c0973609/actions if the live site
looks stale.

## Things that bite

- **Editing the data from both Macs at once** produces iCloud conflict copies
  (`filename 2.ext`), which silently drop edits. Let sync settle — the iCloud Drive status
  in Finder's sidebar should be idle — before switching machines.
- **Forgetting to `git pull`** before regenerating means a push conflict on a file that
  can't be merged sensibly. `index.html` is generated output: on a conflict, take either
  side, regenerate, and commit the fresh result rather than hand-merging.
- **Editing `index.html` directly** loses the change on the next regeneration. Edit the
  generator or the source data instead.
