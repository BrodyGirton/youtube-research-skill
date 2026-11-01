# YouTube Research Skill

Research YouTube with a repeatable workflow that keeps the evidence behind the answer: source links, timestamped transcripts, visual observations, exact values, and conflicting claims.

Built for **Codex and compatible local skill hosts**. The installed skill is named **`youtube-browser`**.

**[Download the skill ZIP](downloads/youtube-browser.zip?raw=1)** · **[Download and installation guide](INSTALL.md)** · **[Skill instructions](skills/youtube-browser/SKILL.md)**

## What it does

- Searches across several queries, deduplicates candidates, and records why videos were kept or excluded.
- Collects public video metadata and available captions with the bundled yt-dlp helpers.
- Supports browser review and optional local video downloads and frame extraction.
- Preserves chronological transcripts, visual observations, precise facts, and per-video analyses before writing a combined report.
- Uses a small rolling window of source videos. Its finalization command checks saved evidence before deleting reviewed source media.
- Handles quick single-video questions without requiring the full research workflow.

The scripts collect and organize evidence. Your AI assistant performs relevance judgments, inspects images or video with available tools, and writes the analysis. A downloaded video or generated frame is not automatically reviewed.

## Quick start

1. [Download `youtube-browser.zip`](downloads/youtube-browser.zip?raw=1) and extract it.
2. Copy the **`youtube-browser` folder** into your personal skills directory:

   | Platform | Folder |
   | --- | --- |
   | Windows | `%USERPROFILE%\.agents\skills\youtube-browser` |
   | macOS / Linux | `~/.agents/skills/youtube-browser` |

3. Follow [the setup guide](INSTALL.md#3-install-the-optional-local-evidence-tools) to install Python and the local evidence tools for research that needs them.
4. Open a new Codex chat and ask:

   ```text
   Use $youtube-browser to research [your topic]. Search broadly,
   explain which videos you keep, preserve timestamped evidence and
   important details, and write a detailed comparison. Clean up source
   video downloads after reviewing the saved evidence.
   ```

If it does not appear, restart Codex. Existing installations in a host-specific directory can continue using that directory; avoid installing duplicate copies of the same skill.

## Requirements

| Capability | What you need |
| --- | --- |
| Load the skill and follow its instructions | A compatible skill host, such as Codex |
| Browse YouTube pages and inspect playback | Browser tools provided by your host |
| Collect metadata/captions or run broad discovery | Python 3.11+ and `yt-dlp[default]` |
| Solve YouTube JavaScript challenges when required | Deno or a supported Node.js version on `PATH` |
| Extract frames, merge video streams, or convert audio | FFmpeg and ffprobe on `PATH` |

Python helpers run locally. The skill does not bundle a model, browser integration, speech recognition service, or third-party binaries. Availability of visual/audio analysis depends on your host. No separate API key is required by these helpers.

## Example requests

**One video**

```text
Use $youtube-browser to explain this video: [YouTube URL].
Give the main steps, exact settings, timestamp links, and any uncertainty.
```

**Compare tutorials**

```text
Use $youtube-browser to compare tutorials for [task and version].
Include prerequisites, failed attempts, differences in the demonstrated
results, and the timestamps supporting your recommendations.
```

**Broad research**

```text
Use $youtube-browser to investigate [question] across several search angles.
Keep a screening log, context transcripts, fact ledgers, detailed per-video
analyses, and a final report. Preserve exact numbers and qualifications.
Delete reviewed source media only after verifying the saved evidence.
```

## Repository contents

```text
skills/youtube-browser/
  SKILL.md                    Workflow and evidence requirements
  agents/openai.yaml          Display metadata
  requirements.txt            Local Python dependency
  references/                 Browser, research, and setup guidance
  scripts/youtube_evidence.py Captions, metadata, media, and frames
  scripts/youtube_batch.py    Discovery, screening, context, and cleanup
tests/                        Offline helper checks
tools/package.py              Rebuild the downloadable skill ZIP
downloads/                    Ready-to-install package and checksum
INSTALL.md                    Full installation and troubleshooting guide
```

Research results, downloaded media, credentials, local environments, and the retired compatibility skill are excluded from this repository.

## Limitations and responsible use

Caption availability and YouTube access vary. Automatic captions can misstate words, values, or negation. Sparse screenshots do not establish full-video or audio coverage. Public counts cannot reveal actual retention, click-through rate, or private channel analytics.

Use public or otherwise authorized source material. The default helpers do not export browser cookies or sign into accounts. They do not bypass access restrictions. Keep your research outputs outside the installed skill folder.

## Development

From the repository root, with Python 3.11+:

```sh
python -m unittest discover -s tests -v
python tools/package.py
```

The tests use local fixtures and mocked acquisition; they do not download YouTube videos. Live access can change independently of a passing test suite.

Installation paths and dependency setup follow [OpenAI's skill documentation](https://learn.chatgpt.com/docs/build-skills), [yt-dlp installation guidance](https://github.com/yt-dlp/yt-dlp/wiki/Installation), and [YouTube JavaScript runtime guidance](https://github.com/yt-dlp/yt-dlp/wiki/EJS).
