// ==UserScript==
// @name         YouTube Shorts Annihilator Pro
// @namespace    https://tampermonkey.net/
// @version      4.0.0
// @description  Multi-layer YouTube Shorts blocker using JSON API interception, CSS rules, DOM scrubbing, and route redirection.
// @author       Custom
// @match        *://*.youtube.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(function () {
    'use strict';

    const CONFIG = {
        REDIRECT_SHORTS_URL: true,
        REDIRECT_TARGET: 'https://www.youtube.com/', // Set to 'WATCH' to open in standard player instead
        OBSERVER_DEBOUNCE_MS: 80
    };

    const STYLE_ID = 'yt-shorts-annihilator-styles';

    const CSS_RULES = `
        /* Rich Shelves & Reel Shelves */
        ytd-reel-shelf-renderer,
        ytd-rich-shelf-renderer[is-shorts],
        ytd-rich-section-renderer:has(ytd-rich-shelf-renderer[is-shorts]),
        ytd-rich-section-renderer:has(ytd-reel-shelf-renderer),
        ytd-rich-section-renderer:has(a[href*="/shorts/"]),
        grid-shelf-view-model:has(a[href*="/shorts/"]),
        grid-shelf-view-model:has(ytm-shorts-lockup-view-model),
        grid-shelf-view-model:has(ytm-shorts-lockup-view-model-v2),

        /* Lockup View Models (2025-2026 Architecture) */
        ytm-shorts-lockup-view-model,
        ytm-shorts-lockup-view-model-v2,
        yt-shorts-lockup-view-model,
        ytd-rich-item-renderer:has(ytm-shorts-lockup-view-model),
        ytd-rich-item-renderer:has(ytm-shorts-lockup-view-model-v2),
        ytd-rich-item-renderer:has(a[href*="/shorts/"]),

        /* Search Results, Related Videos & Channel Grids */
        ytd-video-renderer:has(a[href*="/shorts/"]),
        ytd-grid-video-renderer:has(a[href*="/shorts/"]),
        ytd-compact-video-renderer:has(a[href*="/shorts/"]),
        yt-lockup-view-model:has(a[href*="/shorts/"]),
        ytd-Item-section-renderer:has(ytd-reel-shelf-renderer),

        /* Left Sidebar Navigation Buttons */
        ytd-guide-entry-renderer:has(a[title="Shorts"]),
        ytd-guide-entry-renderer:has(a[href*="/shorts"]),
        ytd-mini-guide-entry-renderer:has(a[title="Shorts"]),
        ytd-mini-guide-entry-renderer[aria-label="Shorts"],

        /* Channel Tabs & Feed Filter Chips */
        yt-tab-shape[tab-title="Shorts"],
        tp-yt-paper-tab:has([title="Shorts"]),
        yt-chip-cloud-chip-renderer:has(yt-formatted-string[title="Shorts"]) {
            display: none !important;
            visibility: hidden !important;
            height: 0 !important;
            max-height: 0 !important;
            width: 0 !important;
            margin: 0 !important;
            padding: 0 !important;
            overflow: hidden !important;
            pointer-events: none !important;
        }
    `;

    // Layer 1: Inject persistent CSS stylesheet
    function ensureStylesheet() {
        if (document.getElementById(STYLE_ID)) return;
        const root = document.head || document.documentElement;
        if (!root) return;

        const style = document.createElement('style');
        style.id = STYLE_ID;
        style.type = 'text/css';
        style.appendChild(document.createTextNode(CSS_RULES));
        root.appendChild(style);
    }

    // Layer 2: Block and redirect direct /shorts/ URLs
    function handleShortsNavigation() {
        if (!CONFIG.REDIRECT_SHORTS_URL) return;

        const currentPath = window.location.pathname;
        if (currentPath.startsWith('/shorts/')) {
            document.querySelectorAll('video').forEach(video => {
                try {
                    video.pause();
                    video.removeAttribute('src');
                    video.load();
                } catch (_) {}
            });

            if (CONFIG.REDIRECT_TARGET === 'WATCH') {
                const videoId = currentPath.split('/shorts/')[1]?.split(/[/?#&]/)[0];
                if (videoId) {
                    window.location.replace(`https://www.youtube.com/watch?v=${videoId}`);
                    return;
                }
            }

            window.location.replace(CONFIG.REDIRECT_TARGET);
        }
    }

    // Layer 3: Deep DOM inspection and node removal for dynamic elements
    function scrubDOM() {
        // Remove any rich shelf explicitly marked as shorts
        document.querySelectorAll('ytd-rich-shelf-renderer[is-shorts], ytd-reel-shelf-renderer').forEach(shelf => {
            const section = shelf.closest('ytd-rich-section-renderer') || shelf;
            section.remove();
        });

        // Remove shelves by checking header text ("Shorts")
        document.querySelectorAll('ytd-rich-shelf-renderer, grid-shelf-view-model').forEach(shelf => {
            const titleNode = shelf.querySelector('#title, [role="heading"], yt-formatted-string');
            if (titleNode && titleNode.textContent.trim().toLowerCase() === 'shorts') {
                const section = shelf.closest('ytd-rich-section-renderer') || shelf;
                section.remove();
            }
        });

        // Remove individual video cards linking to /shorts/
        document.querySelectorAll('a[href^="/shorts/"]').forEach(anchor => {
            const card = anchor.closest(
                'ytd-rich-section-renderer, ytd-reel-shelf-renderer, grid-shelf-view-model, ' +
                'ytd-rich-item-renderer, ytd-video-renderer, ytd-grid-video-renderer, ' +
                'ytd-compact-video-renderer, yt-lockup-view-model, ytm-shorts-lockup-view-model-v2, ytm-shorts-lockup-view-model'
            );
            if (card) {
                card.remove();
            }
        });

        // Remove sidebar entries matching "Shorts"
        document.querySelectorAll('ytd-guide-entry-renderer, ytd-mini-guide-entry-renderer').forEach(entry => {
            const titleAttr = entry.querySelector('a')?.getAttribute('title');
            const labelText = entry.querySelector('.title, yt-formatted-string')?.textContent?.trim();
            if (titleAttr === 'Shorts' || labelText === 'Shorts') {
                entry.remove();
            }
        });

        // Remove Shorts tab on channel pages
        document.querySelectorAll('yt-tab-shape, tp-yt-paper-tab').forEach(tab => {
            if (tab.textContent.trim().toLowerCase() === 'shorts') {
                tab.remove();
            }
        });
    }

    // Layer 4: Recursively strip Shorts objects from YouTube API JSON payloads
    function sanitizeYouTubeData(obj) {
        if (!obj || typeof obj !== 'object') return obj;

        if (Array.isArray(obj)) {
            for (let i = obj.length - 1; i >= 0; i--) {
                const item = obj[i];
                if (isShortsDataNode(item)) {
                    obj.splice(i, 1);
                } else {
                    sanitizeYouTubeData(item);
                }
            }
            return obj;
        }

        for (const key of Object.keys(obj)) {
            const val = obj[key];
            if (val && typeof val === 'object') {
                if (isShortsDataNode(val)) {
                    delete obj[key];
                } else {
                    sanitizeYouTubeData(val);
                }
            }
        }
        return obj;
    }

    function isShortsDataNode(node) {
        if (!node || typeof node !== 'object') return false;

        if (
            node.reelShelfRenderer ||
            node.reelItemRenderer ||
            node.shortsLockupViewModel ||
            node.richShelfRenderer?.isShorts ||
            node.richSectionRenderer?.content?.richShelfRenderer?.isShorts ||
            node.richSectionRenderer?.content?.reelShelfRenderer
        ) {
            return true;
        }

        const navUrl =
            node.navigationEndpoint?.commandMetadata?.webCommandMetadata?.url ||
            node.richItemRenderer?.content?.shortsLockupViewModel?.onTap?.innertubeCommand?.commandMetadata?.webCommandMetadata?.url;

        if (typeof navUrl === 'string' && navUrl.startsWith('/shorts/')) {
            return true;
        }

        return false;
    }

    // Hook into window.fetch to clean YouTube internal API responses
    function interceptFetchAPI() {
        const originalFetch = window.fetch;
        if (!originalFetch) return;

        window.fetch = async function (...args) {
            const response = await originalFetch.apply(this, args);
            try {
                const url = typeof args[0] === 'string' ? args[0] : args[0]?.url || '';
                if (url.includes('/youtubei/v1/browse') || url.includes('/youtubei/v1/search') || url.includes('/youtubei/v1/next')) {
                    const cloned = response.clone();
                    const data = await cloned.json();
                    sanitizeYouTubeData(data);
                    return new Response(JSON.stringify(data), {
                        status: response.status,
                        statusText: response.statusText,
                        headers: response.headers
                    });
                }
            } catch (_) {
                // Fallback to original response if parsing fails
            }
            return response;
        };
    }

    // Bootstrapping
    ensureStylesheet();
    handleShortsNavigation();
    interceptFetchAPI();

    // Navigation & History hooks
    window.addEventListener('yt-navigate-start', handleShortsNavigation, true);
    window.addEventListener('yt-navigate-finish', () => {
        ensureStylesheet();
        handleShortsNavigation();
        scrubDOM();
    }, true);
    window.addEventListener('popstate', handleShortsNavigation, true);
    window.addEventListener('DOMContentLoaded', () => {
        if (window.ytInitialData) {
            sanitizeYouTubeData(window.ytInitialData);
        }
        ensureStylesheet();
        scrubDOM();
    });

    // Debounced MutationObserver
    let timer = null;
    const observer = new MutationObserver(() => {
        if (timer) return;
        timer = setTimeout(() => {
            timer = null;
            ensureStylesheet();
            handleShortsNavigation();
            scrubDOM();
        }, CONFIG.OBSERVER_DEBOUNCE_MS);
    });

    observer.observe(document.documentElement, {
        childList: true,
        subtree: true
    });
})();
