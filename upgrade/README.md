# upgrade

Install a **specific** PS5 firmware from a USB stick.

The "Update" button in Settings only ever offers the *latest* version. If you
want a particular one — for example 13.60 → 14.10 while 14.20 is out — you
put that firmware's file on a stick. Any official PUP installs in the
newer-than-current direction, as long as the [keyset](../downgrade/README.md)
is still active.

## Step 1: Get the firmware file

Each firmware is one file named `PS5UPDATE.PUP`.

- **Latest version:** Sony's official
  [PS5 system software page](https://www.playstation.com/en-us/ps5/system-software/)
  serves it directly, no account needed.
- **An older version than the latest:** Sony only hosts the newest one, so
  get it from a homebrew mirror (e.g. PSXHAX) and prefer a link that
  publishes an MD5 checksum.

Then verify it before trusting it:

```bash
python3 downgrade/ps5-fw-downgrade.py --verify /path/to/PS5UPDATE.PUP --md5
```

The same script downloads a PUP for you if you have a direct mirror link:

```bash
python3 downgrade/ps5-fw-downgrade.py --url https://mirror.example/PS5UPDATE.PUP
```

A real PUP is **about 1 GB**. A file of a few KB is an error page.

## Step 2: Prepare the USB stick

1. Format the stick as **exFAT** (FAT32 also works).
2. Put the file here, at the root of the stick:

   ```
   PS5/
     UPDATE/
       PS5UPDATE.PUP
   ```

   Exact folder names, exact file name. A truncated copy (common with slow
   mirrors) gives the error `SU-101269-9`.

## Step 3: Install

1. Plug the stick into the console.
2. Settings → System → System Software and Updates →
   **Install Update from USB Drive**.
3. Confirm the prompt, don't touch the console for ~10 minutes.
4. It reboots on its own into the new version.

## What changes

- ✅ games and saves survive
- ❌ the **jailbreak does not** — you re-run your exploit afterwards, and
  ShadowMount re-mounts your game dumps (wait ~30 s after jailbreaking
  before launching a homebrew game)
- As of October 2026: **13.60 is the newest fully jailbreakable firmware**;
  14.00–14.20 have no public payload chain yet. Upgrading there means no
  homebrew until one ships.

## Can I go back afterwards?

Plain USB downgrades from 14.x are currently flaky (see
[issue #10](https://github.com/lvndry/ps5-firmware-pin/issues/10)); the
[Testkit method](https://github.com/lvndry/ps5-firmware-pin/issues/7) is the
workaround. Upgrades are not affected.
