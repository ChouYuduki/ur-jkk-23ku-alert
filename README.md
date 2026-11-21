# UR / JKK Tokyo 23 Wards Vacancy Watcher

**English** | [简体中文](README.zh-CN.md) | [日本語](README.ja.md)

Watches **UR Chintai** (UR賃貸) and **JKK Tokyo** (東京都住宅供給公社, first-come-first-served vacancies) for newly listed apartments in Tokyo's 23 wards.
When rent + common fee is below your limit, it sends a push notification to your phone via [ntfy](https://ntfy.sh), and records how long each apartment stayed on the market before being taken.

> Unofficial tool. It calls the endpoints that the UR / JKK websites use internally, so it may break when the sites change. Please keep the request rate reasonable.

## Features

- **Real-time monitoring**: by default, UR is checked every 5 minutes and JKK every 3 minutes during the day (JKK is first-come-first-served, so it checks more often)
- **Price filter**: only notifies when rent + common fee is below `RENT_MAX` (default ¥120,000); layouts can be filtered too
- **Rich notifications**: ward, building, room number, layout, floor area, floor, price, nearest station; tapping a UR notification opens the room page
- **Rent-out tracking**: when a listing disappears, it is logged to CSV with its estimated time on the market; `rent_stats.py` summarizes by ward, layout and price band
- **Night mode**: checks every 58 minutes at night, and sends a low-priority notification (no sound) when switching modes
- **Backoff on failure**: waits longer after consecutive failures, alerts you after 3 failed rounds, and notifies you again when it recovers

Created automatically at runtime (excluded by `.gitignore`):

| File | Contents |
| --- | --- |
| `ur_seen.json` / `jkk_seen.json` | Listings already seen (don't delete, or everything will be notified again) |
| `ur_history.csv` / `jkk_history.csv` | Rent-out history, opens directly in Excel |

## Requirements

- Python 3.8+ (Linux / macOS)
- The ntfy app on your phone ([iOS](https://apps.apple.com/app/ntfy/id1625396347) / [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy))

## Usage

### 1. Set up phone notifications

In the ntfy app, tap **+** and create a topic name, e.g. `ur-8f3k2q`.

### 2. Get the scripts and set the topic

```bash
git clone https://github.com/ChouYuduki/ur-jkk-23ku-alert.git ~/house
cd ~/house
echo 'export NTFY_TOPIC=<the topic name you just created>' >> ~/.bashrc
source ~/.bashrc
```

### 3. Test notifications

```bash
python3 ur_watch.py --test
python3 jkk_watch.py --test
```

If your phone receives two test notifications, you're set.

### 4. Start watching (two terminals)

```bash
# Terminal 1
cd ~/house && python3 ur_watch.py

# Terminal 2
cd ~/house && python3 jkk_watch.py
```

- The first run sends "N matching vacancies right now"; after that, only new listings are notified
- Keep both terminals open and don't let the computer sleep

### 5. View rent-out statistics

```bash
python3 rent_stats.py          # all listings
python3 rent_stats.py --ok     # only listings that match your price filter
```

## Settings

Edit these near the top of each script:

| Setting | Default | Description |
| --- | --- | --- |
| `RENT_MAX` | `120000` | Notify only when rent + common fee is below this; `0` = no limit |
| `LAYOUTS` | `set()` | Layout filter, e.g. `{"1K", "1DK", "1LDK"}`; empty = all |
| `WARDS` (UR only) | all 23 wards | Remove wards you don't need |
| `INTERVAL_SEC` | UR `300` / JKK `180` | Daytime check interval (seconds) |
| `NIGHT_START` / `NIGHT_END` | `0` / `7` | Night hours in Japan time (can cross midnight, e.g. `18` / `9`) |
| `NIGHT_INTERVAL_SEC` | `3480` | Night check interval (seconds; 3480 s = 58 minutes) |
| `MAX_BACKOFF_SEC` | `3600` | Longest wait after consecutive failures (seconds) |
| `GONE_CONFIRM_ROUNDS` | `2` | A listing counts as rented only after missing for this many successful checks in a row |

Environment variables: `NTFY_TOPIC` (required), `NTFY_SERVER` (for a self-hosted ntfy), `DISCORD_WEBHOOK` (optional, also posts to Discord).

## Acknowledgements

These open-source projects were references for the endpoints and request flow:

- [songqintai/ur-monitor](https://github.com/songqintai/ur-monitor): UR search-by-area endpoint
- [tabikiji/UR_vacancy_notifier](https://github.com/tabikiji/UR_vacancy_notifier): two-step approach (vacancy counts, then room lists) for UR
- [m-okumura/akiya-monitor](https://github.com/m-okumura/akiya-monitor): HTTP search flow for JKKねっと
