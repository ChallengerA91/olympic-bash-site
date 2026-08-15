# Olympic Bash — website

The download page for OlympicBash.com. A single HTML file, no build step —
same "double-click and it works" philosophy as the game itself. This is a
**separate project from the game repo on purpose**, so nothing here ever
collides with work happening in `Olympic Bash/`.

```
index.html          the whole site (nav, hero, download button, footer)
version.json         what "the server" is — see below
downloads/            put the actual installer/zip here before you deploy
```

## See it on your own computer

Double-click `index.html`. It opens in your browser exactly as it will look
live — except the "Download" button won't know the real version number yet
(browsers block a local file from reading another local file for security
reasons). That part only works once it's actually hosted, which is normal.

## What "connects to the server" means here

There's no real server, no database, nothing to maintain. `version.json`
sits right next to `index.html` and looks like this:

```json
{
  "version": "0.1.0",
  "url": "https://olympicbash.com/downloads/OlympicBashSetup.exe",
  "update_zip": "https://olympicbash.com/downloads/OlympicBash.zip",
  "released": "Aug 15, 2026",
  "notes": "Playtest build — update this file each time you ship a new build."
}
```

The page reads that file the moment someone loads it, so the Download
button always points at whatever's current — nobody ever downloads a stale
build by accident. `version.version` is a bare number (no "v" prefix —
the page and the game both add that themselves when they display it).

**`url` and `update_zip` serve two different consumers, on purpose.**
`url` is what the website's Download button uses — the installer, meant for
a human to run. `update_zip` is what the game's own in-game auto-updater
fetches — it downloads that file and unzips it itself, so it needs an
actual zip, not something that runs an installer wizard. **Both must be
kept present and pointing at real files for every release**, or the
in-game "UPDATE NOW" button breaks with "the downloaded update looked
broken" — this happened once already (Aug 15 2026) when `url` was
switched to the installer without adding `update_zip`, and every player
who hit the mandatory update gate got stuck. `build.cmd` produces both
`dist\OlympicBashSetup.exe` and `dist\OlympicBash.zip` from the same
build — upload both.

**As of Aug 15 2026, this file matters more than it used to.** The game now
checks it on every launch and, if a build is exported with `build.cmd`
(which stamps a real version number into it — see the game repo's `VERSION`
file), a mismatch here **blocks play entirely** until the player updates —
not a soft "new version available" badge anymore. That means:

- **Always update `version.json` and re-upload the matching installer to
  `downloads/` in the SAME commit.** Bumping the number here without a
  matching build locks out anyone who updates to "the new version" and gets
  handed the old build — they'd never stop being told to update.
- A build exported from source that was never given a bumped `VERSION`
  first will just re-claim the same number as last time, which is safe
  (no mismatch, no false block) — but also means players won't be offered
  it as an update. Bump `VERSION` in the game repo before running
  `build.cmd` for anything meant to actually reach players.
- This file only matters to players running an **exported** build. Playing
  from source (`godot --path godot`) skips the check entirely — there's no
  meaningful "install directory" to protect there.

## Publishing a new build

1. Drop the new build in `downloads/` — **both files**, `OlympicBashSetup.exe`
   AND `OlympicBash.zip`. `build.cmd` produces both from the same build;
   the installer is for the Download button, the zip is for the in-game
   auto-updater (see the `update_zip` note above — skipping this half
   breaks in-game updates, not the website).
2. Edit `version.json`: bump `version`, update `url`/`update_zip` if either
   filename changed, set `released` to today's date.
3. Commit and push. GitHub Pages redeploys automatically within a minute
   or two of a push to `main` — nothing else to trigger.

That's the entire release process. No code changes needed for a new build.

## Where this actually lives

Hosted on **GitHub Pages**, on the account already logged in on this
machine (`ChallengerA91`) — no new account, no cost, no separate host to
manage:

- Repo: https://github.com/ChallengerA91/olympic-bash-site
- Live now at: https://challengera91.github.io/olympic-bash-site/
- Configured to serve at `olympicbash.com` once DNS points there (see below)

Every `git push` to `main` redeploys it. GitHub Pages has no meaningful
file-size limit for a site this size, so the game build can live in
`downloads/` for now. If a build ever gets huge (several hundred MB+), a
game-file host like itch.io is a better fit than baking it into the repo
— worth revisiting once you know the real file size, not before.

## Going live at OlympicBash.com: one step left

Everything on GitHub's side is already configured — the repo has a
`CNAME` file for `olympicbash.com` and Pages is set to expect it. The only
thing left needs your Namecheap login, which I don't have and shouldn't:

In Namecheap: **Domain List → Manage → Advanced DNS**, and add these
records (delete any existing `A` or `CNAME` records on `@` first so they
don't conflict):

| Type | Host | Value |
|---|---|---|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | challengera91.github.io. |

DNS changes can take anywhere from a few minutes to a few hours to take
effect — that's normal, not broken. Once it resolves, GitHub automatically
issues an HTTPS certificate for the domain with no further action from
you. Say the word once you've added those and I'll check that it's
resolving.

## If you want a proper backend later

Right now `version.json` is the entire "server." If you later want to
track download counts, push a changelog, or manage multiple builds
(Windows/Mac, playtest vs. public), that's a small Supabase project
instead of a static file — a table with one row per version. Not needed
today; flagging it so future-you knows the upgrade path exists without
throwing away any of this.
