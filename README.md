# wako discovery catalog

Editorial list packs powering the **wako discovery** rows and heroes on the
[wako](https://wako.app) home tabs.

Served to the apps through the jsDelivr CDN:

- `https://cdn.jsdelivr.net/gh/wako-app/discovery-catalog@main/catalog.json` — regular profiles
- `https://cdn.jsdelivr.net/gh/wako-app/discovery-catalog@main/kids-catalog.json` — child profiles

Clients refresh at most every 12 hours and fall back to their last good copy
(then to an embedded snapshot) when unreachable. **Editing this repo updates
every client within the TTL — no app release needed.** Removing a row is the
kill switch for a dead or misbehaving source.

## Document format

```jsonc
{
  "version": 1,
  "tabs": {
    // Keys: foryou | movies | shows (anime reserved). Unknown keys are
    // ignored by older clients.
    "foryou": {
      // Optional hero override for the tab — feeds the billboard/carousel
      // pool. Trakt lists only. `title` becomes the hero badge label.
      "hero": { "src": "trakt", "user": "...", "slug": "...", "title": { "en": "..." } },
      "rows": [
        // A public Trakt list. `type` filters server-side (typed tabs).
        { "id": "stable-row-id", "src": "trakt", "user": "...", "slug": "...",
          "type": "movie", "title": { "en": "Row Title", "fr": "Titre" } },
        // A TMDB discover query. `{contentLang}` resolves to the profile's
        // content language. `skipForContentLang` hides the row for those
        // languages (e.g. an "in your language" rail is redundant in EN).
        { "id": "...", "src": "tmdb-discover", "mediaType": "movie",
          "params": { "withOriginalLanguage": "{contentLang}", "sortBy": "popularity.desc" },
          "skipForContentLang": ["en"], "title": { "en": "..." } },
        // A wako-api discovery pack pointer. Expands in place into the pack's
        // rows for THIS tab (rows resolved server-side — sources behind wako
        // API keys, e.g. MDBList dynamic lists; titles come from the server).
        // Removing the pointer is the kill switch for that route's traffic.
        { "src": "wako-api", "id": "main-pack", "path": "/discovery/packs/main" }
      ]
    }
  }
}
```

Forward compatibility is part of the contract: clients drop any row whose
`src` or shape they don't understand — adding new source kinds here never
breaks old app versions.

## Curation rules

- Lists must be **alive and maintained** (check `updated_at` via the Trakt API)
  — daily for freshness lists, within the year for evergreen ones.
- Keep cross-list overlap low (< 20% of the first 50 items) within a tab and
  against For You — the packs exist for discovery, not repetition.
- Keep it around **10 rows per tab maximum**.
- `id` is the row's stable identity (clients key UI and visibility prefs on
  it) — never reuse an id for different content.
