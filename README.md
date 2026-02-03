# qul-resources

Static runtime assets for the Quran Competition app (mushaf pages, fonts, ligatures, ornaments).
This repo is served via a CDN (jsDelivr) and referenced from the app at runtime.

## What is in this repo

Only **runtime assets** live here. Build-time sources stay in the app repo.

```
mushaf/
  v4/
    pages/                # 1..604 page JSON files
    words.json
    verse-index.json
    meta.json
  fonts/
    v4/woff/              # page fonts p1..p604
    surah-name-v4.woff2
    surah-name-v4.ttf
  ligatures.json
  ornaments/
    surah-frame.svg
  manifest.json
```

## CDN usage (jsDelivr)

This repo is intended to be **public** and served via jsDelivr:

```
https://cdn.jsdelivr.net/gh/sadjous/qul-resources@v1.0.0/mushaf
```

The app reads `VITE_MUSHAF_BASE_URL` to know where to fetch assets.

## Setup (no build step)

This repo is static. Just add/replace files and push.

Recommended for updates:

1) Copy new assets into `mushaf/` (overwrite existing files).
2) Update `mushaf/manifest.json` (optional but recommended).
3) Commit + tag.
4) Push to GitHub.

## Update workflow

From the app repo (where assets are generated):

1) Copy assets into this repo:
```
rsync -a --delete /path/to/app/public/mushaf/ /path/to/qul-resources/mushaf/
```

2) (Optional) Update manifest:
```
python3 - <<'PY'
import json
from pathlib import Path
base = Path('mushaf')
page_count = len(list((base/'v4/pages').glob('*.json')))
font_count = len(list((base/'fonts/v4/woff').glob('p*.woff')))
manifest = {
  'version': 'v4',
  'pages': page_count,
  'fonts': {
    'v4_woff': font_count,
    'surah_name': ['surah-name-v4.woff2', 'surah-name-v4.ttf'],
  },
  'files': {
    'words': 'v4/words.json',
    'verse_index': 'v4/verse-index.json',
    'ligatures': 'ligatures.json',
    'ornament': 'ornaments/surah-frame.svg',
  },
}
(base/'manifest.json').write_text(json.dumps(manifest, indent=2), encoding='utf-8')
PY
```

3) Commit + tag + push:
```
git add mushaf
git commit -m "Update mushaf runtime assets"
git tag vX.Y.Z
git push && git push --tags
```

4) Update the app environment:
```
VITE_MUSHAF_BASE_URL=https://cdn.jsdelivr.net/gh/sadjous/qul-resources@vX.Y.Z/mushaf
```

## Verification

Check a few assets from the CDN:

```
curl -I https://cdn.jsdelivr.net/gh/sadjous/qul-resources@vX.Y.Z/mushaf/manifest.json
curl -I https://cdn.jsdelivr.net/gh/sadjous/qul-resources@vX.Y.Z/mushaf/v4/pages/1.json
curl -I https://cdn.jsdelivr.net/gh/sadjous/qul-resources@vX.Y.Z/mushaf/fonts/v4/woff/p1.woff
```

## Notes

- Keep this repo **public** if you want jsDelivr to work.
- Always pin a **tag** (not `main`) in production to avoid cache inconsistencies.
