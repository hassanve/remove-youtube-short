# Remove YouTube Shorts ( Oct 2026 Architecture)

A lightweight, multi-layer Userscript that completely removes YouTube Shorts from the homepage, search results, subscriptions, sidebar navigation, channel tabs, and related video feeds. Updated for YouTube's late-2026 View-Model (`ytm-shorts-lockup-view-model-v2`) and Trusted Types security policies.

## Features

- **API Payload Sanitization:** Intercepts `/youtubei/v1/` JSON responses (`browse`, `search`, `next`) and strips Shorts objects before they render.
- **Engine-Level CSS Blocking:** Uses modern `:has()` relational selectors targeting `/shorts/` endpoints rather than fragile class names.
- **Active DOM Scrubbing:** Automatically purges dynamically loaded Shorts shelves and cards from memory to save RAM and CPU cycles.
- **Route Interception:** Blocks direct navigation to `/shorts/*` URLs and immediately redirects to the YouTube homepage (or standard `/watch?v=` player).

## Installation

1. Install the [Tampermonkey](https://www.tampermonkey.net/) किंवा [Violentmonkey](https://violentmonkey.github.io/) browser extension.
2. **[Click here to install the Userscript](https://raw.githubusercontent.com/hassanve/remove-youtube-shorts/main/remove-youtube-shorts.user.js)**.

### Important: Chromium / Brave / Chrome (Manifest V3) Requirement

On modern Chromium-based browsers, you **must** enable user script permissions for Tampermonkey to execute scripts[cite: 1]:
1. Open `chrome://extensions` or `brave://extensions` in your address bar.
2. Enable **Developer mode** in the top-right corner.
3. Click **Details** on the Tampermonkey extension card.
4. Toggle **Allow User Scripts** to **ON**[cite: 1].
5. Hard refresh YouTube (`Ctrl + F5`).

## Configuration

You can customize the behavior when clicking a direct `/shorts/` link by editing the `CONFIG` object at the top of the script:

- `REDIRECT_TARGET: 'https://www.youtube.com/'` (Default): Redirects to the YouTube homepage.
- `REDIRECT_TARGET: 'WATCH'`: Redirects `/shorts/VIDEO_ID` to the standard `/watch?v=VIDEO_ID` player.

## License

MIT License
