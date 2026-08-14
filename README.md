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

1. Drop the new zip in `downloads/` (or wherever you're hosting large
   files — see the Netlify note below).
2. Edit `version.json`: bump `version`, update `url` if the filename
   changed, set `released` to today's date.
3. Redeploy (drag the folder onto Netlify again, or if you connect Netlify
   to a GitHub repo it redeploys automatically on push).

That's the entire release process. No code changes needed for a new build.

## Going live: OlympicBash.com

You own the domain. Two things need to happen once:

### 1. Host the files somewhere (I'd use Netlify — free, no command line)

1. Go to [app.netlify.com/drop](https://app.netlify.com/drop) and make a
   free account.
2. Drag this whole folder (`OlympicBash-Website`) onto the page.
3. Netlify gives you a random URL like `chipper-narwhal-123.netlify.app` —
   that's your site, live, immediately. Check it works before moving on.

Netlify handles files well over 100MB, so the game zip can live in
`downloads/` and get deployed right along with the site. If a build ever
gets huge (several hundred MB+), a game-file host like itch.io is a better
fit than baking it into the site deploy — worth revisiting once you know
the real file size.

### 2. Point OlympicBash.com at it

In Netlify: **Site settings → Domain management → Add a domain** → enter
`olympicbash.com` → Netlify shows you either a set of DNS records to add,
or (simpler) offers to become your DNS host directly.

Then in Namecheap: **Domain List → Manage → Nameservers**, or if you keep
Namecheap as DNS, **Advanced DNS**, and add exactly the records Netlify's
page shows you. It's usually one `A` record and one `CNAME`. DNS changes
can take up to a few hours to take effect — that's normal, not broken.

I can't do either of these two steps myself — they need your Netlify
login and your Namecheap login. Everything else (the actual site) is
already built.

## If you want a proper backend later

Right now `version.json` is the entire "server." If you later want to
track download counts, push a changelog, or manage multiple builds
(Windows/Mac, playtest vs. public), that's a small Supabase project
instead of a static file — a table with one row per version. Not needed
today; flagging it so future-you knows the upgrade path exists without
throwing away any of this.
