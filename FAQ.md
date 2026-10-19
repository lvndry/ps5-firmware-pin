# FAQ

The short version: **yes, it's safe.** The script deletes at most one
downloaded file and, in `--full` mode, appends two lines to a hosts file it
backs up first. It never rewrites firmware, never touches your games or
saves, and every change can be read back and verified.

## Is it safe? Can it brick my console?

No. Bricking a PS5 requires the console to fail *mid-way through installing a
firmware*, and this script exists precisely to stop that from ever starting.
What it actually does:

| Mode | Changes it makes |
|---|---|
| `--floor` (default) | Deletes `/update/PS5UPDATE.PUP` **if it exists**. That's it. |
| `--full` | Also backs up `/etc/hosts` to `hosts.fwpin.bak` and appends two lines pointing the update servers at `127.0.0.1` |

That's the complete set of file operations. It does not write firmware, does
not flash anything, and does not restart the console. If the script dies or
you lose power halfway, the worst state left behind is "a pending update file
still exists" — exactly the state you started from.

## Does it modify the firmware?

No. It modifies *update files and one hosts file*. The firmware that's
installed stays byte-for-byte the same. After running the pin, check
Settings > System > System Software: the version number is unchanged.

## What actually happens if I get it wrong?

The worst realistic outcome: the console installs an update anyway (you
accidentally confirmed the prompt). That is **not** a brick:

- The console still works. Games, saves, everything is intact.
- You just lose the jailbreak until your exploit supports the new firmware.
- If you're *stuck* on a firmware the exploit doesn't support, the
  [downgrade script](downgrade/README.md) puts you back on the older one from
  a USB stick. Games and saves survive the downgrade.

So the failure mode is "jailbreak temporarily gone", never "console broken".

## Are my games and saves safe?

Yes. The script only reads the firmware version, lists `/update`, and deletes
the update file if present. Game data lives in completely different folders
that it never opens.

## How do I know it actually worked?

The script verifies its own work and tells you:

- After deleting the update, it re-checks that `/update` is empty and prints
  `/update clean: True`.
- In `--full` mode it re-reads the hosts file after writing and prints
  `pinned update hosts` — or tells you plainly if the write didn't stick.
- If the console is already on a firmware it can't protect (14.x), it stops
  **without changing anything** and tells you why.
- `--dry-run` shows you every change before a single one is made.

Run `--dry-run` first. It's the honest preview.

## Does `--full` affect normal use? Online games?

Only two hostnames are blocked: `update.playstation.com` and
`dl.playstation.com`. Those exist to deliver firmware and app updates.
Playing games, PSN chat, trophies, and multiplayer don't go through them.

If it ever bothers you, `ps5-fw-unpin.py` restores the hosts file from the
backup in one command.

## Will it survive a reboot?

The *protection* does, yes: once the update file is deleted and Automatic
Downloads is off, there's nothing to install. The `--full` hosts lines are
written to the console's real `/etc/hosts`, which persists across reboots.

The *scripts* don't need to run again unless a new update gets downloaded
(for example, because Automatic Downloads was turned back on). Re-running is
always safe — a second run finds nothing to delete and does nothing.

## Is my warranty / ToS affected?

The pin itself doesn't add anything to the equation: you're already running a
jailbreak, and that's what voids the warranty, not a deleted file. Nothing
here makes a console *more* modified than it already is.

## How do I undo it?

```bash
python3 pin/ps5-fw-unpin.py --host 192.168.1.42
```

Restores `/etc/hosts` from the backup if `--full` was used. Then flip
Automatic Downloads back on in Settings. After a `--floor`-only pin there is
literally nothing left on the console to undo.

## Why should I trust a small script from a GitHub repo?

Because there is not much to trust. Read it: [`ps5-fw-pin.py`](pin/ps5-fw-pin.py)
is ~150 lines with no dependencies, and the whole of what it can write to
your console is visible in the source:

- `rm /update/PS5UPDATE.PUP`
- `cp /etc/hosts /etc/hosts.fwpin.bak` (once)
- `printf ... >> /etc/hosts` (two `127.0.0.1` lines)

Nothing else. Run `--dry-run` to see exactly what it would do on *your*
console before letting it.
