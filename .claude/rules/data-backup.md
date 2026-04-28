# Data Backup

This system stores potentially years of health history in local files. That data is irreplaceable. Protecting it is a core responsibility of the system.

## Two layers of protection

### Layer 1: Automatic versioning (built in)

The system maintains a version history of every change to data files. This is invisible to the user — they don't need to know how it works. The system manages it automatically per the procedure in `guides/backup-setup.md`.

This protects against: a file being accidentally overwritten, corrupted, or incorrectly modified. If something goes wrong with a write, the system can restore the previous version instantly.

This does NOT protect against: the user's computer breaking, being lost, stolen, or damaged by water or fire. If the computer is gone, the version history is gone with it.

### Layer 2: External backup (user sets up)

The user needs a backup system that copies their files to a second location. This protects against hardware failure, theft, and physical damage.

The principle: external backup with version history is required. Backups that copy only the current version are insufficient — if a file gets corrupted, the corrupted version syncs and overwrites the good one. Sync-only services don't qualify on their own.

For current recommended backup tools and setup steps, see `guides/backup-setup.md`.

## When to surface

- During onboarding: mention that backup is important and offer to walk through setup
- After significant data additions (first lab results, first research session)
- Periodically (every few months, check if backup is still running)

## After setup

Test the backup. A backup that has never been tested is not a backup that can be trusted. Restoring a file from the backup verifies the system works end-to-end.
