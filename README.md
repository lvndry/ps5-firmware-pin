# ps5-firmware-pin

Keep a jailbroken PS5 on the firmware you want, and get back to it if the
console updated anyway.

Tested on **firmware 13.60.007** with the **Relapse** jailbreak.

## Why

A PS5 jailbreak only works on specific firmware versions. When Sony ships a new
one, the console downloads it in the background and eventually shows an update
screen. Install it and the jailbreak is gone until someone ports the exploit.

This repo has three small Python scripts for that situation:

| You want to | Use | Details |
|---|---|---|
| Stop the console from updating | `pin/ps5-fw-pin.py` | [pin/README.md](pin/README.md) |
| Go back to an older firmware (e.g. 14.00 to 13.60) | `downgrade/ps5-fw-downgrade.py` | [downgrade/README.md](downgrade/README.md) |
| Allow updates again once the exploit supports them | `pin/ps5-fw-unpin.py` | [pin/README.md](pin/README.md#undo-the-pin) |

## Before you start

You need:

- **A computer with Python 3.8 or newer.** No packages to install. Check with
  `python3 --version`. On Windows, use `py` instead of `python3`.
- **This repo on that computer.** `git clone https://github.com/lvndry/ps5-firmware-pin`
  or use **Code > Download ZIP** on GitHub and unzip it. Run every command
  below from inside the `ps5-firmware-pin` folder.
- **The PS5 and the computer on the same network.**
- **Your PS5's IP address.** On the console: Settings > Network > Connection
  Status > View Connection Status > IPv4 Address. It looks like `192.168.1.42`.
  Every command below uses `192.168.1.42` as an example: replace it with yours.

The pin scripts also need the console **jailbroken right now, with the shsrv
payload loaded**. shsrv is the shell the scripts talk to, on port 2323. The
downgrade script runs on your computer only and does not need the jailbreak.

## Stop the console from updating

1. Jailbreak the console and load the shsrv payload.
2. On the computer, preview what the script will do (it changes nothing):

   ```bash
   python3 pin/ps5-fw-pin.py --host 192.168.1.42 --dry-run
   ```

3. Run it for real:

   ```bash
   python3 pin/ps5-fw-pin.py --host 192.168.1.42
   ```

   It reads the firmware version, deletes any update the console already
   downloaded, and checks nothing is left waiting to install. It stops without
   changing anything if the console is on 14.x.

4. On the console, turn off automatic downloads (once):
   Settings > System > System Software and Updates > Automatic Downloads >
   System Software and Apps > **Off**.

That's it. If an update prompt still appears, press **✕** to dismiss it.

Want the console to stop seeing updates at all? See
[`--full` mode](pin/README.md#--full-block-the-update-servers).

## Is it safe?

The short version: **yes.** The script deletes at most one downloaded file and,
in `--full` mode, appends two lines to a hosts file it backs up first. It never
rewrites firmware, never touches your games or saves, and every change can be
read back and verified.

**Can it brick my console?** No. Bricking a PS5 requires the console to fail
*mid-way through installing a firmware*, and this script exists precisely to
stop that from ever starting. What it actually does:

| Mode | Changes it makes |
|---|---|
| `--floor` (default) | Deletes `/update/PS5UPDATE.PUP` **if it exists**. That's it. |
| `--full` | Also backs up `/etc/hosts` to `hosts.fwpin.bak` and appends two lines pointing the update servers at `127.0.0.1` |

That's the complete set of file operations. It does not write firmware, does
not flash anything, and does not restart the console. If the script dies or
you lose power halfway, the worst state left behind is "a pending update file
still exists" — exactly the state you started from.

**Does it modify the firmware?** No. It modifies *update files and one hosts
file*. The firmware that's installed stays byte-for-byte the same. After
running the pin, check Settings > System > System Software: the version number
is unchanged.

**What actually happens if I get it wrong?** The worst realistic outcome: the
console installs an update anyway (you accidentally confirmed the prompt). That
is **not** a brick:

- The console still works. Games, saves, everything is intact.
- You just lose the jailbreak until your exploit supports the new firmware.
- If you're *stuck* on a firmware the exploit doesn't support, the
  [downgrade script](downgrade/README.md) puts you back on the older one from
  a USB stick. Games and saves survive the downgrade.

So the failure mode is "jailbreak temporarily gone", never "console broken".

**Are my games and saves safe?** Yes. The script only reads the firmware
version, lists `/update`, and deletes the update file if present. Game data
lives in completely different folders that it never opens.

**How do I know it actually worked?** The script verifies its own work and
tells you:

- After deleting the update, it re-checks that `/update` is empty and prints
  `/update clean: True`.
- In `--full` mode it re-reads the hosts file after writing and prints
  `pinned update hosts` — or tells you plainly if the write didn't stick.
- If the console is already on a firmware it can't protect (14.x), it stops
  **without changing anything** and tells you why.
- `--dry-run` shows you every change before a single one is made.

Run `--dry-run` first. It's the honest preview.

**Does `--full` affect normal use? Online games?** Only two hostnames are
blocked: `update.playstation.com` and `dl.playstation.com`. Those exist to
deliver firmware and app updates. Playing games, PSN chat, trophies, and
multiplayer don't go through them.

If it ever bothers you, `ps5-fw-unpin.py` restores the hosts file from the
backup in one command.

**Will it survive a reboot?** The *protection* does, yes: once the update file
is deleted and Automatic Downloads is off, there's nothing to install. The
`--full` hosts lines are written to the console's real `/etc/hosts`, which
persists across reboots.

The *scripts* don't need to run again unless a new update gets downloaded
(for example, because Automatic Downloads was turned back on). Re-running is
always safe — a second run finds nothing to delete and does nothing.

**Is my warranty / ToS affected?** The pin itself doesn't add anything to the
equation: you're already running a jailbreak, and that's what voids the
warranty, not a deleted file. Nothing here makes a console *more* modified than
it already is.

**How do I undo it?**

```bash
python3 pin/ps5-fw-unpin.py --host 192.168.1.42
```

Restores `/etc/hosts` from the backup if `--full` was used. Then flip
Automatic Downloads back on in Settings. After a `--floor`-only pin there is
literally nothing left on the console to undo.

**Why should I trust a small script from a GitHub repo?** Because there is not
much to trust. Read it: [`ps5-fw-pin.py`](pin/ps5-fw-pin.py) is ~150 lines with
no dependencies, and the whole of what it can write to your console is
visible in the source:

- `rm /update/PS5UPDATE.PUP`
- `cp /etc/hosts /etc/hosts.fwpin.bak` (once)
- `printf ... >> /etc/hosts` (two `127.0.0.1` lines)

Nothing else. Run `--dry-run` to see exactly what it would do on *your*
console before letting it.
## The one rule

**Never press confirm on an update screen.** Press ✕ to dismiss it. If you are
stuck on a screen that won't let you back out, hold the console's power button
for 10 seconds to turn it off.

## Already updated? Go back

If the console is on a firmware the jailbreak doesn't support, you can install
an older one from a USB stick:

```bash
python3 downgrade/ps5-fw-downgrade.py --url <link to PS5UPDATE.PUP>
```

Sony only links the latest firmware, so the older file comes from a PS5
homebrew mirror. The script downloads it (about 1 GB) and checks it's real.
Then copy it to a USB stick and install it from the console. The full
walkthrough is in [downgrade/README.md](downgrade/README.md). Your games and
saves stay; the jailbreak has to be run again afterwards.

## Troubleshooting

| You see | What it means |
|---|---|
| `error: the following arguments are required: --host` | Add `--host` followed by your PS5's IP address |
| `cannot reach shsrv at ...` | The console isn't jailbroken right now, shsrv isn't loaded, or the IP is wrong |
| `STOP: this console is on 14.x` | The console already updated. See [Already updated? Go back](#already-updated-go-back) |
| `firmware: unknown` | The script couldn't read the version. Check it in Settings > System > System Software before trusting the result |
| `python3: command not found` | Install Python from python.org, or use `py` on Windows |

## Disclaimer

Use at your own risk. Modifying console firmware and running homebrew may void
your warranty and break Sony's terms of service. Not affiliated with Sony.

## License

[MIT](LICENSE)
