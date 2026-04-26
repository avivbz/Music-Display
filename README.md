# Music Display

A fullscreen, always-on music display for an iPad Mini 6 in landscape mode.
Hosted on GitHub Pages at <https://avivbz.github.io/music-display/>.

## What it shows

- **Top:** the current time, very large, centered.
- **Middle:** the currently playing song — title, artist, and source (Spotify or Radio Paradise).
- **Bottom:** three buttons — *Play / Pause*, *Open Spotify*, *Open Radio Paradise*.

The page is white-on-black, with no extra chrome, and uses the Wake Lock API where supported to keep the screen on.

## How it works

### Spotify

- Authorizes via the Spotify Web API using the **OAuth 2.0 PKCE flow** (no backend).
- Client ID: `23da5fe4e62345cb9ce601f75696bbe1`
- Redirect URI: `https://avivbz.github.io/music-display/callback`
- Polls `/me/player/currently-playing` every **5 seconds**.
- The Play / Pause button calls `/me/player/play` or `/me/player/pause`.
- Tokens are stored in `localStorage`; the refresh token is used to renew the access token automatically.

### Radio Paradise

- Polls `https://api.radioparadise.com/api/now_playing?chan=0` every **15 seconds**.
- Field names are read defensively (`title` / `song`, `artist` / `artist_name`).

### Source detection

The display picks the active source based on the most recent activity:

1. If Spotify reports `is_playing: true`, **Spotify** is shown.
2. Otherwise, if the user last tapped *Open Radio Paradise* (or last activity was RP), **Radio Paradise** is shown.
3. As a fallback, the last known Spotify track is shown with a *paused* indicator.

The "last source" is stored in `localStorage` as `last_source` (`spotify` or `rp`).

### Deep links

- *Open Spotify* → `spotify://`
- *Open Radio Paradise* → tries `radiopara://`, falls back to the App Store after ~1.8 s if the page is still visible.

## Files

```
.
├── index.html              # Main app (display + Spotify/RP polling + buttons)
├── callback/
│   └── index.html          # OAuth redirect target — hands the code back to the main app
├── .nojekyll               # Tell GitHub Pages to serve files as-is (skip Jekyll)
├── .github/
│   └── workflows/
│       └── deploy.yml      # Auto-deploy to GitHub Pages on push to main
└── README.md
```

## First-time setup

1. **Create the GitHub repo** `avivbz/music-display` and push the contents of this folder to the `main` branch.
2. In the repo, go to **Settings → Pages**, and set **Source** to **GitHub Actions**.
3. In the Spotify Developer Dashboard for the app with Client ID `23da5fe4e62345cb9ce601f75696bbe1`, register the redirect URI:

   ```
   https://avivbz.github.io/music-display/callback
   ```

4. Push to `main`. The included workflow (`.github/workflows/deploy.yml`) will publish the site automatically.
5. On the iPad: open `https://avivbz.github.io/music-display/` in Safari, tap **Sign in to Spotify** (top-right), authorize. Then **Share → Add to Home Screen** so it launches fullscreen.

## Tips for always-on use

- Add the page to the iPad Home Screen so it opens without Safari's UI.
- In *Settings → Display & Brightness → Auto-Lock*, set to **Never** while the iPad is docked.
- Enable *Settings → Accessibility → Guided Access* if you want to lock the iPad to this single page.
- The app requests a Wake Lock automatically where supported; on iPadOS this requires a recent Safari and may need a tap to activate.

## Notes

- All Spotify tokens live only in the browser's `localStorage`. There is no server.
- Radio Paradise's API must allow CORS from the GitHub Pages origin (currently it does for the public `now_playing` endpoint). If that ever changes, the RP track will silently stop updating and the display will fall back to the last known state.
