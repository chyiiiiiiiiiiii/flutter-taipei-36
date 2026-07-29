# Flutter Taipei Meetup #36 — July 2026 Flutter Digest

**English** · [繁體中文](README.zh-TW.md)

![Flutter Taipei Meetup #36 deck in motion](docs/demo.gif)

A 21-slide self-contained HTML deck covering everything that happened in Flutter
between 28 Jun and 27 Jul 2026. Every item links to a verifiable source.

**[→ Open the deck](https://chyiiiiiiiiiiii.github.io/flutter-taipei-36/)**

> The slides themselves are in Traditional Chinese — they were presented at a
> Taipei meetup. The engine, the tooling and the method behind them are language
> agnostic and documented in English at
> **[meetup-deck](https://github.com/chyiiiiiiiiiiii/meetup-deck)**.

## What's covered

| Chapter | Slides | Highlights |
|---|---|---|
| Opening | 01–02 | The month's three throughlines |
| Official | 03–07 | Release train (3.44.5→.8, 3.47 beta), the single July blog post, `agent-plugins` gaining Claude Code / Codex / Cursor support, **how the Flutter team runs AI evals**, LG open-sourcing the webOS embedder |
| FlutterCon | 08–11 | Orlando overview, **session recordings** (Eric Seidel on why AI makes Flutter matter more), enterprise migration talks, the Agentic Engineering track |
| Flutter Scene | 12–14 | The real-time 3D engine, bdero's five July videos, **an autoplaying wall of 3D footage** |
| Ecosystem | 15–19 | SoFi and Knowunity case studies, Serverpod 4, the official channel, r/FlutterDev highlights, community open source with live demos |
| Closing | 20–21 | Calendar through year end, the month in numbers |

## Controls

| Key | Action |
|---|---|
| `→` `↓` `Space` | Next slide |
| `←` `↑` | Previous slide |
| `O` | Chapter overview |
| `F` | Fullscreen |
| `Esc` | Close the video player |
| Click the left / right screen edge | Page |
| Click a thumbnail | Play the video in place, or open the article / repo |

## How this deck was made

The engine, the research recipes and the verification tooling are packaged as a
separate project:

### → **[meetup-deck](https://github.com/chyiiiiiiiiiiii/meetup-deck)** (MIT)

- **The deck template** — the same engine this deck runs on, with eight layout
  patterns. No AI tooling needed; copy `deck-template.html` and start editing
- **Source recipes** — how to fetch from Reddit, YouTube, Medium, GitHub and
  JS-rendered conference sites when the obvious approach is blocked. For example
  Reddit's JSON API is blocked but its RSS works, and Flutter version numbers are
  only trustworthy from the official releases JSON
- **Verification script** — catches overflow, broken assets and orphaned headline
  lines before you present, and writes a contact sheet of every slide
- Using Claude Code? The whole repository is a skill — clone it into `~/.claude/skills/`

## Technical notes

- One HTML file, no framework, no build step
- Fonts are subsetted offline (690 glyphs actually used, 11 woff2 files, 1.3 MB total)
- Animated WebP and MP4 lazy-load; only the current slide and its neighbours decode
- auto-fit measures content height and scales down when needed, so nothing is clipped
  at any aspect ratio
- YouTube cannot embed from `file://` (error 153), so the deck checks `location.protocol`:
  it plays in place over http(s) and opens a tab under `file://`

## Sources

[`SOURCES.md`](SOURCES.md) has the complete URL list — usable as speaker notes.

Material comes from: blog.flutter.dev · docs.flutter.dev · flutter/agent-plugins ·
lg-flutter-webos · flutterconusa.dev · fscene.dev · youtube.com/@flutterdev ·
youtube.com/@bdero · youtube.com/@nextappevents · serverpod.dev · r/FlutterDev · nowa.dev

## Licence

The deck's **code, layout and styling** are MIT (see [LICENSE](LICENSE)).

The images, video thumbnails and MP4s under `assets/` are **not covered** — they belong
to their original owners (Google, LG, fscene.dev, and the respective YouTube channels
and repository authors) and are included here as citation for a community talk. Check
with the sources before reusing them.

For a clean template with no third-party media, use
[meetup-deck](https://github.com/chyiiiiiiiiiiii/meetup-deck).
