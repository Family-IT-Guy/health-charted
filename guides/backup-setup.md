# Backup Setup

Current recommended tools for the two layers of data backup. See `.claude/rules/data-backup.md` for the principles these tools implement.

## Layer 1: Automatic versioning (current implementation: git)

The system uses git to keep a history of every change to data files. This is invisible to the user.

### Initialization

On first run, if the Health Charted folder does not contain a `.git` directory, initialize it:

```
git init
git add -A
git commit -m "initial health charted setup"
```

### After data file writes

After every data file write that passes verification, commit the change:

```
git add <changed-file>
git commit -m "<descriptive message about what changed>"
```

Commit messages should be plain language descriptions of what changed: "update symptoms: added entry 2026-04-02 hand pain severity 2" or "update profile: added Dr. Smith as hand specialist."

Never mention git to the user. This is invisible infrastructure.

## Layer 2: External backup (current recommended tools)

The user must set this up. The system surfaces the need; the user chooses and configures.

### Mac: Time Machine

Built into macOS. Plug in an external drive, the OS prompts to use it for Time Machine. Accept. Time Machine keeps hourly backups and allows restoring any file to any previous version.

### Windows: File History

Built into Windows. Settings > Update & Security > Backup. Connect an external drive and turn on File History.

### Any platform: Backblaze

Cloud backup service. Keeps 1 year of file history. Runs in the background. Approximately $9/month at time of writing — confirm current pricing at backblaze.com.

### Not sufficient on their own

Dropbox, iCloud, Google Drive, OneDrive file sync — these copy the current version to the cloud. If a file gets corrupted, the corrupted version syncs. Some have version history but with limited retention; not designed as backup. Use alongside a proper backup, not instead of one.

## Testing

After setting up backup, test it. Open the backup software and restore a file. A backup that has never been tested is not trustworthy.
