# Blackpink Views

Blackpink Views tracks the YouTube view counts of BLACKPINK videos and estimates how many days each one needs to reach its next 100 million views. This repository holds the whole site: a Laravel application that reads the YouTube Data API, stores view snapshots in MySQL and renders the ranking with Blade.

## How it works

- `/` lists every video in the `videos` table, sorted by views, with the daily average and the days left to the next multiple of 100 million. The list can be re-sorted in the browser by views, days, average or publish date.
- `/?date=YYYY-MM-DD` shows the ranking as it was on that day, from the stored snapshots.
- `/details/{slug}` shows a single video.
- A view count older than 10 minutes is refreshed from the YouTube API on page load, and each fetch is saved in `videos_views`.

Artisan commands keep the data up to date:

| Command | What it does |
| --- | --- |
| `videos:autofetch` | Fetches the current view count of every video and saves a fixed snapshot. |
| `videos:autodelete {date?}` | Removes the duplicate snapshots of a day, keeping the last one per video and every fixed one. |
| `videos:getmedia` | Recalculates each video's daily growth average from the last seven days. |
| `videos:addplaylist {playlist} {type=video}` | Adds the videos of a YouTube playlist to the database. |

The scheduler (`routes/console.php`) runs `autofetch`, `autodelete` and `getmedia` once a day.

## Requirements

- PHP **8.2+** and Composer
- Node.js with npm (Vite 7)
- MySQL
- A YouTube Data API v3 key

## Running locally

Copy the environment file and fill in the database settings and `YOUTUBE_API_KEY`:

```bash
cp .env.example .env
```

Install the dependencies, generate the app key, create the tables and build the assets:

```bash
composer setup
```

Start the server, the queue, the log viewer and Vite together:

```bash
composer dev
```

Add videos from a playlist, then fetch their first counts:

```bash
php artisan videos:addplaylist <playlist-id>
php artisan videos:autofetch
```

To run the daily commands in production, call the Laravel scheduler from cron:

```bash
php artisan schedule:run
```
