## Worldwide Electronic Entertainment Buffering Bot
## "WEEB-bot"

WEEB-bot is a Python automation tool that monitors your anime download list, searches [Nyaa.si](https://nyaa.si) for the next episode of each series you follow, and automatically adds the magnet link to your qBittorrent download queue, all without any manual input.

---

## Table of Contents
- [How It Works](#how-it-works)
- [Requirements](#requirements)
- [Installation](#installation)
- [Setting Up the Excel File](#setting-up-the-excel-file)
- [Configuration](#configuration)
- [Running the Program](#running-the-program)
- [Understanding the Output](#understanding-the-output)
- [Known Limitations](#known-limitations)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## How It Works

1. WEEB-bot reads your `Downloadlist.xlsx` file to get the list of series you follow, along with the last episode number you have downloaded for each.
2. For each enabled series, it calculates the next expected episode number and searches Nyaa.si for a matching torrent.
3. Each search result is validated against a set of filters: the release group (publisher), resolution, and a strict check that the episode number appears in the right position in the torrent title, directly after the series name, to prevent false positives.
4. If a valid match is found, the magnet link is sent to qBittorrent via its Web UI API, and the result is logged back into the Excel file.
5. WEEB-bot then immediately checks whether the next episode after that is also available, and keeps going until it either hits the configured per-run episode cap or finds no more available episodes for that series.

---

## Requirements

- **Python 3.8 or higher**
- **qBittorrent** with the **Web UI enabled** (Tools → Preferences → Web UI), qBittorrent must be running before WEEB-bot is started
- The following Python libraries:

```
openpyxl
requests
beautifulsoup4
qbittorrent-api
nyaapy (optional, WEEB-bot will fall back to scraping Nyaa.si directly if this is not installed)
```

WEEB-bot has been confirmed to work on **Windows 11**. It should also work on **Linux and macOS** provided Python 3.8+, qBittorrent with Web UI, and the required libraries are installed.

---

## Installation

**1. Clone the repository:**
```bash
git clone https://github.com/YOUR_USERNAME/WEEB-bot.git
cd WEEB-bot
```

**2. Install the required Python libraries:**
```bash
pip install openpyxl requests beautifulsoup4 qbittorrent-api nyaapy
```
> If `nyaapy` fails to install, you can skip it, WEEB-bot will fall back to scraping Nyaa.si directly using `requests` and `beautifulsoup4`.

**3. Set up qBittorrent:**
Enable the Web UI and create a username, password and download category. Full step-by-step instructions are in [qbittorrent-setup.md](qbittorrent-setup.md).

**4. Place `Downloadlist.xlsx` in the same folder as `WEEB-bot.py`**, or update the `EXCEL_PATH` variable in the configuration section to point to wherever you saved it.

**5. Add `Downloadlist.xlsx` to your `.gitignore`** to prevent your qBittorrent credentials from being accidentally pushed to your repository:
```
echo Downloadlist.xlsx >> .gitignore
```

---

## Setting Up the Excel File

A pre-formatted `Downloadlist.xlsx` file is included in this repository. It contains three sheets that WEEB-bot reads from and writes to:

---

### Sheet 1: `Parameters_Series`

This is your watchlist. Each row defines one series. The columns are:

| Column | Description | Example |
|---|---|---|
| `Enabled` | Whether WEEB-bot should process this series. Set to `No` to skip it. | `Yes` |
| `SeriesName` | The name of the series, exactly as it appears in the torrent title on Nyaa.si | `Boku no Hero Academia Final Season` |
| `Publisher` | The release group to filter for | `Erai-raws` |
| `Resolution` | The resolution to filter for | `1080p` |
| `OtherFilters` | Any additional search terms to include in the Nyaa search query | `WEBRip` |
| `CurrentFileNumber` | The episode number you last downloaded. WEEB-bot will search for this number + 1. | `10` |

> **Tip:** The `SeriesName` must match the way the release group writes it in their torrent titles. If searches are returning no results, check Nyaa.si manually and compare the title format.

---

### Sheet 2: `Download_Log`

This sheet is written to automatically by WEEB-bot, you do not need to edit it. Every time a torrent is successfully added to qBittorrent, a new row is appended here with the following information:

| Column | Description |
|---|---|
| `SeriesName` | Name of the series |
| `FileNumber` | Episode number that was downloaded |
| `DownloadedFileName` | Full torrent title as it appeared on Nyaa.si |
| `ReleaseDate` | Date the torrent was posted on Nyaa.si |
| `DownloadDate` | Date and time WEEB-bot added the torrent (UTC) |
| `MagnetLink` | The magnet link that was sent to qBittorrent |
| `Status` | Always `Added` for successfully queued torrents |
| `Notes` | Reserved for future use |

> The `Download_Log` also serves as a fallback: if `CurrentFileNumber` is empty for a series in `Parameters_Series`, WEEB-bot will look up the highest episode number it has ever logged for that series and use that instead.

---

### Sheet 3: `QB_Credentials`

This sheet stores your qBittorrent login details. WEEB-bot reads from it at startup. The columns are:

| Column | Description | Example |
|---|---|---|
| `QB_Username` | Your qBittorrent Web UI username | `WEEB-bot` |
| `QB_Password` | Your qBittorrent Web UI password | `your_password_here` |
| `QB_Category` | The download category WEEB-bot will tag torrents with in qBittorrent | `WEEB-bot` |

Your credentials go in **row 2**, directly below the header row.

> **Important:** `Downloadlist.xlsx` contains your qBittorrent password and should **never** be committed to a public repository. Add it to your `.gitignore` to keep it local. The repository includes a blank template version of the file, use that as your starting point and fill in your own credentials.

---

## Configuration

All user-adjustable settings are grouped at the top of `WEEB-bot.py` under the `USER CONFIGURATION` section. You should not need to edit anything outside of this section.

```python
# --- Excel file settings ---
EXCEL_PATH = "Downloadlist.xlsx"   # Path to your Excel file
TAB_PARAMS = "Parameters_Series"   # Sheet name for series parameters
TAB_LOG    = "Download_Log"        # Sheet name for the download log
TAB_CREDENTIALS = "QB_Credentials" # Sheet name for qBittorrent credentials

# --- qBittorrent Web UI connection ---
QB_HOST = "http://127.0.0.1"       # Host where qBittorrent Web UI is running
QB_PORT = 8080                     # Port for qBittorrent Web UI
                                   # Username, password and category are stored in
                                   # the QB_Credentials sheet in Downloadlist.xlsx

# --- Episode catch-up limit ---
MAX_EPISODES_PER_SERIES = 12       # Max episodes to download per series per run.
                                   # Set to None for unlimited.

# --- Episode number matching ---
EPISODE_NUMBER_SEARCH_WINDOW = 5   # Characters after series name to search for episode number.
                                   # Default of 5 covers standard " - 11" formatting.
                                   # Increase if your titles use longer separators.

# --- Scraping politeness delays (seconds) ---
SCRAPE_DELAY_MIN = 2.5
SCRAPE_DELAY_MAX = 5.0

# --- Inter-episode delay (seconds) ---
INTER_EPISODE_DELAY_MIN = 3.0
INTER_EPISODE_DELAY_MAX = 5.0

# --- qBittorrent confirmation settings ---
QB_CONFIRM_TIMEOUT = 8.0           # Seconds to wait for qBittorrent to confirm a torrent was added
QB_CONFIRM_POLL    = 0.5           # How often (seconds) to poll during confirmation wait
```

---

## Running the Program

### Recommended IDE, Thonny

For a lightweight, beginner-friendly way to run and edit WEEB-bot, [Thonny](https://thonny.org) is recommended. It is free, open source, and comes with Python built in, making setup straightforward. Download it from the official website: **https://thonny.org**

### Running from the command line

Make sure qBittorrent is running and its Web UI is accessible, then run WEEB-bot from the folder containing `WEEB-bot.py` and `Downloadlist.xlsx`:

```bash
python WEEB-bot.py
```

> **Important:** WEEB-bot checks the qBittorrent connection at startup. If qBittorrent is not running or the Web UI is not reachable, WEEB-bot will print an error and exit immediately, it will not search Nyaa.si if it cannot add torrents.

WEEB-bot will process each enabled series in your `Parameters_Series` sheet one by one and print its progress to the shell.

> **Tip:** WEEB-bot is designed to be run on a schedule. You can use **Windows Task Scheduler** (Windows), **cron** (Linux/macOS), or any other scheduler to run it automatically at a set interval, for example, once per day.

---

## Understanding the Output

WEEB-bot prints a status message for every significant action it takes, so you always know what it is doing at any point. A typical successful run looks like this:

```
Loading credentials from Downloadlist.xlsx
Connecting to qBittorrent Web UI...
Connected to qBittorrent Web UI.
Loading series from Downloadlist.xlsx

--- Processing series: 'Boku no Hero Academia Final Season' ---

  Searching for 'Boku no Hero Academia Final Season' episode 11 (query: Erai-raws Boku no Hero Academia Final Season 1080p 11)
  Delaying 3.4s before scraping Nyaa...
  3 result(s) found:
    1. [Erai-raws] Boku no Hero Academia Final Season - 11 [1080p CR WEBRip HEVC AAC][MultiSub] | Category: Anime - English-translated | Date: 2025-12-13 12:17
    2. [Erai-raws] Boku no Hero Academia Final Season - 11 [1080p CR WEB-DL AVC AAC][MultiSub]  | Category: Anime - English-translated | Date: 2025-12-13 09:32
    3. [Erai-raws] Boku no Hero Academia Final Season - 05v2 [1080p CR WEBRip HEVC AAC][MultiSub] | Category: Anime - English-translated | Date: 2025-11-02 07:26
  Accepted: '[Erai-raws] Boku no Hero Academia Final Season - 11 [1080p CR WEBRip HEVC AAC][MultiSub]' -> passed all filters (distance=34)
  Accepted: '[Erai-raws] Boku no Hero Academia Final Season - 11 [1080p CR WEB-DL AVC AAC][MultiSub]' -> passed all filters (distance=34)
  Rejected: '[Erai-raws] Boku no Hero Academia Final Season - 05v2 [...]' -> episode number 11 not found in the 5 characters after series name
  Final choice: '[Erai-raws] Boku no Hero Academia Final Season - 11 [1080p CR WEBRip HEVC AAC][MultiSub]' (distance=34)
  Adding torrent to qBittorrent...
  Updating download log and series list...
  Successfully queued and logged 'Boku no Hero Academia Final Season' #11.
  Waiting 5.2s before checking next episode...
  Searching for 'Boku no Hero Academia Final Season' episode 12 ...
  No results found. Moving to next series.

Logging out of qBittorrent Web UI...
Done.
```

If qBittorrent is not running when WEEB-bot starts, you will see:
```
Loading credentials from Downloadlist.xlsx
Connecting to qBittorrent Web UI...
ERROR: Could not connect to qBittorrent Web UI: [error details]
Make sure qBittorrent is running and the Web UI is enabled before running WEEB-bot.
```

When no new episode is available yet, WEEB-bot will report `No results found` and move on, it will try again the next time it runs.

---

## Known Limitations

- **qBittorrent must be started first:** WEEB-bot connects to qBittorrent's Web UI at startup. If qBittorrent is not running, WEEB-bot will exit with an error rather than continuing without it.
- **Nyaa.si availability:** WEEB-bot scrapes Nyaa.si directly. If the site is down or changes its HTML structure, searches may fail or return no results.
- **Episode number formatting:** The episode number matching assumes the episode number appears within `EPISODE_NUMBER_SEARCH_WINDOW` characters after the series name in the torrent title. Unusual title formats may require increasing this value in the configuration.
- **Series name must match Nyaa exactly:** The `SeriesName` in your Excel file must match how the release group writes it in their torrent titles. Minor differences in spacing, punctuation, or capitalisation may cause searches to return no results.

---

## Disclaimer

WEEB-bot is a general-purpose automation tool. It searches a publicly accessible website and passes URLs to a locally installed application. It does not host, store, reproduce, distribute, or transmit any copyrighted content, and it has no control over the content indexed by third-party websites such as Nyaa.si.

**The author of this software is not responsible for how it is used.** It is the sole responsibility of the user to ensure that their use of this software complies with all applicable local, national, and international laws, including but not limited to copyright law, intellectual property law, and any terms of service of third-party platforms accessed through or in conjunction with this software.

Downloading, distributing, or otherwise reproducing copyrighted material without authorisation from the rights holder may be illegal in your country. The author neither encourages nor condones the use of this software for any unlawful purpose.

This software is provided "as is", without warranty of any kind. The author accepts no liability for any damages, legal consequences, or other harm arising from the use or misuse of this software.

By using this software, you acknowledge that you have read this disclaimer and accept full and sole responsibility for your own actions.

---

## License

MIT License

Copyright (c) 2026 Sjiekesjeng

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
