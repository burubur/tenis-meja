# Jadwal Turnamen Tenis Meja Indonesia

A one-page static site that lists table tennis (tenis meja / ping pong) tournaments in Indonesia.
It needs no build step: `index.html` (plain HTML/CSS/vanilla JS) reads `tournaments.json` and renders it.

## Design and views

The look is a bold sports-site style inspired by event pages such as worldtabletennis.com. It uses only original CSS gradients and patterns: no third-party logos, photos, or assets are copied or hotlinked.

- Near-black nav and footer, white content, an orange-to-red accent gradient, and blue badges for categories/divisions. Fonts: Barlow Condensed (headings, small uppercase labels) and Barlow (body), from Google Fonts.
- **Daftar (list):** large rounded event cards with a dark gradient band, date range, title, venue and city/province, status badges, an "Info turnamen" button, a grid of category badges, and a live countdown (Hari/Jam/Menit/Detik) for events that have not started. A strip under the band shows fee, registration deadline, and organizer.
- **Kalender:** a month grid (Monday first) with prev/next buttons, plus an agenda list for that month. Events without a date only appear in the list view. Picking a month in the Bulan filter jumps the calendar to it.
- **Detail page:** `#/event/<id>` (the id is URL-encoded). It has a back bar, a hero banner, and sections Ikhtisar, Divisi, Pendaftaran (fee, deadline, contact), and Sumber. An unknown id shows "Turnamen tidak ditemukan".
- Search, filters, and toggles apply to both list and calendar. The UI is in Indonesian and mobile-first.

```
tenis-meja-site/
├── index.html         # the whole site
├── tournaments.json   # the only data file (Tournament Scout's real data)
├── tournaments.sample.json  # 2 [CONTOH] demo entries, kept for layout testing
├── favicon.svg
└── README.md
```

## Preview locally

Chrome blocks `fetch()` of JSON from `file://`, so serve the folder over HTTP:

```bash
cd ~/Downloads/tenis-meja-site
python3 -m http.server 8765 --bind 127.0.0.1
# open http://localhost:8765
```

## Data format (`tournaments.json`)

This is the same shape that Tournament Scout produces, so Scout's file can replace this one unchanged.
Scout's master copy lives at `/home/box/shared/tournament-scout/tournaments.json` (on the bot box). Copy it over `tournaments.json` here.

```text
{
  "updated_at":  string, required   ISO 8601 date or datetime, e.g. "2026-11-17T20:50:00+07:00"
  "source_note": string | null      optional, free text about where the data came from
  "tournaments": array, required    may be empty []
}

tournament = {
  "id":                    string, required   unique slug, e.g. "kejurnas-tm-2026-jakarta"
  "name":                  string, required
  "start_date":            string | null      "YYYY-MM-DD"; null = date not announced yet
  "end_date":              string | null      "YYYY-MM-DD"; null = one-day event
  "city":                  string | null      used for the Kota filter, so keep spelling consistent ("Jakarta Selatan", "Bandung")
  "province":              string | null
  "venue":                 string | null
  "categories":            string[] | null    e.g. ["Tunggal Putra Umum", "Ganda Veteran 40+"]
  "registration_fee":      string | null      free text, e.g. "Rp150.000 per peserta"
  "registration_deadline": string | null      "YYYY-MM-DD"
  "organizer":             string | null
  "contact":               string | null      free text: official organizer contact from a public source only
  "source_urls":           string[], required at least one entry. http(s) URLs become links ("Sumber", or "Sumber 1", "Sumber 2", …);
                                              other text (e.g. "info komunitas, belum dikonfirmasi") is shown as plain text, not a link
  "date_found":            string | null      "YYYY-MM-DD", the date Scout found the entry
  "status":                "confirmed" | "unconfirmed"   anything other than "confirmed" shows a "Belum dikonfirmasi" badge
  "notes":                 string | null      shown under the details when present
  "sample":                boolean, optional  true = demo entry; shows a "Contoh data / sample" badge and a top banner
}
```

How the page handles the data:

- Optional fields that are `null`, missing, or `""` show as "Belum diumumkan".
- Events are sorted by `start_date`. Entries with `start_date: null` are listed last under "Tanggal belum diumumkan"; they are never treated as past and drop out when a month is picked.
- "Hanya yang terkonfirmasi" hides everything whose `status` is not `confirmed`. The count line reads e.g. "12 turnamen · 2 terkonfirmasi" for the cards currently shown.
- An event is "past" once its `end_date` (or `start_date` if there is no end date) is before today. Past events are hidden unless "Tampilkan yang sudah lewat" is ticked.
- The registration deadline is flagged as "N hari lagi" when it is 7 days away or less, and as "Pendaftaran ditutup" once it has passed.
- The Kota / Provinsi and Bulan dropdowns are built from the data (`city` and `province` as two option groups, and the month of `start_date`).
- Only `http(s)` URLs are rendered as links. Text is HTML-escaped.
- For backwards compatibility the page also accepts `divisions` in place of `categories` and a single `source_url` in place of `source_urls`.

Example entry:

```json
{
  "id": "contoh-open-bandung-2026",
  "name": "[CONTOH] Kejuaraan Tenis Meja Terbuka Bandung",
  "start_date": "2026-12-12",
  "end_date": "2026-12-14",
  "city": "Bandung",
  "province": "Jawa Barat",
  "venue": "GOR Contoh, Jl. Contoh No. 1",
  "categories": ["Tunggal Putra Umum", "Tunggal Putri Umum", "Ganda Veteran 40+", "Pemula"],
  "registration_fee": "Rp150.000 per peserta",
  "registration_deadline": "2026-11-22",
  "organizer": "Panitia Contoh",
  "contact": "kontak@example.com (contoh)",
  "source_urls": ["https://example.com", "https://example.com"],
  "date_found": "2026-11-17",
  "status": "confirmed",
  "notes": "Entri contoh untuk pratinjau. Bukan acara nyata.",
  "sample": true
}
```

`tournaments.sample.json` holds 2 sample entries (`sample: true`, names starting with `[CONTOH]`, sources set to `https://example.com`). To preview the layout with them, copy it over `tournaments.json`. Never add an invented tournament without `sample: true`.

### Privacy rule for published data

Only publish information that comes from a public web source. Never publish chat-group names, private-chat references, individual people's names, or personal phone numbers. Replace sources that exist only in community chats with the text `info komunitas, belum dikonfirmasi` (it is not rendered as a link), and set `contact` to `null` when it came from such a chat.

### Updating the data

1. Replace or edit `tournaments.json`, keeping the shape above, and set `updated_at` to now.
2. Check that it is valid JSON: `python3 -m json.tool tournaments.json > /dev/null && echo OK`
3. Reload the page. It fetches `tournaments.json?v=<timestamp>`, so a stale cached copy is never shown.

## Hosting (GitHub Pages, prepared but not published yet)

Planned repo: `https://github.com/burubur/tenis-meja` (public). Pages will serve the `main` branch root.
Planned live URL: **https://burubur.github.io/tenis-meja/**. The site goes live only after Burhan's go-ahead.

To publish (from this folder, which is already a git repo with one commit on `main`):

```bash
gh repo create burubur/tenis-meja --public --source . --push --description "Jadwal turnamen tenis meja Indonesia"
gh api -X POST repos/burubur/tenis-meja/pages -f 'source[branch]=main' -f 'source[path]=/'
gh api repos/burubur/tenis-meja/pages/builds/latest   # poll until "status": "built"
```

How updates flow:
1. Tournament Scout updates `/home/box/shared/tournament-scout/tournaments.json`.
2. Copy that file over `tournaments.json` in this repo.
3. Run the privacy check (see "Privacy rule for published data") and validate the JSON with `python3 -m json.tool tournaments.json > /dev/null`.
4. Commit and push: `git commit -am "data: update tournaments" && git push`.
5. GitHub Pages redeploys in about a minute. The page fetches `tournaments.json?v=<timestamp>`, so visitors see the new data on their next load.

**Alternative: Netlify Drop.** Drag the folder onto https://app.netlify.com/drop to get a `*.netlify.app` URL. Each data update means dragging the folder again, or linking the GitHub repo so every push deploys.
