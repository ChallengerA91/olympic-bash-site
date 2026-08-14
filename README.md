# Olympic Bash — website

The download page for OlympicBash.com. A single HTML file, no build step —
same "double-click and it works" philosophy as the game itself. This is a
**separate project from the game repo on purpose**, so nothing here ever
collides with work happening in `Olympic Bash/`.

```
index.html          the whole site (nav, hero, download button, footer)
version.json         what "the server" is — see below
downloads/            put the actual game .zip here before you deploy
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
  "url": "https://olympicbash.com/downloads/OlympicBash.zip",
  "released": "TBD",
  "notes": "Playtest build — update this file each time you ship a new zip."
}
```

The page reads that file the moment someone loads it, so the Download
button always points at whatever's current — nobody ever downloads a stale
build by accident. Later, when the game itself wants to check "am I the
latest version," it fetches this exact same file and compares its own
version number to `version.version`. That's the whole contract — one small
JSON file, no account, no cost.

## Publishing a new build

1. Drop the new zip in `downloads/`.
2. Edit `version.json`: bump `version`, update `url` if the filename
   changed, set `released` to today's date.
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
file-size limit for a site this size, so the game zip can live in
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
