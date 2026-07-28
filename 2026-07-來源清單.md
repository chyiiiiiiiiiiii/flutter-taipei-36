---
tags: [flutter, meetup, 月報, 2026-07]
date: 2026-07-27
project: Flutter Taipei Meetup #36
---

# 2026 年 7 月 Flutter 大小事 — 來源清單

投影片：[[slides.html]]（瀏覽器直接開，← → 翻頁、F 全螢幕）
資料期間：2026-06-28 → 2026-07-27

---

## 七月摘要

- stable 停在 **3.44.8**，整月零 feature release；3.47 於 07-13 切出 beta 分支，預計 8 月 stable
- 官方 agent skills 一個月接上 **Claude Code / Codex / Cursor**，並加入 evals 評分機制
- **LG 開源 webOS embedder**（`lg-flutter-webos`，⭐201）
- **FlutterCon USA** 07-16~17 於 Orlando，70 位講者 / 65 場議程 / 400 位開發者
- **Flutter Scene** 3D 引擎七月連發 box3d 0.1.0、0.19.0、0.20.0 ＋ scene 0.1.0 ＋ rapier 0.3.0
- **Serverpod 4 public beta**（07-08）：全端 hot reload ＋ 內建 MCP server

---

## 1. 版本現況

| 日期 | 版本 | 頻道 |
|---|---|---|
| 07-06 | 3.44.5 | stable |
| 07-09 | 3.44.6 | stable |
| 07-13 | 3.47.0-0.1.pre（Dart 3.13.0） | beta 分支 |
| 07-20 | 3.44.7 | stable |
| 07-23 | 3.44.8 | stable |

- Dart stable 整月維持 3.12.2
- 7 月 **沒有** feature release；3.47 stable 預計 8 月
- 來源：`https://storage.googleapis.com/flutter_infra_release/releases/releases_macos.json`
- 來源：https://docs.flutter.dev/release/release-notes

## 2. 官方 blog（整月僅 1 篇）

- **Learning faster with Antigravity** — Andrew Brogdon，07-01
  https://blog.flutter.dev/learning-faster-with-antigravity-cd735bfe44e7
- （前一則，卡在上次小聚後）**Vibe once, run anywhere with Antigravity and Flutter** — Craig Labenz & Rody Davis，06-29
  https://blog.flutter.dev/vibe-once-run-anywhere-with-antigravity-and-flutter-25af06e60a91
- （補充）**How we built a Flutter-powered AI coffee shop** — Craig Labenz，06-22
  https://blog.flutter.dev/how-we-built-a-flutter-powered-ai-coffee-shop-878c60a11f1a

## 3. flutter/agent-plugins

https://github.com/flutter/agent-plugins — ⭐ 2,742，7 月 21 次 commit

| 日期 | 內容 | PR |
|---|---|---|
| 07-14 | Claude Code plugin 設定檔進 repo | #172 |
| 07-16 | export RuleConfig / RuleConfigPatch | #182 |
| 07-17 | 新增 Codex plugin | #177 |
| 07-17 | API boundary runner + CI check | #183 |
| 07-20 | tool/generator test 遷移到 ConfigParser.loadConfig | #186 |
| 07-21 | 新增 Cursor plugin 設定 | #178 |
| 07-21 | contributor-pr-description skill | #190 |
| 07-23 | run-evals 編排 skill + 替已發布 skill 補 evals | #189 / #193 |
| 07-24 | 用靜態資料集測 code quality rubric（Meta-Evals） | #194 |

安裝：`npx skills add dart-lang/skills --skill '*' --agent universal`
文件：https://docs.flutter.dev/ai/agent-skills · https://docs.flutter.dev/ai/mcp-server

### 官方的 AI evals 怎麼做（07-23 / 07-24 加入）

單一真實來源：`tool/dart_skills_lint/evals/README.md` 與 `.agents/skills/run-evals/SKILL.md`
官方自述：「這些 evaluation 本質上是 skill 的 unit test」，範圍僅限 `dart_skills_lint` 生態，不是通用評估框架。

**兩層 rubric**

1. **Per-skill** — 每個 skill 一份 `<skill>/evals/evals.json`，欄位：
   - `prompt`：真實的使用者請求（涵蓋主流程或宣稱能處理的邊界案例）
   - `expected_chat_output`：agent 該對使用者說什麼
   - `expected_repo_state`：可驗證的斷言陣列（repo 與追蹤檔案的最終狀態）
   - `repo_criteria`：指向共用 rubric 的相對路徑
   - `agent_config`：用哪個 harness（已發布 skill 用 `bare-agent`，內部貢獻者 skill 用個人 profile）
2. **Cross-skill** — `evals/code_quality_rubric.json`，四類通用標準：
   - `compilation_and_health`：零語法錯誤、`dart analyze --fatal-infos` 零錯誤零警告、新舊測試都要過
   - `effective_dart_and_idioms`：Effective Dart；分支代數型別時要用 Dart 3 switch expression 與窮盡 pattern matching；避免不必要的 dynamic、裸字串拼接、未處理的 Future
   - `cross_platform_compatibility`：路徑走 `package:path` 或正斜線；CLI 語法跨平台；不寫死 `C:\` 或 `/usr/local/bin`
   - `directory_and_placement_hygiene`：程式碼進 `lib/`、測試進 `test/`、可執行腳本進 `bin/` 或 `tool/`；執行後不留暫存檔或孤兒目錄

**run-evals 的執行流程**

1. 讀 README 分辨 per-skill 與 cross-skill
2. 定位目標 `evals.json`；cross-skill 則找 `evals/*_evals.json`
3. 依 `agent_config` 決定要 spawn 的 harness
4. 預設用 `Workspace: branch` 開一個 **With-Skill** subagent；**只有使用者明確要求比較時**才另開 **Baseline**（同任務、不給 skill）
5. 強制要求 subagent 把檔案修改限制在自己的工作目錄，禁用絕對路徑碰父 workspace
6. subagent **不准 commit**，只回傳 `git diff` 與 `dart format` / `dart analyze` / `dart test` 的輸出
7. LLM judge 依 `agent_judge_prompt.md`：對**每一條**期望明確標 PASS / FAIL 並給一句理由；FAIL 時必須同時列出「期望什麼」與「實際發現什麼導致失敗」
8. 產出 markdown artifact（metadata、判定理由、原始 diff 與 stdout）

**Workspace 隔離的硬性規定**：偵測到多個 workspace 掛載時 `Workspace: branch` 會失敗，此時**必須警告使用者、不得靜默 fallback 到 `Workspace: inherit`** —— 併發跑會造成 git state bleed，失敗情境的改動會被同時執行的成功情境看到。

**Meta-Evals（07-24）**：`evals/*_evals.json`（如 `code_quality_rubric_evals.json`）針對 `evals/test_data/` 下的靜態 fixture（`bad_code/` 與 `ok_code/`）評分，反過來驗證 rubric 本身抓不抓得到反模式、會不會誤殺乾淨的程式碼。

**刻意不評的東西**：`dart format` 本來就修得掉的排版瑣事、要跑好幾分鐘才生得出來的完整系統架構、超出 `dart_skills_lint` 範圍的 skill（例如一般 Flutter app 建立）。

**結構把關**：`dart test test/skills_evals_test.dart` 檢查全 repo 所有 `evals.json` 的結構一致性。

## 4. LG webOS — Flutter embedder 開源

- GitHub org `lg-flutter-webos`（06-28 建立）
  - `flutter-webos` ⭐201 https://github.com/lg-flutter-webos/flutter-webos
  - `plugins` / `ndk` / `devcontainer` / `artifacts`（整套 SDK）
- 官方 showcase：https://flutter.dev/showcase/lg-electronics
- Reddit 07-12 熱門：https://www.reddit.com/r/FlutterDev/comments/1uuh90d
- FlutterCon 相關議程：Flutter on Big Screen & Beyond — Jin Ho Chung, Yunsoo Kim（LG）

## 5. FlutterCon USA 2026

07-16~17 · Orlando, OCCC West Concourse · 與 droidCon Orlando 2026 併場
70 speakers / 65 sessions / 400 developers
議程：https://www.flutterconusa.dev/fluttercon-agenda

四條特別軌：Agentic Engineering · TechLead Summit · Unsolved Unconference · Flutter at Scale

**Keynote**：Flutter is everywhere — Craig Labenz, Loïc Sharma

### 議程錄影（07-18 起陸續上架）

上架在 **nextapp devCon** 頻道 https://www.youtube.com/@nextappevents （與 droidCon 場次混在同一頻道）
查詢日 2026-07-27，該頻道當時只有下列兩支 flutterCon USA 2026 場次：

| 上片日 | 標題 | 長度 | 連結 |
|---|---|---|---|
| 07-20 | **Why AI Makes Flutter More Important, Not Less** — Eric Seidel（Flutter 共同創辦人，現為 Shorebird CEO） | 27 min | https://youtu.be/KT399tupCn8 |
| 07-18 | **Keynote — Flutter Is Everywhere** — Craig Labenz & Loïc Sharma | 40 min | https://youtu.be/kryeoX3yO8s |

Eric Seidel 那支的簡報標題是「Why Flutter matters more in the AI era」，正面回應七月 Reddit 上「AI 時代 Flutter 開發者還有沒有搞頭」的討論。

### 規模化 / 企業
- Why SoFi ♥️ Flutter — Tyler Stromberg
- Conquering chaos at scale（Panel）— Eric Seidel, Jason McGraw, Maksim Zadorskii, Joni Pepin
- Flutter at Scale: A Roadmap for the Modern Enterprise — David DeRemer, Abdallah Shaban
- Flutter Migration at Scale: Migrate Tamara App in 10 Weeks — Karem Ebrahim
- Flutter at Enterprise scale at GEICO — Manaswi Daksha
- Hot Ones: The Spicy Truth About Migrating to Flutter — Leo Farias
- The Flutter Enterprise Journey: From Native Apps to AI-Enabled Multi-Platform — Mark Fairless, Tom Quercia
- From Vibe-Coded Viral MVP to Launchable Product in 90 Days — Óscar Martín, Liz Sweigart

### Agentic Engineering
- Self-Healing Flutter UI with Widgetbook: Teaching Coding Agents to Judge Their Own Work — Lucas Josefiak, Morgan Hunt
- Making AI Predictable: Skill-Driven Flutter Development — Ivanna Kaceviča
- Designer + Engineer + Claude: Figma → Flutter 跨職能 AI workflow — Rémy Baudet, Camila Mazer
- Next steps and new patterns for GenUI — Andrew Brogdon
- Flight Mode AI: Building Local LLM Apps Easily with LlamaDart — Jhin Lee
- Your Next Flutter Teammate Is Not Human — Andrea Della Porta, Vadym Pinchuk
- Building an Integrated AI-Powered Development Stack — Rui Alonso, Jorge Coca, Dominik Šimoník
- Building Modern Web Apps with Jaspr, Gemini and Antigravity — Argel Bejarano, Anya Isabel Arguelles Pesqueira
- Next-level Generative AI: Moving Beyond Copilots to a Multi-Agent Autonomous Workforce — Steven Stamps
- Is Flutter a Smart Bet in an AI-Driven Job Market? — Ivanna Kaceviča

### 平台邊界
- Desktop windowing APIs — Loïc Sharma（Google）, Matt Kosarek（Canonical）
- The JavaScript Exit: Flutter Web Beyond JavaScript — Andrea Della Porta
- Flutter on Big Screen & Beyond: Building Embedder for LG webOS — Jin Ho Chung, Yunsoo Kim
- Bare Metal Flutter: Raspberry Pi, GPIO, and the Embedded Frontier — Matthew Jones
- Flutter on Foldables: How hard could it be? — Vadym Pinchuk
- Bringing hot reload to your full Dart stack — Viktor Lidholt（Serverpod）
- Hook, Line, and Sinker: Dart Hooks and FFI in Production — Jon Holtan
- Effective Dart Admin SDK in Firebase Dart Functions — Alexander Nohe
- Meta-Flutter: Architecting a Package to Automate the IDE — Kali
- Bridging Ecosystems: Embedding Flutter Components in a Web App — Hudson Proenca, Karlo Verde
- Become a Full + Video Stack Dev — Simon Auer
- Building a Production Viral Video Feed in Flutter — Hugo Walbecq, Dominik Šimoník

### 其他
- From Lottie to Flutter: How Etsy Rebuilt Year in Review with Flutter Native Animations — Jorge Coca, Filip Maj
- Don't Break the Brand: Maintaining a Design System in Flutter — Bettina Carrizo, Mauricio Miguez, Valentina Llavayol
- Beyond the Code: Identity, Burnout & Resilience — Amanda Lentz
- Full Stack AI features（Workshop）— Alexander Nohe, Arthur Thompson, Rosário Fernandes

## 6. 企業案例（都有數字）

- **SoFi 三連發**（官方 YouTube）
  - 06-25 How SoFi deleted a million lines of code switching to Flutter — https://youtu.be/RBObC8T7ud0
  - 07-08 4 out of 5 devs at SoFi prefer Flutter over native development — https://youtu.be/yJts6W-tMmg
  - 07-15 SoFi built a super app using Flutter — https://youtu.be/kvl25HdOo2U
- **Knowunity** 07-23 — Flutter + BigQuery + Gemini 撐 10M MAU
  - https://youtu.be/LOOc0zlmCQ8
  - 同批：This AI studying app with 25M+ downloads is built with Flutter — https://youtu.be/sfBYTVI2Cd0
- **Etsy** — Year in Review 從 Lottie 改用 Flutter 原生動畫（FlutterCon 議程）

## 7. 官方 YouTube 本月上片

| 日期 | 標題 | 連結 |
|---|---|---|
| 07-24 | What pub package can't you live without? | https://youtu.be/bmcPkjivVxI |
| 07-23 | How Flutter, BigQuery & Gemini power Knowunity's 10M MAU AI learning app | https://youtu.be/LOOc0zlmCQ8 |
| 07-23 | This AI studying app with 25M+ downloads is built with Flutter | https://youtu.be/sfBYTVI2Cd0 |
| 07-17 | Name that Google Developer mascot | https://youtu.be/Fy7XTpBAxGA |
| 07-15 | SoFi built a super app using Flutter | https://youtu.be/kvl25HdOo2U |
| 07-14 | Iterable (Technique of the Week) | https://youtu.be/i-cTuLn9KNo |
| 07-10 | First programming language learned vs. favorite | https://youtu.be/XAQe22mCDBM |
| 07-10 | VGV's AI toolkit — Observable Flutter #92 | https://youtu.be/zhiK0YLLIp0 |
| 07-08 | 4 out of 5 devs at SoFi prefer Flutter over native | https://youtu.be/yJts6W-tMmg |
| 07-01 | Flutter's streamlined onboarding process | https://youtu.be/gr-1CHY7hYM |
| 06-30 | SliverSemantics (Widget of the Week) | https://youtu.be/lPWrd08swlw |

## 8. Serverpod 4 Public Beta（07-08）

https://serverpod.dev/blog/serverpod-4-public-beta · https://github.com/serverpod/serverpod
Demo：https://youtu.be/NwhfB8lgyxA

- 自稱「第一個會全端 hot reload 的 agentic coding engine」
- `serverpod start` 託管 server + database + web，改動即時 hot reload，DB migration 在 server 執行中套用
- 內建 agent skills + MCP server
- 100+ 新功能：社群登入、DB geography 型別、改良 web server、client-side database、model 動態型別、improved future calls
- **offline-first sync engine 沒趕上 beta**，留到 stable

## 8.5 Flutter Scene — 即時 3D 引擎

網站 https://fscene.dev/ · GitHub https://github.com/bdero/flutter_scene （⭐537）· pub.dev/packages/flutter_scene
作者 **bdero**（Brandon DeRosier）— 就是當初在 Flutter Engine 內部做出 Flutter GPU 的工程師。

> Scene 是完整的 3D 引擎，不是 renderer wrapper。渲染、物理、音訊、工具、完整資產管線，全部是 Dart package，跑在 Flutter 支援的每個平台。

### 七月版本節奏

| 日期 | 版本 |
|---|---|
| 07-02 | `flutter_scene_box3d` 0.1.0 |
| 07-17 | `flutter_scene` 0.19.0 |
| 07-27 | `flutter_scene` 0.20.0 ＋ `scene` 0.1.0 ＋ `flutter_scene_rapier` 0.3.0 |

07-27 那天的三包齊發是結構性重構：核心 scene document 抽成獨立 `scene` package、物理解耦成純模擬合約（可插拔後端）、粒子系統轉為公開 API。

### 能力清單（來自 fscene.dev）

- **後製棧**：bloom、SSR、ambient occlusion、god rays、帶 bokeh 的景深、霧、自動眼睛曝光、抗鋸齒與解析度縮放
- **打光**：image-based lighting、程序化物理天空（可瞄準太陽）、無上限點光/聚光（per-light culling）、軟陰影
- **物理**：一個 component 接上剛體；角色控制器、關節、載具、動態堆疊
- **粒子與 VFX**：flipbook 火焰、curl noise 煙、拉伸火花、ribbon trail、instanced mesh 碎片
- **Gaussian Splat**：`.ply` / `.splat` 實拍捕捉當一般節點載入，與一般幾何正確互相遮擋
- **3D 表面上的活 widget**：真的 Flutter widget 貼在 mesh 上，按鈕可按、slider 可拖、動畫繼續跑
- **全部熱重載**：改模型、shader、貼圖、環境都即時生效，不重啟、不掉 state
- **編輯器 ＋ MCP**：桌面場景編輯器內建 MCP server，AI agent 用跟人一樣的可還原指令改場景
- **無障礙**：semantics component 讓 3D 節點內容流進 Flutter accessibility tree
- **六個平台**（含 web，Skwasm / CanvasKit 皆可）＋ 自訂 embedder · 40 個範例

### 七月影片（youtube.com/@bdero）

| 日期 | 標題 | 連結 |
|---|---|---|
| 07-26 | This campfire is 100% procedural. No assets, just Flutter | https://youtu.be/jIllTfjLJzA |
| 07-23 | Dashsurfers, a 3D endless runner built entirely in Flutter（120 fps） | https://youtu.be/yMXrCAcmM8U |
| 07-23 | This endless runner is a Flutter app | https://youtu.be/9fJRsSumZAs |
| 07-21 | Flutter Scene in 90 Seconds: Real-Time 3D for Flutter | https://youtu.be/Vzn_yidhAKA |
| 07-21 | All of this is real-time 3D running in Flutter | https://youtu.be/793YonfStFs |

動圖素材來源：https://github.com/bdero/flutter_scene_media （已下載到 `assets/scene/`）

## 9. Reddit r/FlutterDev 當月熱門（top / month）

| # | 日期 | 標題 | 連結 |
|---|---|---|---|
| 1 | 07-14 | Flutter is such a godsend for Linux app development | https://www.reddit.com/r/FlutterDev/comments/1uwiuyt |
| 2 | 07-12 | Flutter Now Supports LG webOS. I Think This News Is Much Bigger Than We Realize | https://www.reddit.com/r/FlutterDev/comments/1uuh90d |
| 3 | 07-22 | Why we built our own Flutter runtime | https://www.reddit.com/r/FlutterDev/comments/1v3k1oj |
| 4 | 07-21 | How I reduced iOS simulator RAM usage by up to 4× | https://www.reddit.com/r/FlutterDev/comments/1v28e9e |
| 5 | 07-08 | We optimized our app launch to do less and it got 70% faster | https://www.reddit.com/r/FlutterDev/comments/1uqra6e |
| 6 | 07-08 | Full-stack hot reload — server, website, web, and app — is now a thing 🚀 | https://www.reddit.com/r/FlutterDev/comments/1uqqe5m |

深讀推薦：**Why We Built Our Own Flutter Runtime** — Nowa（Raed Abdallah），07-22
https://nowa.dev/blog/why-we-built-our-own-flutter-runtime/
> 核心問題：怎麼在 Flutter app 裡跑另一個 Flutter app。release mode 沒有編譯器，所以 Shorebird 得改引擎才做得到 code push。文章比較 FlutterFlow 的自訂格式路線 vs. 自建 runtime 路線，並點出「AI 不擅長寫自訂格式」是前者的致命缺口。

## 10. 社群開源專案（本月新面孔）

| 專案 | 說明 | 日期 |
|---|---|---|
| 純 Dart PDF 引擎 | 開源版 PSPDFKit / Syncfusion 替代 | 07-15 |
| kaisel 1.0 | sealed class + pattern matching 的 router，無字串、無 codegen | 07-17 |
| [gputext](https://github.com/definev/gputext) ⭐36 | flutter_gpu 的 GPU 向量文字 renderer | 07-07 |
| haptify | Dart CLI，音訊 → iOS/Android 觸覺回饋，可 runtime 執行 | 07-13 |
| GlyphPact | 本地 SVG → Flutter icon 編譯器，codepoint 穩定 | 07-25 |
| flutter_background_geolocation 開源替代 | 原版付費授權 | 07-14 |
| Alacritty(Rust) 驅動的 Flutter 終端機 widget | Dart FFI 接 Rust 實戰 | 07-02 |
| 開源可自架 session replay | 跟 console log 對時間軸 | 07-25 |
| 高度客製 QR code package | | 07-21 |
| Numberflow for Flutter | 原版動畫數字的高保真移植 | 07-09 |
| Bumbuild | 原生 macOS 的 Flutter build launcher | 07-24 |
| PicoView | 把 widget 鏡射到外接 LCD 觸控面板 | 07-19 |
| 跨平台 NES 模擬器 | | 07-12 |

## 11. 接下來的行事曆

| 日期 | 活動 | 地點 |
|---|---|---|
| 8 月（預計） | Flutter 3.47 stable | — |
| 08-07 | I/O Connect China | APAC |
| 08-12~13 | RenderATL | Atlanta, GA |
| 09-03~05 | Flutter & Friends | Stockholm, 瑞典 |
| 09-22~24 | FlutterConf LATAM | Cancún, 墨西哥 |
| 10-07~09 | next.app devcon | Berlin, 德國 |
| 10-19~20 | All Things Open | Raleigh, NC |
| 10-29~30 | FlutterKaigi / Flutter Ninjas | 東京 |

來源：https://flutter.dev/events

> **Flutter Connection（巴黎）**：官網 https://flutterconnection.io/ 目前仍停在 2025 年 4 月那一屆，**2026 場次尚未公布日期**。網路上流傳的「11 月 5–6 日」查不到官方出處；同期巴黎確定舉辦的是 Swift Connection（11/2–3）。等官方公告再更新。

---

## 查證備註

- Reddit 的官方 JSON API 與 firecrawl 都被擋，改走 `https://www.reddit.com/r/FlutterDev/top/.rss?t=month`（RSS 可通，且尊重 top 排序）
- FlutterCon 議程頁是 JS 渲染，firecrawl 拿不到，改用 agent-browser 取 accessibility snapshot
- 議程清單為 **Day 1（07-16）全房間**；Day 2 分頁在無頭瀏覽器點不動，Day 2 議程未完整收錄
- 版本資訊一律以 Flutter 官方 release JSON 為準，不採信二手文章
