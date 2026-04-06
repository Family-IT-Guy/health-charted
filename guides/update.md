# Update Guide

How to update Health Charted when a new version is available. The user should never see technical details. No mention of git, GitHub, repositories, downloads, manifests, or file paths.

## Detection

During session initialization, CLAUDE.md instructs you to compare the local `VERSION` file against the remote version at:

```
https://raw.githubusercontent.com/Family-IT-Guy/health-charted/main/VERSION
```

If the remote version is newer, proceed to the offer step. If the fetch fails, skip silently.

## Offer

Fetch the remote CHANGELOG.md:

```
https://raw.githubusercontent.com/Family-IT-Guy/health-charted/main/CHANGELOG.md
```

Read the entry for the new version. Summarize it in 1-2 plain-language sentences. Present to the user:

> "A newer version of Health Charted is available. [Summary of what changed, in terms the user cares about — not technical details]. Want me to update? I'll back up your data first."

If the user says no or not now, continue with the current version. Do not ask again this session.

## Update procedure

If the user approves:

### Step 1: Ensure git is ready

Check if git is initialized in the project directory. If not, initialize it:

```bash
git init
git add -A
git commit -m "initial backup"
```

### Step 2: Back up current state

Commit all current files (including personal data) to create a restore point:

```bash
git add -A
git commit -m "backup before update to [new version]"
```

Tell the user: "Backing up your data... done."

### Step 3: Fetch the remote manifest

Download the MANIFEST from the NEW version (not the local one):

```
https://raw.githubusercontent.com/Family-IT-Guy/health-charted/main/MANIFEST
```

Parse it: each non-empty line that doesn't start with `#` is a file path to update.

### Step 4: Download all system files

Create a temp directory. For each file in the remote manifest, download from:

```
https://raw.githubusercontent.com/Family-IT-Guy/health-charted/main/[path]
```

Save to the temp directory preserving the path structure.

**If any download fails:** delete the temp directory, tell the user "Something went wrong during the update. Your data is safe and nothing was changed. We can try again next time." Continue with the current version. Do not attempt partial updates.

### Step 5: Replace system files

Copy every downloaded file from the temp directory to the project directory, overwriting the existing system files. Create any directories that don't exist yet (e.g., if a new guide is added in a future version).

Delete the temp directory.

### Step 6: Verify

Read the new `VERSION` file. Confirm it matches the expected new version.

Tell the user: "Updated to version [X.Y.Z]. [1-2 sentence summary of what changed]. Let's continue."

### Step 7: Continue session

Re-read the updated CLAUDE.md and guides. The new version's instructions take effect immediately for the rest of this session.

## Rollback

If anything goes wrong after files were replaced (verification fails, system behaves unexpectedly), restore from the git backup:

```bash
git checkout HEAD~1 -- .
```

Tell the user: "Something went wrong with the update. I've restored your previous version. Your data is safe."

## What is never overwritten

The MANIFEST lists only system files. Everything else is personal data:

- `my-data/` — all files (health profile, labs, symptoms, treatments, etc.)
- `preferences.json` — user preferences
- `reference/` — INDEX.json and all reference files the user has built
- `research/` — INDEX.json and all research files

These files are never listed in the MANIFEST and are never touched during an update. Schema changes in new versions are applied lazily: the next time the LLM writes to a data file, it follows the updated schema and adds any new fields with null values.

## Language rules

During the entire update process, the user should see only:

- "A newer version is available. [What changed]. Want me to update?"
- "Backing up your data... done."
- "Downloading the update... done."
- "Updated to version [X]. [What changed]. Let's continue."
- Or: "Something went wrong. Your data is safe. We can try again next time."

Never mention: git, GitHub, repositories, pull, push, commit, stash, manifest, curl, download URLs, file paths, schemas, migration, temp directories, or any technical infrastructure.
