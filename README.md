# LyricVibe

LyricVibe is a premium Electron + React + TypeScript desktop lyrics visualizer designed for a vertical monitor setup. It detects music playback when possible, fetches lyrics through a provider layer, and renders large cinematic lyrics with album-art blur, neon motion, particles, and audio-reactive effects.

## Tech Stack

- Electron for the desktop shell
- React + TypeScript for the app UI
- Tailwind CSS plus custom CSS variables for the visual system
- Framer Motion for lyric transitions
- Web Audio API for optional beat-reactive visuals
- Local JSON settings stored in Electron `userData`

## Setup

```bash
npm install
npm run dev
```

Useful scripts:

```bash
npm run typecheck
npm run build
npm start
npm run build:vite
```

`npm run build` uses a Rollup-based local build script that avoids native esbuild service startup issues in restricted environments. `npm run build:vite` is kept for the standard Electron Vite pipeline.

## Amazon Music Detection

Amazon Music detection is implemented as a provider chain in `src/main/detection`.

1. **Amazon Music Web API provider**
   - File: `src/main/detection/amazonWebApiProvider.ts`
   - Uses the official Amazon Music Web API playback endpoints when an access token is supplied.
   - Requires the Login with Amazon access token plus the Security Profile ID in the `x-api-key` header.
   - The Client ID is used by the Login with Amazon helper. The Client Secret is not stored in the desktop app.
   - Amazon's current public docs describe these playback APIs as closed beta / preview, so this app treats the API path as optional.
   - Reference: [Amazon Music Web API Playback Overview](https://developer.amazon.com/docs/music/API_playback_overview.html)

### Login with Amazon setup

LyricVibe includes a small `auth.html` helper for Login with Amazon. It is designed for a personal GitHub Pages setup at:

```text
https://anttothe1.github.io/LyricVibe/auth.html
```

In your Amazon Security Profile web settings, add:

```text
Allowed JavaScript Origin:
https://anttothe1.github.io

Allowed Return URL:
https://anttothe1.github.io/LyricVibe/auth.html
```

Then run the app, paste your Login with Amazon Client ID, and choose **Sign in with Amazon**. The helper uses the direct browser-based PKCE flow, sends the access token back to LyricVibe with the `lyricvibe://amazon-auth` desktop callback, and saves it locally.

Important: Login with Amazon alone does not unlock all Amazon Music account data. Music endpoints still require Amazon Music beta approval, the right Music scopes, and a Security Profile ID that Amazon Music has enabled.

2. **Windows System Media Session provider**
   - File: `src/main/detection/systemMediaSessionProvider.ts`
   - Uses Windows Global System Media Transport Controls through PowerShell/WinRT.
   - Best when Amazon Music or a browser exposes the currently playing media to Windows.

3. **Window title provider**
   - File: `src/main/detection/windowTitleProvider.ts`
   - Reads visible Amazon Music/browser window titles and parses common `Title - Artist` formats.
   - Useful when media sessions are unavailable but the player window title contains track metadata.

4. **Manual search fallback**
   - Page: Manual Song Search
   - Lets you type a title/artist and load lyrics manually.

The app polls detection every few seconds and updates the visualizer when the detected song changes.

## Lyrics Providers

Lyrics are resolved in `src/renderer/lib/lyrics.ts`.

Built-in providers:

- `demo`: local synced demo lyrics with word-level highlighting on the first line.
- `lrclib`: public synced/unsynced lyric lookup by artist/title.
- `lyrics-ovh`: public unsynced lyric lookup by artist/title.

To add a new provider:

1. Implement the `LyricsProvider` shape in `src/renderer/lib/lyrics.ts`.
2. Add `search(query)` if the provider supports search.
3. Add `getLyrics(track)` and return:

```ts
{
  ok: true,
  provider: 'your-provider',
  synced: true,
  lines: [
    {
      id: 'line-1',
      text: 'Lyric text',
      startMs: 12000,
      endMs: 16500,
      words: [
        { text: 'Lyric', startMs: 12000, endMs: 13000 },
        { text: 'text', startMs: 13000, endMs: 16500 }
      ]
    }
  ]
}
```

4. Push the provider into the `lyricProviders` array.

If a provider only supports plain lyrics, set `synced: false` and return lines without timestamps. LyricVibe will display them in fallback line-by-line mode.

## Visual Themes

Themes live in `src/renderer/data/themes.ts`.

Included modes:

- Cyberpunk Neon
- Minimal Black
- Album Art Blur
- Space Particles
- Retro VHS
- Liquid Wave
- Rainy Window
- Hologram Mode

The Theme Editor can save a local custom theme and export/import it as JSON.

## Keyboard Shortcuts

- `F`: Toggle fullscreen
- `T`: Cycle theme
- `Arrow Up`: Increase lyric size
- `Arrow Down`: Decrease lyric size
- `Space`: Toggle playback through the Windows media session provider when supported

## Notes

- System audio capture is not automatic in Electron. Beat effects use the Web Audio API through a user-approved audio input such as microphone or stereo mix. If no input is enabled, LyricVibe keeps a smooth ambient animation.
- Album art appears when the detection or manual track source supplies an image URL. The app uses a generated visual fallback otherwise.
- Amazon Music API support may require beta access and credentials from Amazon.
