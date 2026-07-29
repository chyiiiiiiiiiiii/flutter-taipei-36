# Flutter Taipei Meetup #36 — 2026 年 7 月 Flutter 月報

[English](README.md) · **繁體中文**

![Flutter Taipei Meetup #36 deck in motion](docs/demo.gif)

21 頁自包含 HTML 簡報，涵蓋 2026 年 6/28 – 7/27 的 Flutter 動態。
每一則都附可查證來源，內容只寫事實與說明。

**線上看**：https://chyiiiiiiiiiiii.github.io/flutter-taipei-36/

## 內容

| 章節 | 頁 | 重點 |
|---|---|---|
| 開場 | 01–02 | 七月的三條主線 |
| 官方 | 03–07 | 版本車道（3.44.5→.8、3.47 beta）、blog、agent-plugins 接三家 IDE、**官方 AI evals 實作拆解**、LG webOS embedder 開源 |
| FlutterCon | 08–11 | Orlando 總覽、**議程錄影**（Eric Seidel、Keynote）、企業遷移實例、Agentic Engineering 軌 |
| Flutter Scene | 12–14 | 即時 3D 引擎、bdero 七月 5 支影片、**自動播放的 3D 影片牆** |
| 生態與社群 | 15–19 | SoFi / Knowunity 案例、Serverpod 4、官方頻道、r/FlutterDev 熱門、社群開源 demo |
| 收尾 | 20–21 | 八月到年底行事曆、數字回顧 |

## 操作

| 按鍵 | 動作 |
|---|---|
| `→` `↓` `空白` | 下一頁 |
| `←` `↑` | 上一頁 |
| `O` | 章節總覽 |
| `F` | 全螢幕 |
| `Esc` | 關閉影片播放器 |
| 點畫面左右邊緣 | 翻頁 |
| 點縮圖 | 播 YouTube（頁內）／開文章／開 repo |

## 這份簡報是怎麼做的

引擎、取材方法與驗證工具都整理成一個獨立專案了：

### → **[meetup-deck](https://github.com/chyiiiiiiiiiiii/meetup-deck)**（MIT）

裡面有：

- **簡報引擎模板** —— 這份簡報用的同一套，含八種版面樣板。不需要任何 AI 工具，
  複製 `deck-template.html` 就能開始改
- **取材食譜** —— Reddit / YouTube / Medium / GitHub / 會議官網在「常規做法被擋」
  之後的可行路徑。例如 Reddit 的 JSON API 被封但 RSS 通、Flutter 版本號只有
  官方 releases JSON 是可信的
- **驗證腳本** —— 放映前抓 overflow、破圖、標題孤行，並產出所有頁面的 contact sheet
- 用 Claude Code 的話，整包就是一個 skill，clone 進 `~/.claude/skills/` 即可

## 技術

- 單一 HTML，無框架、無 build step
- 字體為離線子集（掃出實際用到的 690 字元，11 個 woff2 共 1.3 MB）
- 動態 webp 與 MP4 lazy load，只解碼當前頁與前後一頁
- auto-fit：量測內容高度，超出就等比縮放，任何螢幕比例都不會被切
- YouTube 從 `file://` 無法內嵌（error 153），故偵測 protocol：http(s) 下就地播放，file:// 下開新分頁

## 來源

`2026-07-來源清單.md` 有完整的 URL 清單，可當講稿。

素材出處：blog.flutter.dev · docs.flutter.dev · flutter/agent-plugins · lg-flutter-webos ·
flutterconusa.dev · fscene.dev · youtube.com/@flutterdev · youtube.com/@bdero ·
youtube.com/@nextappevents · serverpod.dev · r/FlutterDev · nowa.dev

素材版權屬各原始作者，此處僅為社群分享用途引用。

## 授權

簡報的**程式碼與版面**採 MIT（見 [LICENSE](LICENSE)），可自由取用改作。

`assets/` 底下的圖片、影片縮圖與 MP4 **不在此授權範圍內** ——
那些屬於各原始作者（Google、LG、fscene.dev、各 YouTube 頻道與 repo 作者），
本專案僅為社群分享目的引用。要重複使用請自行向來源確認。

想要乾淨、沒有第三方素材的模板，用 [meetup-deck](https://github.com/chyiiiiiiiiiiii/meetup-deck)。
