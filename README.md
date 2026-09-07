# synclife

A bash script that syncs directories between your laptop and a home server over Tailscale, using `rsync`.

## Features

- Two-way sync: `--push` (desktop → server) or `--pull` (server → desktop)
- Resolves the server's address dynamically via `tailscale ip`, so nothing is hardcoded
- Per-directory skip support (`--skip BOOKS`)
- Optional `--delete` to mirror deletions between source and destination
- One config block to map local ↔ remote directory pairs

## Requirements

- [rsync](https://rsync.samba.org/)
- [Tailscale](https://tailscale.com/) installed and connected, with your server reachable by its Tailscale hostname
- SSH access to the server (SSH key auth strongly recommended — see below)

## Setup

1. Copy `synclife.sh` somewhere in your `PATH` (e.g. `~/.local/bin/synclife`) and make it executable:
   ```bash
   chmod +x synclife
   ```
2. Open the script and edit the constants at the top:
   ```bash
   TAILSCALE_SERVER_NAME="zimaos"   # your server's Tailscale hostname
   SERVER_USERNAME="youruser"       # the SSH user on the server
   ```
   `SERVER_IP` is resolved automatically at runtime — you don't need to set it.
3. Edit the `HOME_DIRS` and `SERVER_DIRS` associative arrays so each key maps a local folder to its corresponding remote folder. **Every key must exist in both arrays** — the script checks for this and will refuse to run on a mismatch.
4. Set up SSH key auth to the server so the script doesn't prompt for a password every run:
   ```bash
   ssh-keygen -t ed25519          # if you don't already have a key
   ssh-copy-id youruser@<server>
   ```

## Usage

```
synclife --push/--pull [options]
```

| Option | Description |
|---|---|
| `--push` | Push local files to the server (default) |
| `--pull` | Pull files from the server to local |
| `--skip CONST` | Skip syncing the specified constant (e.g. `--skip BOOKS`). Repeatable. |
| `--delete` | Delete files on the destination that no longer exist on the source. **Use with caution.** |
| `-h`, `--help` | Show usage |

### Examples

```bash
synclife                              # push everything (default direction)
synclife --pull --skip BOOKS --skip MOVIES
synclife --pull --delete              # mirror server → laptop, removing local files not on server
```

## How directory pairs work

Each entry in `HOME_DIRS` has a matching key in `SERVER_DIRS`:

```bash
declare -A HOME_DIRS=(
    [BOOKS]="$HOME/Books"
    [MUSIC]="$HOME/Music/files_organized"
    ...
)

declare -A SERVER_DIRS=(
    [BOOKS]="$SERVER_HOME/Media/Books"
    [MUSIC]="$SERVER_HOME/Media/Music"
    ...
)
```

`--push` syncs `HOME_DIRS[KEY]` → `SERVER_DIRS[KEY]` for every key; `--pull` reverses the direction. Add, remove, or rename keys as needed — just keep both arrays in sync.

## Caution

- `--delete` is destructive: it removes files at the destination that aren't present at the source. Run once without `--delete` (or with `rsync --dry-run` manually) if you're not sure what will be affected.
- Directory names with spaces (e.g. `TV Shows`) are supported, but if you hit odd remote path errors, try adding `--protect-args` (`-s`) to the `rsync` calls in the script.
- The script assumes a strict one-to-one mapping between `HOME_DIRS` and `SERVER_DIRS` keys and will exit with an error if a key is missing from either side.

## License

MIT.
