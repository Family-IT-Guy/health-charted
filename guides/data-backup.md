# Data Backup

This system stores potentially years of health history in local files. That data is irreplaceable. Protecting it is a core responsibility of the system.

## Two layers of protection

### Layer 1: Automatic versioning (built in)

Health Charted uses git to keep a history of every change to your data files. This is invisible to you. You don't need to know what git is or how it works. The system manages it automatically.

This protects against: a file being accidentally overwritten, corrupted, or incorrectly modified. If something goes wrong with a write, the system can restore the previous version instantly.

This does NOT protect against: your computer breaking, being lost, stolen, or damaged by water or fire. If your computer is gone, the version history is gone with it.

### Layer 2: External backup (you set this up)

You need a backup system that copies your files to a second location. This protects against hardware failure, theft, and physical damage.

**Important:** your backup must keep file history (the ability to go back to older versions of a file), not just a copy of the current files. If a file gets corrupted and your backup copies the corrupted version, the backup is also corrupted.

**Good backup options:**
- **Mac: Time Machine.** Built into your Mac. Plug in an external drive, your Mac asks if you want to use it for Time Machine. Say yes. It keeps hourly backups and lets you restore any file to any previous version.
- **Windows: File History.** Built into Windows. Go to Settings > Update & Security > Backup. Connect an external drive and turn it on.
- **Any platform: Backblaze.** Cloud backup service that keeps 1 year of file history. Runs in the background. About $9/month.

**Not sufficient on their own:**
- Dropbox, iCloud, Google Drive, OneDrive file sync. These copy the current version of your files to the cloud. If a file gets corrupted, the corrupted version gets synced. Some of these services do keep version history, but the retention period is limited and it's not their primary purpose. Use them in addition to a proper backup, not instead of one.

**After setting up backup:** test it. Open your backup software and restore a file. A backup you've never tested is not a backup you can trust.

## When to surface

- During onboarding: mention that backup is important and offer to walk through setup
- After significant data additions (first lab results, first research session)
- Periodically (every few months, check if backup is still running)

## Git initialization

On first run, if the Health Charted folder does not contain a `.git` directory, initialize it:

```
git init
git add -A
git commit -m "initial health charted setup"
```

After every data file write that passes verification, commit the change:

```
git add <changed-file>
git commit -m "<descriptive message about what changed>"
```

Commit messages should be plain language descriptions of what changed: "update symptoms: added entry 2026-04-02 hand pain severity 2" or "update profile: added Dr. Smith as hand specialist."

Never mention git to the user. This is invisible infrastructure.
