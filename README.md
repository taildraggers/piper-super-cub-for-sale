# Piper Super Cub For Sale

Daily aggregator of Piper Super Cub aircraft classified listings from
[Barnstormers.com](https://www.barnstormers.com), published as a static page
(`docs/index.html`) meant to be embedded via `<iframe>` on taildraggers.com.

## How it works

- `scraper/barnstormers.py` runs two Barnstormers advanced searches, for ads whose
  headline contains **"Super Cub"** or **"PA18"**, walks each one's result pages
  (`&page=N`) until a page turns up no new ads, then visits each unique listing's
  detail page to pull out the price, location, and posted date (falling back to regex
  heuristics over the visible text since the site doesn't expose structured data).
  The title is derived from the listing URL's own SEO slug, since every detail page
  shares one generic `<title>`/`<h1>`.
- Only whole-aircraft-for-sale listings are published. Each ad's title must identify
  a Super Cub - a PA-18 (any variant: PA-18A, PA-18-150, ...), L-18 or L-21 model code,
  or "Super Cub" by name without some other Cub model (PA-11, PA-12, J-3, ...) in the
  title; see `_MODEL_CODE_RULES`/`_OTHER_MODEL_RE` in `scraper/barnstormers.py`.
  Titles that read as parts, accessories, services, or raffles are dropped (phrases
  like "on floats" or "on tundra tires" are allowed - see `ALLOWED_PHRASES` in
  `scraper/common.py`). Every surviving listing's title is rewritten to a canonical
  **`YEAR Piper MODEL`** form when the ad states a model year (e.g.
  `1955 Piper PA-18 Super Cub`), or just **`Piper MODEL`** when it doesn't - a missing
  year isn't disqualifying, since plenty of genuine ads simply don't state one in the
  title - regardless of how the original ad was worded, so the page reads consistently.
- `main.py` runs the scraper, de-duplicates results, and renders them into
  `docs/index.html` titled **"Other Piper Super Cub Ads on the Web"**, with one row per
  listing: Title (linked to the original ad), Price, Location, Date Posted, and Site
  Posted On. Links use `rel="noopener noreferrer"` and the page sets a
  `no-referrer` meta policy, so Barnstormers never sees that the click came from
  taildraggers.com.
- `.github/workflows/daily-scrape.yml` runs the whole thing once a day (13:00 UTC),
  commits the regenerated `docs/index.html` if it changed, and can also be triggered
  manually from the Actions tab (`workflow_dispatch`).

## One-time setup: enable GitHub Pages

This repo publishes `docs/index.html` as a plain static file — GitHub Pages just needs
to be pointed at it once:

1. Go to **Settings → Pages** in this repository.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Branch: `main`, folder: `/docs`. Save.
4. GitHub will publish the page at `https://taildraggers.github.io/piper-super-cub-for-sale/`
   (may take a minute or two the first time).

## Embedding on taildraggers.com

```html
<iframe
  src="https://taildraggers.github.io/piper-super-cub-for-sale/"
  title="Other Piper Super Cub Ads on the Web"
  style="width: 100%; height: 800px; border: 0;"
  loading="lazy">
</iframe>
```

## Running locally

```bash
pip install -r requirements.txt
python main.py
```

This writes/overwrites `docs/index.html`.

## Notes

- If Barnstormers changes its markup or is briefly unreachable, the run logs will
  show a `[warn]`/`[error]` line pointing at what broke rather than failing silently.
- The scraper identifies itself with a browser-like `User-Agent` and adds a short
  delay between requests to be polite to the site.
