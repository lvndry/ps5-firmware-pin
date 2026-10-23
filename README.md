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
| Move to a **specific** newer firmware (the Update button only offers the latest) | `PS5/UPDATE/` on a USB stick | [upgrade/README.md](upgrade/README.md) |
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

**Yes.** The script deletes at most one update file and, in `--full` mode,
appends two lines to a hosts file it backs up first. It never writes firmware,
never touches games or saves, and verifies every change before it reports.

**Can it brick the console?** No. Bricking needs a failed *install*, and this
script only stops one from starting. Its complete set of writes:

- `rm /update/PS5UPDATE.PUP` (if present)
- `cp /etc/hosts /etc/hosts.fwpin.bak` (once, `--full`)
- two `127.0.0.1` lines appended to `/etc/hosts` (`--full`)

If it crashes or you lose power halfway, the worst state left is "a pending
update file exists" — the state you started from.

**Worst realistic case?** You accidentally confirm an update: the jailbreak
is gone until your exploit supports the new firmware. Console, games and saves
are untouched; the [downgrade script](downgrade/README.md) gets you back if
you're stuck.

**Does `--full` affect online play?** No — it blocks only the two update
hostnames, not game traffic.

**How do I undo it?** `python3 pin/ps5-fw-unpin.py --host 192.168.1.42`
restores the hosts backup. After a `--floor`-only pin there's nothing to undo.

**Still not sure?** Read the source: ~150 lines, no dependencies
([`ps5-fw-pin.py`](pin/ps5-fw-pin.py)), and `--dry-run` previews every change
before one is made.

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
