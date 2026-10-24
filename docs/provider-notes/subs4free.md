# Subs4Free provider notes

## Public behavior

- Site: `https://www.subs4free.info`
- Supported media: movies only.
- Supported languages:
  - Greek, `ell` / `el`
  - English, `eng` / `en`
- Search endpoint shape: `/search_report.php?search=<query>&searchType=1`
- Download endpoint shape: `/getSub.php`
- No credentials are exposed in the Bazarr UI.

## Compatibility quirks

- Bazarr labels this provider as broken for some users, so live checks must prove both search and download before promotion.
- Search may return direct `.movie-details` result cards or a `Mov_sel` dropdown of matching movie pages.
- Result language is carried by `elgif` and `engif` sprite classes, plus language-specific URL paths.
- Download detail pages expose a hidden `id` field and an image submit button. The worker posts that id with generated `x` and `y` click coordinates to `/getSub.php`.
- The legacy anti-block sequence touches `images.subs4free.info/favicon.ico` and the two `subs4series.com/includes/anti-block...` URLs before posting the download form.
- Downloads can be direct subtitle bytes, ZIP archives, or RAR archives.
- `getSub.php` answers a granted download with a `302` to the file (`smp/tmp/<name>.zip`). It also enforces an anti-flood penalty on the client IP: a refused download is a `302` back to the subtitle's detail page with a short body such as `Refresh timorias [1]<BR>` or `Polla alla isxiei timoria...<BR>`. Followed, the redirect lands on the detail page, which still carries the download form. A second download a couple of seconds after the first was refused in live checks. The worker raises `DownloadLimitExceeded` for both shapes and does not retry, since a retry inside the penalty is refused again. The host maps that class onto its own `DownloadLimitExceeded`, which pauses the provider for 3 hours by default.

## Live notes

Live verification on 2026-05-31 reached `https://www.subs4free.info/` through Cloudflare and returned HTTP `200`. Search for `Inception` returned current `movie-details` result cards with Greek subtitle rows, uploader names, download counts, and download detail links.

## License notes

The implementation in this catalog is a clean-room MIT worker using only public request and markup behavior.
