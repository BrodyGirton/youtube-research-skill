# Download and install

This guide installs the **YouTube Research Skill** (`youtube-browser`) into Codex. Installing the instructions and installing the optional local evidence tools are separate steps. The Python tools are needed for broad discovery, downloaded captions/media, and offline evidence processing.

## 1. Download the skill

1. Open [the skill ZIP](downloads/youtube-browser.zip?raw=1). If GitHub displays a file preview, choose **Download raw file**.
2. Extract `youtube-browser.zip` using **Extract All** on Windows or your normal archive tool on macOS/Linux.
3. Find the extracted `youtube-browser` folder. It should contain `SKILL.md`, `agents`, `references`, `scripts`, and `requirements.txt`.

You can also use GitHub's **Code → Download ZIP**. That downloads the whole repository; the installable folder inside it is `skills/youtube-browser`.

An SHA-256 checksum is provided in [downloads/SHA256SUMS.txt](downloads/SHA256SUMS.txt). To verify your download:

```powershell
# Windows: run in the folder containing your downloaded ZIP.
Get-FileHash -LiteralPath .\youtube-browser.zip -Algorithm SHA256
```

```sh
# macOS
shasum -a 256 youtube-browser.zip
# Linux
sha256sum youtube-browser.zip
```

Compare the value with the checksum file.

## 2. Put the folder where Codex can find it

### Windows

1. Press **Win + R**, enter `%USERPROFILE%\.agents\skills`, and press Enter.
2. If that folder does not exist, create `.agents` in your user folder, then create `skills` inside it.
3. Copy the extracted `youtube-browser` folder into `skills`.

The result must be:

```text
%USERPROFILE%\.agents\skills\youtube-browser\SKILL.md
```

Do not leave an extra nested `youtube-browser` folder between the skill folder and `SKILL.md`.

### macOS and Linux

Create `~/.agents/skills` if necessary, then copy the extracted `youtube-browser` folder into it. In macOS Finder, **Go → Go to Folder** accepts `~/.agents/skills`.

The result must be:

```text
~/.agents/skills/youtube-browser/SKILL.md
```

### Existing installations and project-only installs

If your host already loads the skill from another personal directory, such as `~/.codex/skills`, update that installation instead of adding a duplicate. If you only want the skill in one project, place it at `<project>/.agents/skills/youtube-browser`.

Do not overwrite an existing skill folder without first saving a backup of any custom changes.

These paths are based on [OpenAI's local skill discovery documentation](https://learn.chatgpt.com/docs/build-skills). Other skill hosts may use different locations or lack this workflow's browser tools.

## 3. Install the optional local evidence tools

For the full workflow, install:

- **Python 3.11 or newer** from [python.org](https://www.python.org/downloads/) or your operating system's package manager.
- **Node.js 22+** from [nodejs.org](https://nodejs.org/en/download), or **Deno 2.3+** from [Deno's instructions](https://docs.deno.com/runtime/getting_started/installation/).
- **FFmpeg and ffprobe** using an option linked from [ffmpeg.org](https://ffmpeg.org/download.html). They are needed for frame extraction, audio conversion, and merging separate video/audio streams.

The runtime minimums above reflect [yt-dlp's upstream requirements](https://github.com/yt-dlp/yt-dlp/wiki/EJS) checked on November 1, 2026. Consult that page if requirements change. The helper enables Node automatically when Deno is absent.

Make the installed commands available on `PATH`, then open a new terminal. Check `python --version` (or `python3 --version`), `node --version` or `deno --version`, `ffmpeg -version`, and `ffprobe -version` as appropriate. On Windows, if `python` is unavailable, try `py` for the environment-creation command below.

### Windows setup commands

Open PowerShell and run:

```powershell
$skillDir = Join-Path $env:USERPROFILE '.agents\skills\youtube-browser'
python -m venv "$skillDir\.venv"
& "$skillDir\.venv\Scripts\python.exe" -m pip install --upgrade pip
& "$skillDir\.venv\Scripts\python.exe" -m pip install --upgrade -r "$skillDir\requirements.txt"
& "$skillDir\.venv\Scripts\python.exe" "$skillDir\scripts\youtube_evidence.py" doctor
```

Change `$skillDir` if you chose a different installation location. You do not need to activate the environment or change PowerShell's execution policy. If `.venv` already exists, use its Python for updates rather than recreating it.

### macOS / Linux setup commands

Open Terminal and run:

```sh
skill_dir="$HOME/.agents/skills/youtube-browser"
python3 -m venv "$skill_dir/.venv"
"$skill_dir/.venv/bin/python" -m pip install --upgrade pip
"$skill_dir/.venv/bin/python" -m pip install --upgrade -r "$skill_dir/requirements.txt"
"$skill_dir/.venv/bin/python" "$skill_dir/scripts/youtube_evidence.py" doctor
```

Change `skill_dir` for a different installation location. Some Linux distributions require their `python3-venv` package before environment creation works.

### What a successful check means

The `doctor` command prints a JSON report. Look for:

- `yt_dlp.ok` set to `true` and a reported version.
- Paths for `ffmpeg` and `ffprobe` when you need video/frame/audio features.
- A path for `deno` or `node` for YouTube's JavaScript challenges.

A missing optional binary can leave some features unavailable. This local check does not guarantee that every YouTube video is accessible, has captions, or can be downloaded.

## 4. Use the skill

Start a new Codex chat. Mention `$youtube-browser` in the prompt; in hosts that use an `@` skill picker, select **YouTube Search and Analysis** there. If it does not appear, restart the host.

Try:

```text
Use $youtube-browser to analyze [YouTube URL]. Explain what the creator
actually demonstrates, preserve exact settings and values, and link to
the relevant timestamps. Tell me what evidence you could inspect.
```

Or:

```text
Use $youtube-browser to research [topic]. Use several complementary
searches, record keep/drop decisions, inspect the retained videos,
preserve transcripts and visual details, and write a thorough report.
Clean up source video downloads after verifying the saved evidence.
```

The assistant will need access to the scripts and an appropriate place to save research. Visual analysis requires actual image/video inspection tools. If the host cannot provide them, it should disclose the coverage limit.

## Update

1. Download the new skill ZIP.
2. Back up custom changes before replacing the instructions, references, and scripts.
3. Preserve your existing `.venv` in the same install location; it is not included in the ZIP.
4. Run the pip command from step 3 again to update dependencies when needed.
5. Run `doctor`, then start a new chat or restart the host if changes are not detected.

Keep research outputs outside the skill folder so updating the skill does not affect your research. If you move an existing installation, recreate its virtual environment at the new location.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Skill does not appear | Confirm `youtube-browser/SKILL.md` is directly in the skills directory, then restart the host. Check for a disabled skill setting or duplicate installation. |
| Python command not found | Install Python, open a new terminal, or use `py` on Windows / `python3` on macOS/Linux. |
| `No module named yt_dlp` | Install requirements using the skill's `.venv` Python, not an unrelated Python installation. |
| FFmpeg or ffprobe is `null` | Install the actual binaries and put their folder on `PATH`; a Python package named `ffmpeg` is insufficient. |
| JavaScript challenge error | Check the runtime version and update `yt-dlp[default]` in the same environment. |
| No transcript | The video may lack matching captions. Inspect available languages or use authorized audio transcription with a suitable tool. yt-dlp does not transcribe speech. |
| HTTP 403/429, sign-in, or bot check | Read the retained failure log. Stop repeated attempts; report the access limit or use source files you can legitimately provide. |
| No browser tool in the host | Use the helper-supported portions of the workflow and disclose the missing browser/visual coverage. Installing the skill does not install browser automation. |

More detail: [local evidence tooling](skills/youtube-browser/references/ytdlp-tooling.md) and [the broad research workflow](skills/youtube-browser/references/batch-research.md).

## Remove

Remove only the installed `youtube-browser` folder from your skills directory, after saving any custom changes. This also removes its private environment if it is inside that folder. Research saved elsewhere is unaffected.
