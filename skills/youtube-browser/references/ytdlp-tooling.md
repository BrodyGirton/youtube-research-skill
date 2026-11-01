# yt-dlp tooling

## Setup and sources

Official references (checked 2026-11-01):

- [yt-dlp README and options](https://github.com/yt-dlp/yt-dlp#usage-and-options)
- [Installation](https://github.com/yt-dlp/yt-dlp/wiki/Installation)
- [YouTube JavaScript runtime/EJS setup](https://github.com/yt-dlp/yt-dlp/wiki/EJS)

The helper requires Python 3.11+. Prefer an isolated environment inside this skill, rather than changing a project's dependencies. On Windows, substituting the actual skill directory:

```powershell
$skillDir = Join-Path $env:USERPROFILE '.agents\skills\youtube-browser'
python -m venv "$skillDir\.venv"
& "$skillDir\.venv\Scripts\python.exe" -m pip install --upgrade 'yt-dlp[default]'
python "$skillDir\scripts\youtube_evidence.py" doctor
```

On macOS/Linux use `python3`, `.venv/bin/python`, and the local skill path. The helper automatically checks that private environment first. Do not recreate an existing environment unnecessarily. Update it through that environment's Python when extractor errors warrant an update. The portable ZIP excludes installed dependencies.

YouTube support needs matching yt-dlp EJS components and a supported JavaScript runtime. Installing `yt-dlp[default]` includes EJS. Deno is enabled by default; this helper explicitly enables Node when Deno is absent and Node is present. Check current upstream minimum runtime versions if a challenge fails. FFmpeg and ffprobe binaries are needed for frame extraction, audio conversion, and separate-stream video merging. A Python package called ffmpeg does not provide these binaries.

## Evidence collection

Examples use `HELPER` for the absolute path to `scripts/youtube_evidence.py` and `WORK_DIR` for a task-local directory. Quote paths and URLs; pass arguments as a list when invoking from code. Do not interpolate source text into shell commands.

```text
python HELPER doctor
python HELPER collect "https://www.youtube.com/watch?v=VIDEO_ID" --out WORK_DIR
python HELPER collect "https://youtu.be/VIDEO_ID" --out WORK_DIR --language es
python HELPER collect "https://youtu.be/VIDEO_ID" --out WORK_DIR --media video
python HELPER collect "https://youtu.be/VIDEO_ID" --out WORK_DIR --media audio
```

The helper accepts a single YouTube watch/Shorts/live/embed URL or video ID, canonicalizes it, disables user yt-dlp configuration/plugins, limits retries, and refuses active/upcoming live streams. For a requested set, collect each video separately. No automatic playlist expansion, comments, account cookies, or thumbnail download is performed.

Captions: prefer a manual exact/base-language track, then automatic original/exact/base tracks. Only JSON3-capable tracks are selected. The selected track is recorded as manual or automatic; automatic tracks can also be translated. Read the track name and `is_translated` flag where present. No matching track produces `unavailable`, not a fabricated transcript. To inspect other tracks, use the `caption_tracks` lists in metadata and rerun with the relevant language code. Existing JSON3 captions can be normalized offline:

```text
python HELPER normalize "captions.json3" --out WORK_DIR
```

Saved outputs:

- `metadata.json`: selected descriptive fields, chapters, public counts, available caption languages; excludes signed stream URLs and request headers.
- `captions.LANG.json3`, `transcript.json`, `transcript.md`: original captions and readable cue-level transcript, when available. Normalization merges only identical overlapping adjacent cues; non-identical rolling text remains intact.
- `media.*`: optional video at at most 720p, or extracted MP3 audio. Video selection does not fall back to an uncapped resolution. Increase resolution deliberately through yt-dlp directly if small on-screen text requires it.
- `manifest.json`: tool version, UTC collection time, canonical URL, stage statuses, warnings, and relative output paths. Always inspect it; files alone do not establish success. `partial` returns exit code 2 while preserving usable evidence; `failed` returns 1.

For thumbnail/package analysis, inspect the thumbnail through an available image tool. If needed, use yt-dlp directly with `--ignore-config --no-plugin-dirs --no-playlist --skip-download --write-thumbnail -o "WORK_DIR/%(id)s.%(ext)s" URL`; a title or description does not reveal the thumbnail's visual design.

## Visual evidence

Collect video, then choose timestamps for the hook, transitions, key demonstrations, and closing. Sample more densely around any claim about editing. Full-video downloads may be large; target a section with yt-dlp's `--download-sections "*START-END"` and FFmpeg when appropriate. Sections can download extra media depending on source format; preserve the requested source offset.

```text
python HELPER frames "PATH_TO_MEDIA" --out WORK_DIR --at 0 5 15 30
python HELPER frames "PATH_TO_CLIP" --out WORK_DIR --at 0 5 --offset 120
```

`--at` values and `--offset` are seconds. The helper rejects timestamps outside the local file and saves JPEGs plus `frames.json` with local and original-video times. Use a fresh frame directory to avoid overwriting evidence. Actually open frames with the image-viewing tool; generating files is not visual inspection. It is fine to use a video-capable tool instead when available. A static frame cannot prove motion or audio behavior.

## Recovery

- Read the failing stage's local log and retained manifest. Retry only after a concrete change (runtime, version, language, format); stop after two changed attempts and report the limitation.
- For missing captions, try another appropriate listed language or an available transcription tool after collecting audio. yt-dlp does not provide speech recognition. Do not provision paid services or upload media merely to fill a gap without authorization.
- For HTTP 403/429, bot checks, or restrictions, report the access failure. Do not rotate proxies or repeatedly hammer the service. Browser cookies are account credentials: use them only when the user explicitly authorizes that account access, via yt-dlp's supported local mechanism; do not print, copy into reports, or package them.
- Private/unavailable/region-limited media may require user-provided files or authorized access. Never equate extraction failure with the video's content. Use local media or supplied captions when available.
