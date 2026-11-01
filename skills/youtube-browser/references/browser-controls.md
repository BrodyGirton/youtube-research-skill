# Browser controls and recovery

Use the current session's browser tools and returned documentation. These examples were verified with `mcp__cua_repl.js` in the ChatGPT in-app browser on 2026-11-01. They do not authorize private APIs, new packages, hidden application-state reads, or unadvertised capabilities.

## Bind the intended surface

On the first `cua_repl` invocation, perform exactly one documented entry-point call. For a new YouTube search in the requested built-in browser:

```javascript
var ytTab = await cua.createBrowserTab(
  'iab',
  'https://www.youtube.com/results?search_query=' + encodeURIComponent('TOPIC'),
  { visible: false }
);
```

Replace TOPIC with the actual user-relevant query. Use `visible: true` when the user wants to watch the interaction or the browser is itself a deliverable. Read the returned browser documentation and initial state before further calls. For an explicitly selected tab, use the tool's tab-mention entry point instead. If inventory is necessary, `await cua.getState()` is an alternative first call; do not run it speculatively after a suitable browser is bound.

Refresh a stale tab after an interruption without selecting a different browser:

```javascript
await cua.listTabs({ browser: BOUND_BROWSER_ID });
```

Inspect the returned entries, then bind the matching current tab ID:

```javascript
ytTab = await cua.getTab(OBSERVED_TAB_ID, { browser: BOUND_BROWSER_ID });
```

Do not hardcode test-session IDs in a reusable workflow. If the handle was discarded by a runtime reset, initialize again as the tool requires. If context begins with a computer-use summary, refresh documentation as the tool requires.

## Interact from observed UI

Read `getAXState()` or the documented DOM snapshot. Use fresh element indices or locators grounded in that state. After actions, observe the resulting state before deciding the next action. Never copy an old numeric index from this reference or assume a selector from a past page will still match.

When the returned browser documentation advertises Playwright:

```javascript
nodeRepl.write(await ytTab.playwright.domSnapshot());
```

If the snapshot exposes the corresponding buttons, these exact-label actions are examples:

```javascript
await ytTab.playwright.getByRole('button', { name: 'more', exact: true }).click();
nodeRepl.write(await ytTab.playwright.domSnapshot());
// After observing Show transcript in the new snapshot:
await ytTab.playwright.getByRole('button', { name: 'Show transcript', exact: true }).click();
nodeRepl.write(await ytTab.content.exportYouTubeTranscript());
```

Labels depend on language, layout, player state, and extensions. The examples are not selectors to retry blindly. If the description is already expanded, do not click `more` again. Expand only the needed panel. Avoid dumping repeated ad URLs or complete recommendations into working context; retain the page regions relevant to the next decision. A filtered snapshot must not hide a blocker or the evidence needed to ground an action.

The export returns a local UTF-8 file path, or an availability error. File reading is appropriate for that actual returned artifact. Inspect its header and time coverage; metadata alone does not establish a transcript's completeness. Keep raw transcripts in task-local work, not the skill or final deliverables by default.

## Playback and precise moments

Use visible transcript/chapter links or the canonical video URL with an original-video time:

```javascript
await ytTab.goto('https://www.youtube.com/watch?v=OBSERVED_VIDEO_ID&t=739s');
nodeRepl.write(await ytTab.playwright.domSnapshot());
```

Substitute an actually observed video ID and relevant time. Navigation can collapse panels or reset playback; do not repeatedly reload an already-correct page. Native slider/keyboard actions are alternatives when the current tool supports them and the player has focus. Do not seek by assigning `video.currentTime` through a read-only evaluation API.

If read-only DOM evaluation is advertised, this can inspect media state without changing it:

```javascript
nodeRepl.write(await ytTab.playwright.evaluate(() => {
  const video = document.querySelector('video');
  return video ? {
    currentTime: video.currentTime,
    duration: video.duration,
    paused: video.paused,
    readyState: video.readyState
  } : null;
}));
```

This alone does not prove that the displayed screenshot has decoded the requested scene: a poster can remain visible after seeking. Observe the screenshot. If needed, use the actual visible Play control, let playback load through the next state observation, then use the observed Pause control. Verify the time reached; tool-call delays or segment skipping can move beyond the intended moment. Revisit a material missed scene through documented controls, or use local frame extraction when available. Record actual times and any offsets, not just the requested target.

For screenshots, use the current documented screenshot API. The browser API's first-use reference is authoritative about saving, emitting, and cleanup. Use only supported mechanisms for any exported screenshot; do not invent local file paths or claim that a tool-only preview was delivered as a file.

## Bounded recovery

- An ad, loading spinner, and unavailable video are different conditions. Inspect which is present. Use a visible Skip button only when offered, then verify the target video's duration and identity.
- A missing transcript export before opening the panel can become available after opening it. Retry once after that concrete change, not in a blind loop.
- When a page is loading, use the tool's state-aware observations. Avoid arbitrary long sleeps and repeated full-page snapshots of unchanged state.
- If an extension changes the DOM, refresh state and derive new controls. Do not disable or configure extensions merely to restore an old selector.
- Stop after two concrete recovery attempts for the same blocked operation. Finish using available evidence or the bundled yt-dlp helper when permitted, and identify the gap.
- If browser control is denied, interrupted, or requires sign-in, describe that specific condition. Follow the tool's permission and handoff policy. Do not work around site restrictions, CAPTCHA, or security interstitials.

For local captions, media, or frame extraction, read [ytdlp-tooling.md](ytdlp-tooling.md) and use this skill's own `../scripts/youtube_evidence.py`. No separate companion skill is required. Verify the same video ID, language, and original source timestamps when combining browser and downloaded evidence.

Product reference: [OpenAI built-in browser guidance](https://help.openai.com/en/articles/20001277-using-the-built-in-browser-in-the-chatgpt-desktop-app). Live tool documentation governs API availability and permissions.
