# PlexAudit

Cross-references your Plex Media Server database against your actual files on disk. Generates a self-contained interactive HTML report — no server, no dependencies, just Python 3.

![image](http://i.imgur.com/Wl6FOWP.jpeg)

---
<h4>Support the project</h4>
<sub>
If you found this tool useful, please consider supporting my BandCamp projects:<br>
<a href="https://ferropop.bandcamp.com">ferropop.bandcamp.com</a> [name your price!]
</sub>

---

## Why use this?

- **"Why is Plex showing the wrong title?"** — see exactly what Plex matched vs. what it guessed from the filename
- **"I have 30,000 files — how do I know what Plex missed?"** — instantly see every file on disk that Plex has never touched, grouped by folder
- **"I have duplicates but don't know which to delete"** — toggle duplicate view and quality columns to compare resolution/bitrate side by side
- **"I fixed some matches in Plex — did it work?"** — re-run and check the Unmatched count

---

## Requirements

- Python 3.8+ (no external dependencies)
- Plex Media Server (Windows, macOS, or Linux)

---

## Usage

```bash
# Basic — auto-detects Plex DB on Windows
python plex_audit.py --scan "D:\Media" "E:\Media"

# Multiple scan directories
python plex_audit.py --scan "D:\Movies" "E:\TV Shows" "F:\Music"

# Specify DB manually
python plex_audit.py --db "C:\path\to\com.plexapp.plugins.library.db" --scan "D:\Media"

# Custom output filename
python plex_audit.py --scan "D:\Media" --out my_report.html

# Debug mode — prints sample paths to diagnose matching issues
python plex_audit.py --scan "D:\Media" --debug
```

The output is a single `.html` file. Open in any browser.

### Auto-detected DB locations (Windows)
```
C:\Users\{user}\AppData\Local\Plex Media Server\Plug-in Support\Databases\com.plexapp.plugins.library.db
C:\Users\{user}\AppData\Roaming\Plex Media Server\Plug-in Support\Databases\com.plexapp.plugins.library.db
C:\ProgramData\Plex Media Server\Plug-in Support\Databases\com.plexapp.plugins.library.db
```

---

## Status classifications

Every file gets one of five statuses:

| Status | Meaning |
|---|---|
| 🟢 **Matched** | File on disk, Plex has confirmed metadata from an external source (TMDb, TVDB, MusicBrainz, etc.) |
| 🟠 **Unmatched** | Plex ran its agent but came back empty — title is guessed from filename. Shows "Match…" in Plex's UI. |
| 🟡 **No metadata** | Plex never attempted a match (folder-browse libraries, personal media, etc.) |
| 🔴 **File missing** | Plex's DB references a file that no longer exists on disk |
| 🔵 **Not in Plex** | File is on disk but Plex has never scanned it |

**How matching is determined:** PlexAudit reads the `guid` field in Plex's database — not just whether a title is present. A file where Plex guessed the title from the filename still has a `local://` or `agents.none` guid and will correctly show as 🟠 Unmatched, even if the title looks right.

Music is handled differently: tracks sourced from embedded ID3 tags get `local://` guids by design. If a track has both a title and an artist/album populated, it's classified as 🟢 Matched regardless of guid.

---

## Report features

### Views
**TV Shows / Movies / Music / All** — columns are tailored per view. Switching tabs auto-enables the relevant extension filters (video for TV/Movies, audio for Music).

Column order is designed for easy visual matching:
- **TV:** Folder → Show → Filename → S → E → Episode Title → Library
- **Movies:** Folder → Filename → Plex Title → Library
- **Music:** Folder → Artist—Album → Filename → # → Track Title → Library

### Filters
- Click any **status card** to filter to just that category
- **Library** dropdown and **search** box (matches filename, folder, show, episode, track)
- **Extension pills** — every extension found is shown as a toggleable pill. Click a category heading (VIDEO / AUDIO / IMAGE / OTHER) to toggle the whole group
- **Reset** clears everything

### Quality columns
Toggle **"Quality columns"** to append Resolution, Bitrate, File size, and Duration to any view. Pulled directly from Plex's DB. Useful for comparing duplicates — a lower-resolution file with a higher bitrate is almost always the worse encode.

### Duplicate filter
Toggle **"Show duplicates"** to filter down to only files where another copy exists in the current view:
- **Movies** — same Plex title
- **TV** — same show + episode title
- **Music** — same artist/album + track title

Duplicates sort adjacent for direct side-by-side comparison. Combine with Quality columns to decide which version to keep.

### Column controls
- **Resize** — drag the right edge of any header. Content clips at any width.
- **Reorder** — drag column headers to swap order. Each view remembers its own arrangement.
- **Sort** — click any header. Sorting by Folder uses filename as a secondary sort key.

### Right-click any filename
- Copy full path
- Copy folder path
- Copy `explorer /select,"D:\path\to\file.mkv"` — paste into **Win+R** to open Explorer with the file highlighted

---

## Reading the database

PlexAudit opens the Plex SQLite DB in **read-only mode** — it never writes to or modifies anything.

Files with `library_section_id = NULL` are extras/featurettes attached to a parent movie. These show as "Extras/Featurettes" in the Library column.

---

## Debug mode

If everything shows as "Not in Plex", run with `--debug`. It prints 5 sample paths from both the Plex DB and disk scan side by side — the most common cause is a path prefix mismatch from a previous Docker or network mount configuration.

---

## License

MIT
