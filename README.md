# CamCal

A clean calendar of Cambridge University talks and events, plus a searchable guide to college sports facilities.

- **Calendar tab**: big month view (agenda view on phones), source filters, event search, click a day or event for details.
- **Facilities tab**: search "squash", "pool", "24-hour gym" or a college name; or browse by category.
- **Automatic updates**: a scheduled GitHub Action re-downloads the event feeds every 3 hours and redeploys the site.

## How the auto-update works

```
sources.json  ->  scripts/update_events.py  ->  site/data/events.json  ->  site/ (static page)
```

`sources.json` lists the iCal feeds:

| Source | Feed |
|---|---|
| Talks.cam (CamTalks list 23088) | `https://talks.cam.ac.uk/show/ics/23088/` |
| University What's On | `https://webservices.admin.cam.ac.uk/events/api/programmes/11/calendar/events.ics?resultSetSize=20000` |

The script uses only the Python standard library. If a source is down, it keeps that source's previous events instead of blanking the calendar, and the page shows a small notice.

**Add another source** (e.g. another Talks.cam list, which is `https://talks.cam.ac.uk/show/ics/<list-number>/`): add an entry to `sources.json` with a new `id`, `name`, `url` and `color`. That's all.

## Publish it (free, GitHub Pages)

1. Create a GitHub repo and push this folder to the `main` branch.
2. Repo **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. Go to **Actions → "Refresh events and deploy" → Run workflow**. After about a minute the site is live at `https://<your-username>.github.io/<repo>/`.
4. Check that run's log: it prints how many events each source returned.

Also works on Netlify or Cloudflare Pages: set the build command to `python3 scripts/update_events.py` and the publish directory to `site`, then add a scheduled build hook (every 3 hours).

## Run it locally

```
python3 scripts/update_events.py      # refresh site/data/events.json
cd site && python3 -m http.server     # open http://localhost:8000
```

(Opening `index.html` by double-click won't work, because browsers block loading the JSON files from `file://`.)

## Edit facilities

Everything is in `site/data/facilities.json`. Each college has a `tags` object: the key is the facility, the value is an optional note shown on the card. Add a tag to the `tags` list at the top if you need a new facility type, with `syn` words people might search for.

## Things to know

- **Talks.cam is being rebuilt.** The University announced it would be replaced on 1 July 2026 by a secure version at the same domain with existing integrations kept, but as of 16 Nov 2026 the old-style page and feed link still show. If the feed ever changes, the Action log will show the Talks.cam source failing, and the fix is a one-line URL change in `sources.json`.
- The `site/data/events.json` shipped in this folder is a small **sample snapshot** so the page isn't empty. The first workflow run replaces it with the full live data.
- Event times are shown in UK time. All-day multi-day items (exhibitions) appear in an "Exhibitions & long-running" section rather than filling every day cell; toggle them with the chip above the calendar.
- This is an unofficial student project. Each event links back to its source page.
