# mamma-mia 的架构、内容管线与约定：对粤语学习 App（diu）的复用评估

> 范围：只看 mamma-mia 本身怎么做，以及每一部分搬到粤语 app 时是直接复用、改造还是替换。粤语书写系统、开放数据与语音技术清单、竞品、教学法、通用 app 架构由其他调研员负责，这里不展开。
> 仓库快照：`li1000v2/mamma-mia` 浅克隆 HEAD `f403849`（“Merge pull request #16 … audio-ok-screened”），本地路径 `/home/user/li1000v2/mamma-mia`。下文写成 `mamma-mia/<路径>` 的都是这个仓库里的文件，可以直接打开核对。网页结论附 URL。日期以 2026-10 为准。
> diu 仓库（`/home/user/Diu`）现在只有一个 `README.md`（“Cantonese learning”），没有代码。

---

## Q1 平台、编译和侧载管线（没有 Mac 的情况下）

### Takeaway
mamma-mia 的做法是：只用 iOS 26，界面用 SwiftUI，工程由 XcodeGen 生成；GitHub Actions 的 macOS runner 负责编译并产出未签名 IPA，再用免费 Apple ID 加 SideStore 侧载，每 7 天续签。这套和语言无关，diu 改名就能用。真正要提前规划的是免费账号最多 3 个侧载 app 的名额：SideStore、MammaMia、Diu 正好占满。

### Cited Findings
- 定位：自用的意大利语旅行学习 app，iOS 26 起，中文讲解，“12 个万能句框 × 旅行场景，30 课 + 10 个情境挑战，FSRS 间隔复习，示范音频离线可用”。联网时可以和 AI 扮演的角色对话，用自己的 Google key，只作加练（`mamma-mia/README.md`）。
- 没有本地 Mac：代码推到 GitHub，Actions 的 macOS runner 编译并产出未签名 IPA，用免费 Apple ID + SideStore 侧载（`mamma-mia/CLAUDE.md` 开头）。编译错误只能看 CI 日志（`gh run view --log-failed`）（`mamma-mia/CLAUDE.md`「编译与测试」）。
- `mamma-mia/project.yml`（XcodeGen）的要点：
  - `name: MammaMia`，`bundleIdPrefix: io.github.li1000v2`，`deploymentTarget iOS "26.0"`，`developmentLanguage: zh-Hans`，`MARKETING_VERSION "0.2.0"`；`MM_GIT_COMMIT` 由 CI 注入，显示在诊断页。
  - `Content/`、`Audio/` 以 folder 资源整个拷进 bundle。
  - 只有一个 application target，依赖本地包 `Packages/MammaCore`。
  - App 层 `SWIFT_VERSION "5.0"`，注释说“没有本地 Mac 来反复修编译错误”，所以放宽并发检查；MammaCore 用 Swift 6 模式。
  - `TARGETED_DEVICE_FAMILY "1"`（只支持 iPhone），竖屏，不支持 Mac Catalyst。
  - Info.plist 里麦克风、语音识别、仅添加照片三项权限说明都是中文。
- `.xcodeproj` 和 `App/Info.plist` 都被 gitignore，CI 每次重新生成；“只改 project.yml，不手改 .xcodeproj”（`mamma-mia/.gitignore`、`mamma-mia/CLAUDE.md`）。
- CI 一共 4 个工作流，都在 `mamma-mia/.github/workflows/`：
  - `content-check.yml`：ubuntu + Python 3.12，跑 `pip install -r tools/requirements.txt`（只有 `jsonschema>=4.23,<5`）和 `python tools/validate_content.py`。改了 `Content/**`、`Audio/**`、`tools/**` 时触发。
  - `core-tests.yml`：在 `swift:6.3-noble` 容器里跑 `swift test --package-path Packages/MammaCore`。改了 `Packages/**`、`Content/**`、`Audio/manifest.json` 时触发。
  - `ios-build.yml`：`macos-26`，Xcode `/Applications/Xcode_26.6.app`，XcodeGen 2.46.0，编模拟器版本（`CODE_SIGNING_ALLOWED=NO`）。改了 `App/**`、`Packages/**`、`project.yml` 时触发，同一分支会取消旧的运行。
  - `ipa.yml`：手动触发或推 `v*` tag。`xcodebuild archive` 不签名；`CURRENT_PROJECT_VERSION` 取 run_number，`MM_GIT_COMMIT` 取 SHA 前 7 位，tag 名决定 `MARKETING_VERSION`。打成 `Payload` zip 当 IPA，artifact 保留 14 天；tag 构建时用 `gh release create` 发 Release，带 `-` 的 tag 发成 pre-release。
  - 各工作流用的 action 都是 `actions/checkout@v7`、`setup-python@v7`、`upload-artifact@v7`。
- 工具链版本的约定：GitHub 上的 Xcode 27 还是预览版，没有用到 iOS 27 SDK 的需求就不换（`mamma-mia/CLAUDE.md`）。
- 侧载步骤（`mamma-mia/docs/sideload.md`）：
  - 第一次需要一台 Windows 电脑，装 iTunes 和 iloader；手机上装 LocalDevVPN，必须连 Wi-Fi，蜂窝数据不行。
  - 用 iloader 装 SideStore，在设置里信任开发者、打开开发者模式。
  - 之后在手机 Safari 里打开 Releases 页下载 IPA，用 SideStore 的 “+” 安装。
  - 硬性限制：免费账号最多同时装 3 个侧载 app（SideStore 自己算一个），每周最多新注册 10 个 App ID，签名 7 天过期（过期后打不开但数据还在），每次装新版也会刷新 7 天。
- 免费 Apple ID 带来的工程约束（`mamma-mia/CLAUDE.md`「免费 Apple ID 的限制」）：
  - 只有一个 target，不加任何扩展，因为小组件这类扩展也占侧载名额。
  - 不用 APNs、iCloud、App Groups，提醒用本地通知。
  - 不改 bundle ID，持久化格式只做加法改动，因为每周都要重装。
- 真机排错：没有 Xcode 控制台，就往 `AppLog` 写日志，用户在「今日 → 诊断与日志」里分享出来（`mamma-mia/CLAUDE.md`「工作方式」；`mamma-mia/App/Support/AppLog.swift`、`mamma-mia/App/Views/DiagnosticsView.swift`）。
- 签名到期提醒：`mamma-mia/Packages/MammaCore/Sources/MammaCore/Travel/DepartureCheck.swift` 读 SideStore 写进 app 的 `embedded.mobileprovision`，只取 `ExpirationDate`，不读 TeamName；有测试 `DepartureCheckTests.swift`。
- 开发流程：从分支开 PR 合进 main（`mamma-mia/CLAUDE.md`）。

### Inferences
- **直接复用**：`project.yml` 和 4 个 workflow 基本照抄，改这几处：
  - 名称改成 `Diu`，bundle ID 改成 `io.github.li1000v2.diu`（新的 App ID 会占一次每周 10 个的名额）。
  - 识别权限说明改成粤语版。
  - `ipa.yml` 里的 `MammaMia.ipa` 改名。
  - `docs/sideload.md` 可以基本照搬。
- **侧载名额**：SideStore + Mamma mia! + Diu = 3，名额就满了。从课程看意大利行程在 12 月（`course.json` 的 `events` 是 12/8 Immacolata，`cross_check.py` 也写着“12 月去意大利旅行”）。行程结束前两个 app 只能共存，不能再装第四个侧载 app。行程后可以卸掉 Mamma mia!；卸载会丢本机数据，卸之前先用「导出进度」备份。
- **签名到期提醒**：mobileprovision 解析和到期提醒是通用能力，建议从 `Travel/DepartureCheck.swift` 拆出来，在 diu 里做成独立的 `Signing/`，平时也提醒续签，不再挂在行程上。
- **Swift 模式**：继续“App 层 Swift 5、Core 层 Swift 6”。没有 Mac 时这是降低 CI 编译迭代成本的做法，值得保留。

### Gaps
- 没有核实 SideStore、LocalDevVPN 在 2026-10 最新的 iOS 26.x 上是否仍然可用，也没查是否有政策变化。mamma-mia 文档只写了“马上测一次续签”这一自检步骤。

---

## Q2 MammaCore 的包边界和 FSRS 集成

### Takeaway
MammaCore 是“只依赖 Foundation + swift-fsrs、能在 Linux 上测”的纯逻辑层。swift-fsrs 固定在 main 的一个提交上，为的是用 FSRS-6。调度、进度、Gemini 请求和错误分类、剧本规划这些可以原样搬到 diu。按意大利语写的分词和判定、提示词、句子切分要重写，旅行层看 diu 的目标决定留不留。卡型枚举里早就声明了没用上的 `minimalPair`，正好拿来做声调最小对立。

### Cited Findings
- 边界规则：MammaCore 只依赖 Foundation 和 swift-fsrs，不许 import UIKit、SwiftUI、SwiftData，保证 Linux 上能测（`mamma-mia/CLAUDE.md`「目录」）。`mamma-mia/Packages/MammaCore/Package.swift`：swift-tools 6.0，`platforms: [.iOS(.v17), .macOS(.v14)]`，单 target `MammaCore`，加一个测试 target。
- FSRS 版本：固定 `revision: "4fbaf20184d62f82a9f44f343337c61a2c5483e9"`，注释是“FSRS-6 is only on main; the last tagged release (v5.0.0) is FSRS-5”。升级时改 Package.swift 里的 revision（`mamma-mia/Packages/MammaCore/Package.swift`、`mamma-mia/CLAUDE.md`）。
- 调度器 `mamma-mia/Packages/MammaCore/Sources/MammaCore/Scheduling/ReviewScheduler.swift`：
  - 构造参数：`FSRS(parameters: FSRSParameters(requestRetention: config.desiredRetention, w: FSRSDefaults.defaultWv6, enableFuzz:, enableShortTerm: true, learningSteps:, relearningSteps:))`。
  - 默认值：desiredRetention 0.9，newCardsPerDay 20，learningSteps `["1m","10m"]`，relearningSteps `["10m"]`。
  - 有 `usesFSRS6` 属性；FSRS 类型留在包内，App 只依赖 MammaCore。
  - 注释写着“FSRS-6 scheduling with the default parameters until there is enough review history to fit our own”。
- 早先的调研也确认了 swift-fsrs README 支持 FSRS-6、需要显式传 `defaultWv6`，并确认工程已经这样做（`mamma-mia/docs/language-learning-github-research-2026-10-16.md` §12）。
- 卡片模型（同一文件）：
  - `CardKind` 有 `listen`（听辨）、`recall`（回忆：看中文意图说出来）、`swap`（换词）、`minimalPair`（最小对立：听两个近似词之一，选是哪个）。
  - 在整个仓库里 grep，`swap` 和 `minimalPair` 只出现在枚举声明里，没有地方用到。
  - `CardID` 的写法是 `phraseID#kind`。`Grade` 是 again、hard、good、easy 四档自评，映射到 FSRS Rating。
  - `DailyPlan` 分四组：due、boosted（按行程用到日加推）、packed（城市包）、new。
- 学习规则：只有复习自评会更新 FSRS 状态；上课只把卡加进来（新卡），挑战只记成绩。每句默认两张卡（listen、recall），各自调度，复习时同一句的两张卡不连着出（`mamma-mia/CLAUDE.md`「内容规则」）。
- 进度文件 `mamma-mia/Packages/MammaCore/Sources/MammaCore/Progress/StudyProgress.swift`：
  - 整个进度是一个 JSON 文件，同时也是备份格式。
  - 每个字段解码时都有默认值，因为“app 每周重装”。
  - `trip`、`practicePhrases`、`spotLogs` 这些可选部分读不出来时只丢弃那一部分；为空时不写这个键。
  - `ProgressStore.swift` 存到 Application Support 的 `progress.json`。
- 模块清单（`mamma-mia/Packages/MammaCore/Sources/MammaCore/`）：
  - `Audio/AudioManifest.swift`
  - `Content/`：`ContentBundle`、`ContentValidator`、`Course`、`Trip`
  - `Dialogue/`：`DialoguePlanner`、`DialogueScript`
  - `Progress/`
  - `RolePlay/`：`GeminiFailure`、`GeminiRequest`、`GeminiStream`、`JSONValue`、`RolePlayContent`、`RolePlayConversation`、`RolePlayFeedback`、`RolePlayPrompt`、`SentenceChunker`、`TurnAllowance`
  - `Scheduling/`：`NeedByPlanner`、`ReviewQueue`、`ReviewScheduler`
  - `Speech/`：`SpeechCheck`、`SpokenSentence`
  - `Study/StudyPlanner`
  - `Time/`：`LocalDate`、`StudyClock`
  - `Travel/`：`CityPack`、`DepartureCheck`、`ShowCards`、`Spots`、`TripDay`
  - 测试 13 个文件，共 138 个 `@Test`（Swift Testing），在 `mamma-mia/Packages/MammaCore/Tests/MammaCoreTests/`。
- `Study/StudyPlanner.swift`：所有内容都不锁，课可以按任意顺序上，挑战随时可以重试；它只负责指出下一课，并提示哪个挑战的前置课已经学完。

### Inferences
- **怎么复用**：建议把代码复制进 diu，改名 `DiuCore`，不和 mamma-mia 共用一个包。理由是自用、两边改动节奏不同，而且 mamma-mia 12 月行程后基本冻结。要是以后第三门语言也要用，再把通用部分抽成独立 SPM 包。
- **逐模块处理**：

| MammaCore 模块 | diu 处理 | 说明 |
| --- | --- | --- |
| `Scheduling/ReviewScheduler`、`ReviewQueue` | 原样复用 | FSRS-6、自评、卡型分开调度、同句卡错开，都和语言无关 |
| `Time/`、`Dialogue/`、`Study/StudyPlanner` | 原样复用 | 剧本规划（followUps 随机抽取）与语言无关 |
| `RolePlay/GeminiRequest`、`GeminiStream`、`GeminiFailure`、`JSONValue`、`TurnAllowance`、`RolePlayConversation` | 原样复用 | 端点、SSE 解析、错误分类、每日上限 |
| `Progress/StudyProgress` | 改造 | 去掉 `trip`、`spotLogs`，或保留为可选；保持“只做加法”的解码策略；可加显示偏好（繁/简、是否显示粤拼） |
| `Audio/AudioManifest` | 改造 | key 要带上粤拼，见 Q9 和 Q14 |
| `Content/Course`、`ContentBundle`、`ContentValidator` | 改造 | 字段 `it` 改成 `yue`、`jyutping` 等；新增粤拼与汉字逐一对齐的校验 |
| `Speech/SpeechCheck`、`SpokenSentence` | 重写分词和归一化，保留对齐算法和判定档位 | 见 Q11 |
| `RolePlay/RolePlayPrompt`、`SentenceChunker`、`RolePlayFeedback` | 重写或改造 | 两行输出都是汉字，原来的切分规则失效，见 Q12 |
| `Travel/`（`Trip`、`CityPack`、`NeedByPlanner`、`Spots`、`ShowCards`、`TripDay`） | 看 diu 的目标 | 有港澳或广州行程就保留并改城市；没有就删掉，`neededAt` 改成可选 |
| `Travel/DepartureCheck` 里的 mobileprovision 解析 | 拆出来复用 | 做成通用的续签提醒 |
| 新增 `Cantonese/` | 新写 | 粤拼解析和校验、汉字切分（含扩展区）、异体与俗写归一、繁简映射表加载、粤语数字和金额 |
| 新增 `Tone/` | 新写，可选 | 基频（F0）曲线提取与归一化，只用来对比，不打分，见 Q11 |

- `CardKind.minimalPair` 现成就是“听一个、选是哪个”，可以直接用于声调对立（同音节不同声调，比如 si1、si2、si3、si4、si5、si6）。这是这个枚举第一次真正派上用场，FSRS 调度不用改。
- 内容里出现的新卡型（比如 `tone`）要按“持久化只做加法”的规则加进 `CardKind`：旧备份读不到新值没关系，但不能改已有 rawValue。

### Gaps
- 没查 swift-fsrs 在 2026-10 是否已经发出带 FSRS-6 的正式 tag。mamma-mia 的 `CLAUDE.md` 说“最新 tag v5.0.0 还是 FSRS-5”，diu 开工时要再看一次。

---

## Q3 内容 JSON Schema、课程结构和内容状态流水线

### Takeaway
mamma-mia 的内容是“课程图（句框 × 场景 × 单元）+ 每课句子 + 挑战剧本 + 在线角色设定”四类 JSON，都由 JSON Schema 约束，再加 Python 和 Swift 两层跨文件校验。每句都有一条状态流水线 `draft → checked → native → audio_ok`，靠 `checkedText` 把审核结论锁在审过的那段原文上。这个结构可以直接搬到粤语。要改的是字段：把 `it` 换成“粤语口语字 + 粤拼”，并且校对锁要同时锁住汉字和读音。

### Cited Findings
- `mamma-mia/Content/schema/course.schema.json` 的结构：
  - 顶层是 `version`、`frames`、`scenes`、`cities`、`events`、`units`。
  - `frame` 有 `id`、`pattern`、`use`、`example`。
  - `scene` 有 `id`、`zh`、`neededAt`，取值匹配 `^(first|last|city:[a-z]+|event:[a-z0-9_-]+)$`。
  - `unit` 有 `id`（`^[LC][0-9]{2}$`）、`kind`（lesson 或 challenge）、`week`（1–5）、`title`、`focus`、`frames[]`、`scene`、`summary`。
  - 全部 `additionalProperties: false`。
- `mamma-mia/Content/course.json` 的实际规模：
  - 12 个句框：prendo、posso、mipuo、dove、quanto、avete、ache、bisogno、passato、sono、non、numero，比如 `"Posso…?"` 用于“问能不能”。
  - 16 个场景：general、bar、boat、hotel、restaurant、train、pizzeria、holiday、directions、metro、sights、shopping、pharmacy、emergency、taxi、smalltalk。
  - 3 座城市（venezia、napoli、roma），1 个节日（immacolata，12 月 8 日）。
  - 30 课 + 10 个挑战，分 5 周，大约每 3 课接一个挑战。前两课专讲发音，L07、L16 是语法小结。
- `mamma-mia/Content/schema/lesson.schema.json`：
  - 顶层 `unit`（`^L[0-9]{2}$`）、`intro`（中文微课）、`phrases[]`。
  - 每句 `{id: ^L[0-9]{2}-[0-9]{2}$, it, zh, frame?, scene, status: draft|checked|native|audio_ok, note?, checkedText?}`。
  - `checkedText` 的说明是“Must equal it unless status is draft, so an edited phrase cannot keep its old review”。
- 实例 `mamma-mia/Content/lessons/L01.json`：
  - intro 是长篇中文发音讲解，借日语假名、英语、拼音搭桥，每条都写“哪里不像”。
  - 句子如 `"it": "Ciao!"`、`"status": "audio_ok"`、`"checkedText": "Ciao!"`，note 里写音节划分和用法。
- 当前状态统计（对 `Content/lessons/*.json` 计数）：83 句 audio_ok，67 句 checked，0 句 draft。
- `mamma-mia/Content/schema/dialogue.schema.json`（离线挑战剧本）：
  - 顶层 `unit`（`^C[0-9]{2}$`）、`title`、`goal`、`speaker`、`voice?`、`status`、`nodes[]`、`followUps?{after, min, max, pool[]}`。
  - 每个 node 是 `{id, npc[]: {it, zh, answer?, distractors?}, question?, reply?: {intent, frame?, choices[≥2]: {it, correct, feedback?}}}`。
  - 例子 `mamma-mia/Content/dialogues/C02.json`：报价那一幕有三个变体，比如 “Sono tre euro e cinquanta.” 的 answer 是 “3.50 €”，干扰项是 “3.15 €”、“13.50 €”；错误选项的 feedback 用中文解释错在哪。
  - 挑战每次运行都会重新洗牌追问、台词变体和选项（`mamma-mia/App/Views/ChallengePlayerView.swift` 文件头注释）。
- `mamma-mia/Content/schema/roleplay.schema.json` 和 `Content/roleplay.json`：
  - 顶层 `fillers[]`（“Allora…”、“Vediamo…”、“Dunque…”）和 `roles[]`。
  - 每个角色 `{unit, name（意大利语）, zh, voice, scene, opens, setting（给模型的英文设定）, task（给学习者的中文任务）}`，比如 C02 的收银员。
- 旅行层：`showcards.json`（出示卡句子和欧盟 14 类过敏原意大利文名称）、`spots.json`（巡礼点的公开地名、地址、坐标和句子）、`trip.example.json`（假日期）。对应的 schema 在 `Content/schema/`（`mamma-mia/CLAUDE.md`「随身」）。
- 状态规则（`mamma-mia/CLAUDE.md`「内容规则」）：
  - 没校对过的不许标 checked。
  - 非 draft 的句子必须带 `checkedText` 且等于 `it`，`validate_content.py` 会拦。
  - 课程句标 audio_ok 要同时满足三条：已经 checked；拼写里没有双辅音；该句每段音频（每个声线，原速和慢速）在 `reviews/audio-screen-cache.json` 里都有当前有效、判为干净的粗筛结果。
  - 含双辅音的句子留在 checked，等人听过再标。音频重生成或重筛后不再干净，就退回 checked。
- 发音讲解约定（`mamma-mia/CLAUDE.md`「内容规则 → 发音讲解」）：
  - 先听示范音频，标准永远是音频，再加上和自己录音的对比。
  - 搭桥只讲“某个音像什么”，可以借假名、拼音、英语，但不给整个词的假名或拼音转写；每个搭桥后面跟一句“哪里不像”。
  - 要明确提醒母语干扰。拼音只借“b、d、g 不送气”这一点。
- 其他内容约定：
  - 只教“怎么问”，不写死票价、营业时间这类会变的事实。
  - 句子里不写用到日，由场景的 `neededAt` 和行程推算。
  - 单元 id 用 `Lxx`、`Cxx`，句子 id 用 `Lxx-nn`，文件名要和 `unit` 一致。
  - 对陌生人一律用 Lei，自述句用阳性。
  - （以上见 `mamma-mia/CLAUDE.md`）
- 交叉校对报告写到 `reviews/cross-check-<单元>.md`，工具不改内容文件；人按报告手动改完，确认没问题才标 checked，同时把 `checkedText` 设成当时的 `it`（`mamma-mia/CLAUDE.md`「交叉校对」；示例 `mamma-mia/reviews/cross-check-L05.md`：“6 条：6 条 OK”）。

### Inferences
- **课程结构可以直接用**。“句框 × 场景”放到粤语旅行或日常生存场景里同样成立。粤语句框的例子：唔該…、我想要… / 俾…我、…幾多錢？、…喺邊度？、有冇…？、可唔可以…？、幾點…？、我…咗、係唔係…？、…定…？、一個 / 兩個…（量词）。场景可以是茶餐廳、港鐵、巴士 / 小巴、的士、酒樓飲茶、街市、便利店、問路、購物、藥房、求助、閒聊。具体课程设计和教学法由其他调研员负责，这里只说明 `course.schema.json` 不用改结构，`frames` 和 `scenes` 换内容即可。
- **状态流水线和“审核结论锁原文”要保留，而且要扩大锁的范围**。粤语句子的“原文”至少包括汉字和粤拼两部分，读音错了和字错了一样要重审。所以 `checkedText` 要同时覆盖 `yue` 和 `jyutping`，具体写法见 Q14。
- **约定要改写**：
  - “Lei / 阳性”这条换成粤语的语域规则，比如“唔該”和“多謝”分场合用；称呼用先生 / 小姐；粤语里基本不用“您”。
  - 发音讲解的搭桥规则可以沿用框架（像什么，再写哪里不像，最后以音频为准），内容换成针对“普通话 + 日语”母语者的粤拼陷阱。比如粤拼的 j、z、c 和拼音字母同形不同音；入声 -p、-t、-k 不除阻，可以借日语促音っ和音读词尾 -ツ、-ク 来感受，再说明哪里不像。这部分属于教学法，只作为约定框架的映射。
- `dialogue.schema.json` 的 `answer` / `distractors` 很适合粤语数字和金额听辨，比如 三十五蚊 / 五十三蚊 / 三蚊半，以及 兩 / 二、廿 / 二十 这类陷阱。

### Gaps
- diu 的学习目标没在材料里看到：是港澳或广州旅行、看港片和 TVB、家人交流，还是纯兴趣？这决定 `neededAt`、行程、出示卡、巡礼点这几层留不留。mamma-mia 的巡礼层是按 JoJo 第五部取景地做的（L23 的 “Sono un fan di JoJo”，以及 `CLAUDE.md` 里“巡礼层只用地名…不放原作截图”），粤语版有没有类似的港片取景地需求，不得而知。

---

## Q4 示范音频管线：生成、自动转写校验、AI 粗筛、重生成、人工听审，以及 Google Cloud 项目被停用的教训

### Takeaway
音频管线分五级：
1. Gemini TTS 生成原速，ffmpeg 放慢出慢速；
2. 用另一个 Gemini 模型转写回来，和原文比对，对不上就重试；
3. 高思考档的 Gemini 逐段听审，结果缓存并和音频哈希绑定；
4. 被标出的录音最多重生成 4 个版本，复筛两次都干净才替换；
5. 人工听最后的可疑片段，以及模型听不准的类别（双辅音）。

整套设计在于承认模型在哪些地方不可靠，并在流程里绕开。2026-10-16 因并发过高，Google Cloud 项目被停用，之后所有工具的并发都压在 8 路以内，遇到 401/403/404 立即停，不许换项目绕过。搬到粤语时，“双辅音需人工听”这条要换成“声调、多音字、文白读需人工听”。

### Cited Findings
- 配置 `mamma-mia/tools/tts_voices.json`：
  - `"ttsModel": "gemini-3.8-flash-tts"`，`"checkModel": "gemini-3.8-flash"`，`"slowTempo": 0.75`。
  - 两个声线：f1 = Kore、m1 = Charon，各带 `rev`；`showCardVoices`、`spotVoices` 只用 f1。
  - `lineVoices` 把个别句子限定到某些声线。
  - 改了声线就把 `rev` 加一，对应音频全部重生成（`mamma-mia/CLAUDE.md`「示范音频」）。
- 生成 `mamma-mia/tools/gen_audio.py`：
  - 端点 `https://aiplatform.googleapis.com/v1/projects/{project}/locations/global/publishers/google/models/{model}:generateContent`。
  - key 从环境变量或 `.env` 读（`GOOGLE_API_KEY`、`GOOGLE_CLOUD_PROJECT_NUMBER`）；云端环境可以注入 `x-goog-api-key` 请求头；报错信息里会把 key 替换掉。
  - 请求体：`responseModalities: ["AUDIO"]`，`prebuiltVoiceConfig.voiceName`，只发原文。原因是“实测带指令时模型会把指令也念出来，systemInstruction 也不支持”（`mamma-mia/CLAUDE.md`）。
  - 每句最多生成 3 次，每次都用 checkModel 转写。转写提示词是 “Transcribe exactly every word spoken in this audio, in whatever language it is. Write numbers… as words… Output only the transcript.”，temperature 0。比对方法是对 `normalized()` 后的两段文字算 `difflib.SequenceMatcher` ratio，≥ 0.85 就通过；3 次都不过就输出 warning，需要人去听。
  - 慢速是把同一段原速录音用 ffmpeg `atempo=0.75` 放慢；先用 `silenceremove` 去掉首尾静音，再编成单声道 AAC 48k 的 m4a。
  - 文件名是 `sha256("<voice>|<speed>|<text>")[:16].m4a`，manifest 的键 `"<voice>|<speed>|<text>"` 必须和 `AudioManifest.key` 一致（`mamma-mia/Packages/MammaCore/Sources/MammaCore/Audio/AudioManifest.swift`）。
  - 参数 `--dry-run`、`--prune`、`--limit`、`--workers`（被压在 `MAX_PARALLEL = 8` 以内）；429 和 5xx 用指数退避重试。
  - 句子里允许写阿拉伯数字，比对前会用 `italian_number()` 把数字拼成意大利语单词。
- 规模：`Audio/` 下 1203 个文件，约 24 MB（本地 `du -sh`）。
- AI 粗筛 `mamma-mia/tools/audio_review.py`：
  - 模型 `gemini-4-argon`，`thinkingConfig.thinkingLevel: HIGH`。
  - 结果缓存在 `reviews/audio-screen-cache.json`，连同音频内容的 SHA-256：重新生成的音频文件名不变，内容变了就会重筛。改提示词时把 `PROMPT_REV` 加一，旧结果全部重筛。
  - 可疑问题分 9 类：misread、omission、extra_speech、stress、prosody、noise、truncation、slow_distortion、other；每条问题的 detail 和整段的 reason 要求用简体中文写。
  - 模型说双辅音的话用正则 `GEMINATION_TALK` 丢掉，含双辅音的句子（正则 `DOUBLE`，cq 也算）统一备注“双辅音需人工听”。理由是“实测会自信地判错”。
  - 不可用的答案不进缓存：拥堵时模型偶尔不思考，或者没听音频就判 ok 而转写和原文毫不相干，这种会换成 WAV 再试一次。被标出的片段再用 checkModel 转写一遍写进报告，因为 Argon 偶尔会编出不相干的整句。
  - 实测单价：一段约 7–8 秒，约 US$0.004（`USD_PER_CLIP`）。
  - 报告是 `reviews/audio-screen-week<N>.md` 加同名 `.html`，可以在浏览器里逐条点听。第 1 周的样例 `mamma-mia/reviews/audio-screen-week1.md`：208 段（原速 104、慢速 104），可疑 0 段，含双辅音 24 句，token 输入 136,422、输出 22,587、思考 109,320，花费约 $0.83。
- 重生成 `mamma-mia/tools/regen_audio.py`：
  - 一条录音指一个声线说一句，原速和慢速一起判。
  - 先复筛现在的版本，两次都干净就保留（原来是误报）；否则最多生成 4 个新版本，复筛两次都干净才替换；都不干净就留最好的一版，记为仍可疑。
  - 决定记在 `reviews/audio-regen-log.json`，报告是 `reviews/audio-regen.md`（`mamma-mia/CLAUDE.md`）。
  - 实际结果（`mamma-mia/reviews/audio-regen.md`）：处理 127 条，换成新版本 82 条，保留原版（误报）42 条，仍可疑 0 条，改为只用另一个声线 3 条。比如 “All'aeroporto di Fiumicino, per favore.” 的 Charon 版本次次读错，其中一次“生成失败（服务按使用条款拒绝请求，HTTP 403）”，最后改为只用 Kore（`lineVoices`）。
- 并发和停用的教训（`mamma-mia/CLAUDE.md`「示范音频」；`mamma-mia/tools/gen_audio.py` 注释）：
  - 2026-10-16 一天发了几千次请求、最高 40 路并发，之后项目被 Google Cloud Trust & Safety 以“违反服务条款”为由停用（HTTP 403）。
  - 此后所有工具共用 `MAX_PARALLEL = 8`，`--workers` 都被压在它以内。
  - 遇到 401/403/404 立即停；“不要换项目或换账号绕过封禁”。
- App 里的兜底：没有音频的句子退回 iOS 自带意大利语语音（`AudioService.swift` 里用 `AVSpeechSynthesisVoice(language: "it-IT")`）；挑战剧本可以用 `voice` 指定角色声线（`mamma-mia/CLAUDE.md`；`mamma-mia/App/Services/AudioService.swift`）。

### Inferences
- **整体形状直接复用**：生成、转写校验、粗筛缓存（和音频哈希绑定）、重生成（两次干净才换）、HTML 点听报告、`lineVoices`、`rev` 机制都不用改。按 mamma-mia 的实测，粗筛一次全量大约是“每 200 段 1 美元以内”的量级，粤语课程规模相近的话成本可控。
- **并发规则原样继承，再收紧一点**：
  - `MAX_PARALLEL = 8` 和“401/403/404 立即停”照搬。
  - 另外建议加每分钟请求数上限和单次运行的总请求预算。当初停用前的触发条件是“一天几千次 + 40 路并发”，只限并发不限总量，大批量重生成时仍有风险。
  - diu 用哪个 Google Cloud 项目要先确认：如果还是那个被停用的项目，要先申诉恢复；按 mamma-mia 自己的规则，不能新开项目绕过。
- 粤语专项的具体改法放在 Q10。

### Gaps
- 仓库里没有记录被停用的项目后来是否恢复，也没有记录 TTS 本身每段的费用（只有粗筛的单价）。

---

## Q5 离线语音识别和“听不听得懂”的判定逻辑

### Takeaway
mamma-mia 用 SpeechAnalyzer + SpeechTranscriber 做离线识别（设备不支持就退到 DictationTranscriber），把识别结果和目标句逐词对齐，只判断“对方能不能听懂”，不打发音分。识别结果只是提示，复习照样自评。对齐算法和判定档位可以复用。但它的分词按“连续字母”切，一句没有空格的中文会被当成一个词，所以粤语必须重写分词和归一化。

### Cited Findings
- `mamma-mia/App/Services/SpeechService.swift` 的流程：
  - 先试 `SpeechTranscriber(locale:, preset: .transcription)`，不行再试 `DictationTranscriber(locale:, preset: .shortDictation)`。
  - 用 `supportedLocales` 选最接近 it_IT 的 locale。模型通过 `AssetInventory` 下载并 `reserve`；别的 app 下载过的模型，本 app 仍要自己 reserve。
  - 识别时先让 analyzer 直接读文件，失败或什么都没听到，再按 `bestAvailableAudioFormat` 转换成 buffer 重试。
  - 两个引擎都不支持时，只保留录音对比和自评。状态和下载进度显示在「诊断、日志与备份」。
  - `recognize(_:logText:)` 在自由对话时不把文字写进日志。
- 判定 `mamma-mia/Packages/MammaCore/Sources/MammaCore/Speech/SpeechCheck.swift`：
  - 用词级编辑距离对齐目标句和识别结果，gap 代价 10。替换代价按拼写相似度算：相似度 ≥ 0.5 时为 `2*gap - min(9, round(sim*10))`，记成 “heardAs”（听成了某个近似词），否则当缺词。
  - 识别器偶尔会调换词序（比如 “1,20 €” 对应 “un euro e venti”），所以对齐后仍缺的目标词，还会去匹配没用过的相同听到词。
  - 判定档位：`clear` 全部听到；`mostly` 覆盖率 ≥ 60%；`unclear`；`silent` 什么都没听到。
  - 注释写着：it answers “would the other person understand?”, not “how good was the accent?”。
- 归一化 `mamma-mia/Packages/MammaCore/Sources/MammaCore/Speech/SpokenSentence.swift`：
  - 不分大小写，去掉重音符和撇号。省音词可以前缀匹配（dov' 对 dove）。阿拉伯数字和意大利语数词等价，价格 “3,50 €” 读作 “3 euro e 50”。
  - 分词规则是 `character.isLetter || character.isNumber` 时累积成一个词，遇到其他字符就断开。
- `ReplyMatcher`（同在 `SpeechCheck.swift`）：
  - 一个选项“合得上”的条件是：至少一半的词被听到，并且转写里没有只属于其他选项的词。
  - 有几个都合得上时，取完整听到的那个；还是分不出就让用户点选。
  - 内容约定是“选项之间要差至少一个听得出的词，只差标点或重音的两个选项只能点选”（`mamma-mia/CLAUDE.md`「开口说」）。
- 识别只是提示，复习照旧自评打分；挑战里说出的回复按选项的词匹配，对不上就点选（`mamma-mia/CLAUDE.md`；`mamma-mia/App/Views/ReviewSessionView.swift` 注释：“It's only a hint: the grade is still the learner's own”）。
- 上课流程（`mamma-mia/App/Views/LessonPlayerView.swift` 文件头注释）：微课 → 听（原速，再慢速，最后才出字幕）→ 跟读并和自己的录音对比 → 看中文意图说出来 → 结束。“说出来”这一步的答案会和离线识别听到的内容对齐展示。

### Inferences
- **分词是粤语版必须重写的地方**。我读代码的结论是：Swift 里汉字的 `isLetter` 为真（Unicode Lo 类），所以 “唔該一杯凍檸茶” 会成为一个词，只有标点能把它切开，`SpeechCheck` 就退化成按分句的全对或全错。粤语版要改成：每个汉字算一个 token（基本对应一个音节）；拉丁字母串（OK、Wi-Fi、D）算一个词；数字串交给粤语数字归一化；全角标点当分隔。详见 Q11。
- 对齐算法（编辑距离加近似替换）、判定四档、“识别只是提示”、ReplyMatcher 的“独有词排除”思路都能复用。只是“词”的单位换成字或音节，“近似”的度量从拼写相似改成读音相似（粤拼）。

### Gaps
- 无。

---

## Q6 在线 AI 角色扮演（用户自己的 Google key）

### Takeaway
每个挑战配一个在线角色。一回合的流程是：录音 → 离线识别，同时播语气词 → Gemini 流式回复 → 按句切开 → Gemini TTS 流式合成、边收边播。key 存在钥匙串里且只限本机，每天有回合上限，结束后的反馈卡会把纠正的句子转成复习卡。工程部分（请求、SSE、错误分类、上限、钥匙串）和语言无关；提示词、句子切分和反馈 schema 要按粤语重写。

### Cited Findings
- 流程（`mamma-mia/CLAUDE.md`「在线 AI 角色扮演」）：
  - 录音 → 离线识别，同时用角色声线播语气词 → Argon（`thinkingLevel` LOW，流式）。如果 6 秒没出字，或者出字前遇到 429/5xx，就改问 Gemini 3.8 Flash；设置里也能直接选 Flash。
  - 回复由 `SentenceChunker` 按句切开，只念第一行的意大利语，第二行是中文翻译。
  - 每句送 gemini-3.8-flash-tts 流式合成，边收边播（`StreamingSpeechPlayer`，24 kHz 16 位 PCM，实为小端）。合成失败的句子用 iOS 自带语音念。
  - live 声线名从 manifest 的 `voiceNames` 读，和示范音频是同一个声音。
- 模型 id 写在 `mamma-mia/Packages/MammaCore/Sources/MammaCore/RolePlay/GeminiRequest.swift`：`argon = "gemini-4-argon"`、`flash = "gemini-3.8-flash"`、`speech = "gemini-3.8-flash-tts"`，反馈卡固定用 Argon。端点 `https://aiplatform.googleapis.com/v1/projects/<project>/locations/global/publishers/google/models/<model>`，后缀 `:streamGenerateContent?alt=sse` 或 `:generateContent`。Argon 只收 LOW、MEDIUM、HIGH，传 `thinking_budget` 会 400（`mamma-mia/CLAUDE.md`）。
- 提示词 `mamma-mia/Packages/MammaCore/Sources/MammaCore/RolePlay/RolePlayPrompt.swift`：
  - 告诉模型学习者的话是语音识别结果，可能有错，要宽容理解。
  - A1–A2、每回合最多 15 词（`maxWords = 15`）、对学习者用 Lei、学习者是男性。
  - 每回合最多一个问题；不说英语，除非学习者问 “Parla inglese?”；不要 emoji、markdown 或舞台说明；数字用词写出来。
  - 方括号里的用户行是 app 的提示，不是学习者说的话。
  - 回复固定两行：意大利语一行，简体中文一行。
  - 角色先开口时发 `openingCue`。
- 反馈卡提示词（同文件）：
  - 告诉模型不要纠正识别造成的拼写和标点问题，只纠正听者会注意到的真错误；已经正确、礼貌的句子不算错。
  - 请求里带上挑战前几课的句子，避免去“纠正”课程本身教的说法。
  - 返回 JSON schema：`summary`（中文）、`corrections[≤3]{said, better, zh, why}`、`expression{it, zh, note}`。
  - 纠正的句子作为 `StudyProgress.practicePhrases`（id 是 `RP-` 加哈希）加两张新卡，课程里已有的不重复加；录音用角色声线合成后存在 Application Support/PracticeAudio（`mamma-mia/CLAUDE.md`；`mamma-mia/App/Services/PracticeAudio.swift`）。
- 句子切分 `mamma-mia/Packages/MammaCore/Sources/MammaCore/RolePlay/SentenceChunker.swift`：
  - 意大利语部分在第一个换行处结束，或者在第一个 CJK 字符处结束（防止模型把翻译写在同一行）。
  - 句末是 `. ! ? …` 后面跟空格加非小写字母；缩写和小数不算句末；短于 `minimumLength = 12` 字符的片段并到下一句，因为 TTS 单念一个词会怪。
  - `isCJK` 只认 `0x3000–0x30FF, 0x3400–0x4DBF, 0x4E00–0x9FFF, 0xF900–0xFAFF, 0xFF00–0xFFEF`，不含 CJK 扩展 B 及以后（U+20000 起）。
- 上限 `mamma-mia/Packages/MammaCore/Sources/MammaCore/RolePlay/TurnAllowance.swift`、`RolePlayConversation.swift`：每场 20 回合，每天默认 40 回合（两场），可调范围 10…200；开场白和反馈卡不计入。只作加练，不写挑战成绩（`mamma-mia/CLAUDE.md`）。
- 错误处理 `mamma-mia/Packages/MammaCore/Sources/MammaCore/RolePlay/GeminiFailure.swift`：
  - 401、403、404 归为 refused，429 归为 rateLimited，5xx 归为 server。
  - 能识别 `API_KEY_INVALID`、`SERVICE_DISABLED`、`BILLING_DISABLED`、`API_KEY_SERVICE_BLOCKED`、`API_KEY_IOS_APP_BLOCKED`、`CONSUMER_SUSPENDED` 等原因。
  - refused 时整个功能停发请求，直到改了设置或「测试连接」通过；不自动重试被拒的项目（`mamma-mia/CLAUDE.md`）。
- key 由用户在「对话 → 在线对话设置」粘贴，存进 iOS 钥匙串，属性是 `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly`；不进仓库、安装包和备份（`mamma-mia/App/Services/KeychainStore.swift`、`mamma-mia/CLAUDE.md`）。
- 对话里识别出的文字不写进 `AppLog`，日志只记延迟和错误码（`mamma-mia/CLAUDE.md`）。

### Inferences
- **原样复用**：`GeminiRequest`（端点和模型枚举）、`GeminiStream`（SSE）、`GeminiFailure`、`TurnAllowance`、`KeychainStore`、`StreamingSpeechPlayer`（24 kHz PCM，可原速或保持音高放慢重播）、“出字超时改问 Flash”、“refused 就停发”、“不记对话文字”。
- **必须改的**：`SentenceChunker` 的“第一个 CJK 字符就是翻译开始”这个前提在粤语里不成立，因为目标语言和翻译都是汉字。另外 `isCJK` 漏了扩展 B 以后的区段，粤语口语字有一部分在那里。详见 Q12。

### Gaps
- 无。

---

## Q7 数据和隐私约定、Linux 测试与工作方式

### Takeaway
仓库公开，所以定下了一条硬规则：个人数据一律不进仓库、不在编译时写进 app，只在手机上填写和导出备份；key 不打印、不提交，日志不记个人字段和对话内容。逻辑层在 Linux 容器里测，测试同时校验内容文件之间的引用。这些约定可以整套照搬到 diu。

### Cited Findings
- 真实行程、酒店地址、航班、过敏、住址等不进仓库，也不在编译时写进 app；`Content/trip.json` 会被 CI 拦下；测试里的行程只用假日期；仓库只放 `Content/trip.example.json`（`mamma-mia/CLAUDE.md`「隐私」；`mamma-mia/README.md`「个人数据」）。`.gitignore` 排除了 `.env`、`.env.*`（保留 `.env.example`）、`Content/trip.json`、`*.local.json`、`MammaMia-progress-*.json`（`mamma-mia/.gitignore`）。
- `.env` 已 gitignore，key 不许打印、不许提交（`mamma-mia/CLAUDE.md`「示范音频」；`mamma-mia/.env.example` 的中文注释）。
- 个人字段的处理：
  - 行程的任何字段都不写进 `AppLog`。
  - 酒店名、地址和忌口备注只用 iOS 自带语音读，不生成示范音频。
  - 巡礼点照片存缩小并去掉位置信息的副本，不进备份。
  - 不申请定位权限；相册只申请“仅添加照片”。
  - （`mamma-mia/CLAUDE.md`「行程与用到日」「随身」；`mamma-mia/project.yml` 权限说明）
- 版权：巡礼层只用地名、用户自己拍的照片和原创图形，不放原作截图、角色形象、台词原文或 logo（`mamma-mia/CLAUDE.md`「隐私」）。
- 测试：`swift test --package-path Packages/MammaCore` 在 `swift:6.3-noble` 容器里跑，“测试也会校验 `Content/` 的跨文件引用”（`mamma-mia/CLAUDE.md`）。`mamma-mia/Packages/MammaCore/Tests/MammaCoreTests/ContentTests.swift` 里有 `repositoryContentIsConsistent`、`courseHasThirtyLessonsAndTenChallenges`、`validatorCatchesBrokenReferences`、`checkedTextIsOptional`、`audioManifestKeyFormat`、`picksAStableVoiceAndFallsBack` 等测试。
- Python 校验 `mamma-mia/tools/validate_content.py`：
  - 检查 schema、跨文件引用、`checkedText` 一致性。
  - audio_ok 句子必须每个声线和速度都有音频，并且粗筛干净；不满足时对 audio_ok 句子报 error，对其他句子只报 warning。
  - 最后打印统计。

### Inferences
- diu 照搬这些约定：公开仓库、个人数据只在手机上、key 只在 `.env` 和钥匙串里、日志不记内容、CI 拦个人数据文件。
- 粤语专项的校验（粤拼和汉字逐一对齐、字表、繁简生成物是否最新）放进 `validate_content.py`，也放进 DiuCore 的 `ContentValidator` 测试，保持“两层都查”的做法。

### Gaps
- 无。

---

## Q8 从 mamma-mia 看得出的用户偏好（报告建议要贴合这些）

### Takeaway
用户的风格很一致：中文讲解、自用、重隐私、省钱、没有 Mac；内容由 LLM 起草，交给另一个模型交叉校对，再由人定稿；凡是“模型不可靠”的地方都在流程里绕开，并且把证据边界写清楚。用户会日语假名、拼音和英语，讲解时也借它们搭桥。

### Cited Findings
- **中文优先**：README、CLAUDE.md、界面文案、权限说明、报告全是简体中文，`developmentLanguage: zh-Hans`；交叉校对和粗筛要求模型用简体中文写理由（`mamma-mia/README.md`、`mamma-mia/project.yml`、`mamma-mia/tools/audio_review.py` 的 PROMPT、`mamma-mia/tools/cross_check.py` 的 INSTRUCTIONS）。
- **自用、零付费账号**：免费 Apple ID + SideStore，第一次安装用 Windows 电脑（`mamma-mia/docs/sideload.md`）。
- **学习者画像**：交叉校对提示词里写着“学习者是母语为中文、会日语假名的成年人，12 月去意大利旅行（威尼斯、那不勒斯、罗马）……发音提示用日语假名近似，有时用拼音”（`mamma-mia/tools/cross_check.py` 的 `INSTRUCTIONS`）。调研文档时区标为 Asia/Tokyo（`mamma-mia/docs/language-learning-github-research-2026-10-16.md` 开头）。
- **隐私**：见 Q7。
- **省钱**：
  - 用户自带 Google key，每天回合上限默认 40。
  - 粗筛按实测单价估算并写进报告（“208 次 × $0.004 ≈ $0.83”）。
  - 只重筛内容变了的音频（哈希缓存）。
  - 没有示范音频的句子退回免费的 iOS 语音。
  - （`mamma-mia/CLAUDE.md`、`mamma-mia/reviews/audio-screen-week1.md`）
- **LLM 辅助 + 交叉校对 + 人定稿**：
  - 内容状态 `draft → checked → native → audio_ok`，“工具不改内容文件”，人按报告改。
  - 模型能力边界写进流程，例如“Argon 判断双辅音长短不可靠”“Argon 偶尔会编出不相干的整句”“拥堵时……没听音频就作答”。
  - （`mamma-mia/CLAUDE.md`「交叉校对」「示范音频」）
- **证据纪律**：调研文档区分“已确认、推断、建议”，并写明“本次未执行该测试，因此这里只确认实现和测试存在，不报告测试通过”（`mamma-mia/docs/language-learning-github-research-2026-10-16.md` §2.2、§12）。
- **给编码代理写约定**：`CLAUDE.md` 的标题就是“给编码代理的约定”，代码由编码代理通过 PR 写（`mamma-mia/CLAUDE.md`）。
- **兴趣**：意大利行程同时是 JoJo 圣地巡礼（L23 “Sono un fan di JoJo”；`README.md` 标题“意大利语巡礼”；`CLAUDE.md`「随身」巡礼点）。

### Inferences
- 报告建议要满足这些条件：
  1. 继续用中文写约定和界面；
  2. 不引入后端、账号、订阅；
  3. 在线功能都是可选加练，用用户自己的 key，带每日上限；
  4. 粤拼、口语字这类最容易被 LLM 写错的内容，要有确定性的字典校验加第二个模型校对，最后人工定稿，不能单靠一个模型；
  5. 对“模型在声调上可不可靠”先做小规模实测再决定流程，符合用户“先测再信”的习惯。
- 用户会普通话（书面中文母语）和日语，是“普通话母语者学粤语”的典型画像。中文翻译字段（`zh`）应该写普通话意思；“书面语对照”对这个用户有额外价值。教学上的具体用法由其他调研员负责。

### Gaps
- 不能确定用户是否已经有粤语基础（听得懂一部分？），也不能确定学粤语的具体动机。

---

## Q9 粤语化：内容 schema（粤拼、口语字和书面语、繁简、多音字）

### Takeaway
把 `it` 拆成几个字段：`yue`（港式繁体书写的口语原文，是唯一的权威文本）、`jyutping`（逐字带调号，和 `yue` 里的汉字一一对齐）、`zh`（普通话意思），以及可选的 `formal`（书面语或普通话说法对照）、`ttsText`（只给 TTS 用的替换文本，处理多音字和文白读）、`accept`（可接受的别的说法）。简体显示形式由工具在构建时生成，不手写。`checkedText` 要同时锁住汉字和粤拼，音频 key 也要带上粤拼，这样改了读音就自动重生成音频。

### Cited Findings
- mamma-mia 现在的字段是 `{id, it, zh, frame?, scene, status, note?, checkedText?}`，并且 `additionalProperties: false`（`mamma-mia/Content/schema/lesson.schema.json`）。
- 审核结论只对审过的那段原文有效，改了原文就要重新校对或退回 draft（`mamma-mia/CLAUDE.md`「内容规则」）。
- manifest 的键是 `"<voice>|<speed>|<text>"`，schema 用正则 `^[a-z0-9]+@[0-9]+\|[ns]\|.+$` 约束（`mamma-mia/Content/schema/audio-manifest.schema.json`）。
- `lineVoices` 本来就是为“只有某个声线念得清楚的句子（比如最小对立）”准备的（`mamma-mia/tools/gen_audio.py` 文件头 docstring）。

### Inferences
- **建议的句子字段**（字段名供参考）：
  - `yue`（必填）：港式繁体（香港字形）的书面粤语原文，口语字照写（嘅、咗、唔、喺、嚟、啲、佢、冇、嘢、咁 / 噉）。它是音频 key、交叉校对和识别比对的唯一权威文本。用繁体做权威文本的理由是：口语字的规范写法和主要粤语资源大多是繁体；繁转简是多对一，简转繁有歧义。
  - `jyutping`（必填）：粤拼，每个音节带 1–6 调号，空格分隔，音节数要等于 `yue` 里汉字的个数；标点原样透传或忽略。英文字母和数字的处理要定规则，比如 “OK” 写 `ou1 kei1`，或者用 `[OK]` 跳过对齐。变调（例如“女”在“個女”里读 neoi2）一律写实际读音，可以选一种标记（比如 `neoi5*2`）保留本调信息，方便讲解。
  - `zh`（必填，沿用）：普通话意思，简体。
  - `formal`（可选）：对应的书面语或普通话说法，比如 佢哋係學生 对 他們是學生，给普通话母语者做“口语和书面语对照”。
  - `ttsText`（可选）：只发给 TTS 的文本。多音字或文白读被 TTS 读错时，换一个同音字或改写来逼出正确读音；显示和 key 仍然用 `yue`。
  - `accept`（可选）：开口说时可接受的别的说法（对应 mamma-mia 调研文档 R07“支持已审核的有效表达”）。纯属异体或俗写的差别交给归一化表处理，不用写进这里。
  - `checkedText`：改成等于 `yue + "|" + jyutping`（或者拆成 `checkedText` 加 `checkedJyutping` 两个字段），只改读音也会让审核失效。
  - 派生标签（由 `validate_content.py` 自动算，不手写）：`polyphone`（句中有字典里不止一个读音的字）、`tonePair`（出现在最小对立组里的字）。它们会触发“需人工听”的规则，见 Q10。
- **繁简**：简体粤语显示形式由 `tools/` 在构建时用 OpenCC 一类的转换表加粤语口语字补充表生成，写进生成文件，比如 `Content/generated/hans.json`。app 按用户设置切换显示。音频 key 和比对一律用 `yue`。这样 DiuCore 不依赖 ICU 的繁简转换，在 Linux 上也能测；同一张映射表也用在识别结果的归一化上。具体映射规则属于书写系统调研员的范围。
- **音频 key 改成** `"<voice>|<speed>|<yue>|<jyutping>"`：读音一改，文件名就变，音频自动重生成。要同步改三处：`AudioManifest.key`、`tools/` 里的 `audio_key()`、manifest schema 的正则。mamma-mia 已经要求这三处保持一致，只是规则延续。
- **剧本和角色配置同样改字段**：`dialogue` 的 `npc` 行、选项都从 `it` 改成 `yue + jyutping`；`roleplay.json` 的 `name` 用粤语，`fillers` 换成粤语语气词（嗯…、等我睇下先…、呃…）。

### Gaps
- 粤拼的取舍细节（英文夹杂、变调标记、懒音是否接受）属于教学和书写规范，需要书写系统和教学法调研员的结论来定。

---

## Q10 粤语化：示范音频生成、转写校验和 AI 粗筛

### Takeaway
Google 的 Gemini 3.8 TTS 系列文档有 Cantonese 条目，Cloud TTS 也有 yue-HK 声音，但没能确认 mamma-mia 现在用的 `gemini-3.8-flash-tts` 的粤语质量和语言判定方式。最大的风险是：mamma-mia 不发任何指令，全靠模型自动判断语言，繁体汉字很可能被当成普通话来念。所以转写校验要多做一件事：判断念出来的是粤语还是普通话。AI 粗筛加上声调、多音字、文白读、普通话读音、入声脱落几类问题，并且先测出模型判声调准不准，再决定信不信它。参照“双辅音需人工听”，设一条“声调、多音需人工听”。

### Cited Findings
- **Gemini 3.8 Flash TTS 的语言**：Google 文档称 `gemini-3.8-flash-tts` 自动检测输入语言，支持 130 多种语言。同系列 `gemini-3.8-flash-lite-tts` 的语言表里有 Cantonese 一行；Flash（非 Lite）的语言表在检索摘要里被截断，没看到 Cantonese 那行（搜索摘要，原页面无法直接抓取）。来源：[Gemini 3.8 Flash TTS](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-tts)、[Gemini 3.8 Flash-Lite TTS](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-lite-tts)、[Agent Platform TTS 概览（两个 3.8 模型的语言表）](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/text-to-speech/overview)。
- 中文版模型页的摘要提到“地道的区域口音”，支持“区域口音、少数民族方言和内嵌的国际音标（IPA）替换”（[Gemini 3.8 Flash TTS 中文页](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-tts?hl=zh-cn)，仅搜索摘要）。
- **Cloud TTS 的粤语声音**：
  - Chirp 3: HD 在 2025-12-30 的发布说明里新增 Chinese (Hong Kong) `yue-HK`，状态是 Preview（[Cloud TTS release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes)）。
  - 声音表里有 `yue-HK-Standard-A` 到 `-D` 四个 Standard 声音，A、C 是女声，B、D 是男声（[Cloud TTS voices](https://cloud.google.com/text-to-speech/docs/voices)）。
  - 2025-05 的发布说明称自定义发音支持所有 locale，只排除 8 个，其中没有 yue-HK（同一 release notes，搜索摘要）。但这条早于 yue-HK HD 上线，能否适用存疑。
- **粤语转写能力**：
  - 只找到厂商自己发布的对比：ElevenLabs 的粤语页面列出 FLEURS 上 “Gemini Flash 2” WER 17.6%，对比 Whisper Large v3 13.2%、Scribe 5.9% 或 3.1%（同一厂商不同页面数字不一致），而且是旧模型（[ElevenLabs Cantonese STT](https://elevenlabs.io/speech-to-text/cantonese)）。
  - 香港中文大学图书馆的口述史项目用 Whisper 转写粤语后，再让 Gemini 整理，提示词特意要求“保留粤语口语的自然风格，避免写得过于书面”（[CUHK Library Cantonese Transcription](https://dsprojects.lib.cuhk.edu.hk/en/projects/cuhk-60th-oralhistory/cantonese-transcription/)）。
- **mamma-mia 当前的做法**：
  - 原速只发原文、不带风格指令，因为指令会被念出来（`mamma-mia/CLAUDE.md`）。
  - 转写提示词是 “in whatever language it is”，不带目标文本（`mamma-mia/tools/gen_audio.py` 的 `transcribe()`）。
  - 比对用 `normalized()`：NFKD 去掉组合符、`[^\w\s]` 换成空格、意大利语数字拼写，再算 `difflib` ratio ≥ 0.85（`mamma-mia/tools/gen_audio.py`）。
  - 粗筛对模型不擅长的类别（双辅音）用正则过滤，并要求人工听（`mamma-mia/tools/audio_review.py` 的 `GEMINATION_TALK`、`DOUBLE`）。
  - 慢速用 `atempo`，音高不变（`mamma-mia/tools/gen_audio.py`；`StreamingSpeechPlayer.swift` 注释 “slowed (pitch kept)”）。

### Inferences
- **生成环节（`gen_audio.py`）**：
  - 语言判定：先查 Agent Platform 的 `speechConfig` 能不能指定 `languageCode`（或者等效参数）。不能的话，靠文本里的口语字（嘅、咗、唔、喺）把自动判断推向粤语。像“你好”“多謝”这样的短句和单字最容易被念成普通话；必要时用 `ttsText` 补上语境，或者给这类条目换引擎。
  - 引擎：`tts_voices.json` 给每个声线加一个 `engine` 字段（`gemini` / `cloud-tts`），粤语读不好的条目可以退到 Chirp 3 HD 或 Standard 的 yue-HK 声音。代价是在线对话的 live 声音和示范音频不再是同一个；mamma-mia 是用 `voiceNames` 保证两者一致的。
  - 多音字和文白读（行 hang4 / hong4、長 coeng4 / zoeng2、好 hou2 / hou3、重 cung5 / zung6 / cung4、樂 lok6 / ngok6、聽 teng1 / ting1、生 saang1 / sang1、平 peng4 / ping4、名 meng2 / ming4）：先信任上下文，读错就用 `ttsText` 换同音字；Google 声称的 IPA 内嵌替换在粤语上能不能用，要实测。
  - 慢速沿用 `atempo=0.75`。它保持音高，声调轮廓不会被扭曲，这一点对粤语是好事。
- **转写校验**：
  - 提示词改成让 checkModel 输出 JSON：`{"language": "yue|cmn|other", "transcript": "…"}`。要求逐字转写成香港书面粤语口语字，不改成书面语，数字按读出来的样子写成汉字。仍然不提供目标文本，避免模型顺着目标去“纠正”。出现 `language != yue` 直接判失败重试，专门拦截“念成普通话”。
  - `normalized()` 改为：NFKC → 去掉标点和空格 → 按繁简表折叠到同一种字形 → 俗写折叠（左→咗、系→係、黎→嚟、野→嘢、D→啲、既→嘅 等常见俗写，只用来比对）→ 粤语数字归一（兩 / 二、廿 / 二十、$35 / 三十五蚊）→ 逐字 SequenceMatcher。
  - 再加一层用字典转成粤拼的比对：转写里用了同音不同字也算对，因为“听者听到了对的音节”。粤拼字典或工具的选型（PyCantonese 等）由技术清单调研员负责。
  - 局限：转写模型自带的语言模型会把轻微的声调错误“脑补”成预期的字，所以转写校验主要拦的是念成普通话、漏读、多读、明显读错，拦不住细微声调错误。
- **AI 粗筛（`audio_review.py`）**：
  - 提示词改成粤语版，同时给模型目标 `yue` 和 `jyutping`。
  - 问题类型增加：`mandarin_reading`（念成普通话或夹普通话音）、`tone`（声调和粤拼不一致，写明哪个字、应读、听到）、`polyphone`（多音字或文白读错）、`final_stop`（入声 -p、-t、-k 丢失）、`lazy_sound`（n→l、ng- 脱落、gw→g；是否算错要内容约定先定）。原有的 omission、extra_speech、prosody、noise、truncation、slow_distortion 保留。
  - **先做校准再信任**：mamma-mia 实测 Argon“会自信地判错”双辅音，粤语声调很可能是同一类风险。建议先拿一小组读音已知的片段（不同声调的单音节和双音节最小对立；用 `ttsText` 故意制造的错调版本；多个引擎的同一句）测 Argon 判声调的准确率。准确率不够，就仿照 `GEMINATION_TALK` 把声调类意见过滤掉，只当提示。
- **audio_ok 门槛的粤语版**（仿照“含双辅音的句子留在 checked”）：
  - 课程句标 audio_ok 需要同时满足：已经 checked；粤拼通过字典校验；每段音频粗筛干净；转写判定为粤语。
  - 带 `polyphone` 或 `tonePair` 标签的句子，以及单音节、单词条目，必须有人听过（母语者抽检记为 `native`，或者新增一个 `listened` 记录）才能标 audio_ok。
  - 用户本人是粤语学习者，早期分辨不了声调，所以 `native`（母语者抽检）在粤语版里的分量比意大利语版重得多。
  - 最小对立组可以用 `lineVoices` 只保留念得最清楚的声线，这正是 mamma-mia 设计 `lineVoices` 的初衷。
- **成本**：按 mamma-mia 的实测（每段粗筛约 $0.004、每句 2 个声线 × 2 种速度），1000 段左右的粗筛约 $4。可能多出来的引擎对比和校准实验费用都不大，但要守 MAX_PARALLEL 和请求预算。

### Gaps
- 没能打开 Google 官方页面直接核对 `gemini-3.8-flash-tts` 语言表里的 Cantonese 一行和它的质量等级（GA 还是 Preview）。代理拒绝访问 ai.google.dev 和 docs.cloud.google.com，以上只依据搜索摘要。
- 没找到任何关于 Gemini TTS 粤语声调、多音字准确性的独立评测，也没找到 `gemini-3.8-flash` 或 `gemini-4-argon` 粤语转写字错率（CER）和“会不会转成书面语”的独立评测。只能自己用已知读音的样本实测。
- Agent Platform 的 `generateContent` 是否接受 `speechConfig.languageCode` 来固定粤语，没有核实。
- Chirp 3 HD 的 yue-HK 到 2026-10 是否已经 GA、有哪些声音名，没有核实。

---

## Q11 粤语化：离线识别和判定（包括“听懂即可、不打分”是否适合学声调）

### Takeaway
多份开发者报告显示 Apple 的 `SpeechTranscriber.supportedLocales` 里有 `yue-CN` 和 `zh-HK`，所以“离线识别加 SpeechCheck”这条路可以走。但“支持”不等于“已装”，输出用繁体还是简体也没确认，必须在真机上探测。判定层要改成按字对齐，读音相同的字算对，同音节不同调记成“听成了第几声”。“听懂即可”在粤语里大体成立，因为声调本身决定词义，识别器听错调往往就听成别的字；但它不足以训练声调，要另外加不打分的声调练习：最小对立听辨卡、单字或词的开口检查、基频曲线对比。

### Cited Findings
- **locale 支持**：
  - FluidVoice 的 issue（2026-10-21，macOS 27.0.1）报告 `SFSpeechRecognizer.supportedLocales` 和 `SpeechTranscriber.supportedLocales` 都包含 `yue-CN, zh-CN, zh-HK, zh-TW`，但 `SpeechTranscriber.installedLocales` 只有 `zh-CN, zh-TW`。选 zh-TW 输出繁体，选 zh-CN 输出简体；zh-HK、yue-CN 的输出没有测（[FluidVoice #1202](https://github.com/altic-dev/FluidVoice/issues/1202)）。
  - 一篇 iOS 26 SpeechAnalyzer 教程列出的支持值里有 `yue_CN`、`zh_HK`（[Anton Gubarenko: iOS 26 SpeechAnalyzer Guide](https://antongubarenko.substack.com/p/ios-26-speechanalyzer-guide)，搜索摘要）。
  - 另一个 issue 报告 macOS 27 上 `SpeechTranscriber` 列出 45 个 locale，其中有 zh-HK、yue-CN（[scrawl #51](https://github.com/kitlangton/scrawl/issues/51)，搜索摘要）。
- **Apple 文档**：先查 `isAvailable` 或 `supportedLocales`，不支持就禁用功能或改用 `DictationTranscriber`；还可以用 `supportedLocale(equivalentTo:)` 找近似 locale（[SpeechTranscriber 文档](https://developer.apple.com/documentation/speech/speechtranscriber)）。
- **模拟器不可信**：模拟器上 `supportedLocales` 返回空数组，`isAvailable` 为 false，必须在真机上测（[Apple Developer Forums 802969](https://developer.apple.com/forums/thread/802969)）。
- 一条第三方粤语输入法的 App Store 用户评论说它的听写只出香港繁体，希望能加简体输出。这只是单条评论，不代表 Apple 系统行为（[App Store 粵拼輸入法](https://apps.apple.com/us/app/%E7%B2%B5%E6%8B%BC%E8%BC%B8%E5%85%A5%E6%B3%95/id1509367629)）。
- **mamma-mia 现状**：
  - locale 选择是“找最接近 it_IT 的那个”，状态里分“需要下载”“下载中”“不支持”，模型要 `AssetInventory.reserve`（`mamma-mia/App/Services/SpeechService.swift`）。
  - 判定只看能不能听懂，不打发音分（`mamma-mia/CLAUDE.md`「开口说」）。
  - `CardKind.minimalPair` 已经声明（`mamma-mia/Packages/MammaCore/Sources/MammaCore/Scheduling/ReviewScheduler.swift`）。

### Inferences
- **`SpeechService` 的改法**：
  - 把“找最接近 it_IT”换成按顺序探测粤语 locale（比如先 `yue-CN`，再 `zh-HK`；哪个是粤语模型、输出什么字形，要在真机上确认），两个都写进 `AppLog` 和诊断页，并允许用户手动切换。
  - 注意 `zh-HK` 这个代码在一些系统里可能指“香港的中文”，未必就是粤语模型，所以要靠实测，不能靠代码名猜。
  - 引擎降级（Transcriber → Dictation → 只录音对比）和 `AssetInventory` 下载逻辑照搬。
- **SpeechCheck 和 SpokenSentence 重写**（DiuCore 新的 `Cantonese/` 和 `Speech/`）：
  1. 分词：每个汉字一个 token。CJK 判定要覆盖基本区、扩展 A、兼容区，以及扩展 B 到 H（U+20000–U+323AF），mamma-mia 的 `isCJK` 漏了扩展 B 以后的区段。拉丁字母串和数字串各算一个 token，全角和半角标点作为分隔。
  2. 归一化：繁简折叠；俗写和异体折叠（见 Q10）；粤语数字和金额归一（$35、35蚊、三十五蚊；兩 / 二；廿 / 二十；百二 = 一百二十；三蚊半；X點半、X個字），这部分替代 `wordsWithAmountsSpelledOut` 和意大利语数词表。
  3. 对齐：沿用编辑距离对齐的骨架，把“近似”的度量从拼写相似换成读音相似。规则是：同一个归一化字算匹配；粤拼完全相同（同音字）也算匹配；声母韵母相同、只有声调不同，算“近似替换”，生成 `heardAs`，界面显示成“『試 si3』听成了『屎 si2』”这类声调提示；其他情况算缺失。这需要一张设备上可用的“字 → 粤拼候选”表（可以从 Unicode Unihan 的 `kCantonese` 字段等开放数据生成，见 [UAX #38 Unihan](https://www.unicode.org/reports/tr38/)；数据选型由技术清单调研员确定），由 `tools/` 按内容字和常见识别输出字裁剪后放进 `Content/data/`。
  4. 判定四档（clear / mostly ≥ 60% / unclear / silent）照搬，覆盖率按字计算。
  5. ReplyMatcher：“词”改成字的二元组或单字关键字。内容约定从“选项之间要差至少一个听得出的词”改成“至少差一个声母或韵母不同的音节，只差声调的选项只能点选”，由 `validate_content.py` 检查。理由是识别器的语言模型可能把只差声调的选项“纠正”到同一个字上。
- **“听懂即可、不打分”是否适合声调学习？部分适合。**
  - 适合的地方：粤语里声调决定词义，识别器听错调常常就听成别的字，所以“对方能不能听懂”天然包含了一部分声调正确性；不打分也避开了不可靠的发音评分，和 mamma-mia 一致。
  - 不够的地方：第一，整句里识别器的语言模型会用上下文把错调“纠正”回来，就像真人也能凭语境听懂，所以 clear 不代表声调对；第二，普通话母语者的典型错误（3 声和 6 声、2 声和 5 声、4 声和 6 声混淆，入声收尾丢失，n 和 l 不分）在整句里很可能被放过。
  - 补救办法，都不打分：
    - (a) `minimalPair` 听辨卡：听一个选是哪个，结果客观，直接进 FSRS；
    - (b) 单字、单词的“说出来”练习：上下文最弱，识别器的 `heardAs` 最能暴露声调问题；
    - (c) 基频（F0）曲线叠图：在 DiuCore 里写一个纯 Swift 的基频提取（比如 YIN 或自相关），按说话人音域归一化成半音，把示范音频和自己的录音叠在一起给学习者看，只用来对比，不出分数。它是纯数组运算，可以在 Linux 上测；
    - (d) 母语者抽检。
  - 这样“识别只是提示、只有自评改卡片状态”的原则保持不变。

### Gaps
- 没有 iPhone（iOS 26.x）上 `SpeechTranscriber` 和 `DictationTranscriber` 对 `yue-CN`、`zh-HK` 的实测：哪个已装或可下载、输出繁体还是简体、会不会输出书面语、对初学者口音和错调的容忍度都不知道。上面的报告大多来自 macOS。
- Apple 识别器对粤港常见的中英夹杂（“check 吓”“OK 嘅”）怎么输出，也没有资料。

---

## Q12 粤语化：在线角色扮演和 iOS 兜底语音

### Takeaway
请求、流式、错误、上限这些工程部分原样复用。提示词要改成“地道香港粤语口语，用港式繁体口语字书写，不写书面语”。输出格式不能再靠“遇到汉字就是翻译开始”来区分，要用带标签的多行格式或 JSON。粤拼不要让 LLM 生成，改在设备上查表得到。iOS 自带的粤语声音（zh-HK）可以继续当兜底，但有一个已知 bug：iOS 18 起，中文按哪种方言念会被系统设置覆盖，开发者报告到 iOS 26 仍然存在。

### Cited Findings
- **iOS 中文朗读的方言 bug**：iOS 18 上 `AVSpeechSynthesisVoice(language: "zh-CN")` 在 Siri 语言设为粤语时会念成粤语，iOS 16、17 没有这个问题。有人发现方言实际由「辅助功能 → 朗读内容 → 声音 → 中文 → 口语语言」控制。楼主 2026 年 3 月补充 “This bug still exists in iOS 26”。Apple 工程师只请大家提交 bug 报告，没有给出绕过办法（[Apple Developer Forums 772348](https://developer.apple.com/forums/thread/772348)）。同类帖子还有 [788218](https://developer.apple.com/forums/thread/788218)、[768259](https://developer.apple.com/forums/thread/768259)。
- **mamma-mia 现状**：
  - 兜底语音用 `AVSpeechSynthesisVoice(language: "it-IT")`（`mamma-mia/App/Services/AudioService.swift`）。
  - 角色扮演回复固定两行（意大利语一行、简体中文一行），`SentenceChunker` 遇到第一个换行或第一个 CJK 字符就认为意大利语部分结束，并且会剥掉 “中文：” 之类标签（`mamma-mia/Packages/MammaCore/Sources/MammaCore/RolePlay/SentenceChunker.swift`）。
  - 提示词限制每回合 ≤ 15 词，数字用词写出来（`RolePlayPrompt.swift`）。
- **语气词**：`fillers` 由 `gen_audio.py` 按每个声线生成，`validate_content.py` 检查每个声线都有（`mamma-mia/CLAUDE.md`）。

### Inferences
- **`RolePlayPrompt` 粤语版的要点**（英文写给模型，沿用 mamma-mia 的写法）：
  - 用香港粤语口语说话，用港式繁体书面粤语书写（嘅、咗、唔、喺、嚟、啲、佢、冇、嘢），不要书面语或普通话词汇（們、的、是、在、沒有、什麼、這、那，要换成 哋、嘅、係、喺、冇、乜嘢、呢、嗰）。
  - 长度上限改按音节或字数算，比如 ≤ 25 字，替代 `maxWords = 15`。
  - 数字按读出来的样子写汉字（三十五蚊、三點半）。
  - 礼貌规则改为粤语习惯（唔該 / 多謝、先生 / 小姐），删掉 Lei 和阳性那两条。
  - 中英夹杂要不要允许、允许多少，需要内容约定先定。
  - 告诉模型：学习者的话是识别结果，可能是简体（取决于识别 locale），也可能有同音错字。
- **输出格式**：改成带标签的行，比如 “粵：…” / “普：…”，或者干脆用结构化 JSON 输出。`SentenceChunker` 改为在标签或换行处切出粤语部分；句末判定用全角的 。！？…；`minimumLength` 按字数算（比如 4 字），不再按 12 个拉丁字符算。
- **粤拼**：LLM 生成的粤拼不可靠（这是 Q10 和 Q11 用字典校验的同一个理由），所以 live 对话的粤拼在设备上用“字 → 粤拼”表加多音字规则现算，点按时才显示，并且标明“自动注音，可能有误”。
- **反馈卡**：schema 字段 `it` 改成 `yue`；`zh` 是普通话意思；`why` 用中文；粤拼由本地表补上，模型不必生成。提示词要告诉模型学习者是普通话母语者，常见错误是书面语或普通话直译、量词、句末助词、双宾语语序（俾本書我）、先 / 添 / 未 的用法。纠正进复习卡的流程（`practicePhrases`、`RP-` 加哈希）照搬。
- **live TTS**：继续用 manifest 的 `voiceNames`，和示范音频同声线。风险和 Q10 一样（自动语言判定可能念成普通话）。合成失败或判定异常时退到 iOS 的 zh-HK 声音。兜底要显式请求 `AVSpeechSynthesisVoice(language: "zh-HK")`，在诊断页列出已安装的 zh-HK 声音和质量档。按论坛报告，系统「朗读内容」里的中文口语语言可能覆盖 app 的请求，所以第一次使用时要提示用户去检查这个设置。
- **钥匙串、每日上限、refused 就停发、不记对话内容**：原样复用。

### Gaps
- 没有核实 Gemini（Argon / Flash）写书面粤语口语字的稳定性，比如会不会混进書面語或简体。只有 CUHK 项目那种“要靠提示词强调保留口语风格”的旁证（见 Q10）。
- 没有核实 iOS 26 粤语声音（Sin-Ji 等）的标识符、增强版或高级版的可用性；检索中看到有人提到 `com.apple.ttsbundle.Sin-Ji-compact`，但没能确认出处。

---

## Q13 粤语化：字体、罕用字和繁简显示

### Takeaway
mamma-mia 不涉及字体问题，这部分基本是新增。粤语口语字里有不少在 CJK 扩展区，处理顺序建议是：先在 `tools/` 里做字表检查，能用常用字就用常用字，必要时附带一个裁剪后的开源字体。显示以港式繁体为主，可以切换成生成的简体。

### Cited Findings
- mamma-mia 判断 CJK 只覆盖到 U+FFEF 以内，没有扩展 B 及以后的区段（`mamma-mia/Packages/MammaCore/Sources/MammaCore/RolePlay/SentenceChunker.swift` 的 `isCJK`）。整个仓库没有自定义字体资源（`mamma-mia/App/Assets.xcassets` 里只有 AppIcon）。
- 开发语言是 `zh-Hans`，所有讲解都是简体（`mamma-mia/project.yml`）。

### Inferences
- `validate_content.py` 增加字表检查：
  - 列出内容里所有 BMP 以外的字和不常用的字，提示作者能否换成常用写法；
  - 检查每个汉字都能在“字 → 粤拼”表里查到；
  - 检查生成的简体文件是最新的。
- 显示：
  - 用 `.environment(\.locale, Locale(identifier: "zh-Hant-HK"))` 或指定港式字体，让系统用香港字形渲染。
  - 如果真机上发现缺字，用 fontTools 按内容字把开源的 Noto Sans HK 或思源黑体港版裁剪成子集，放进 bundle 当后备字体。许可证要由书写系统和技术清单调研员确认，体积预计很小。
  - 粤拼注音：SwiftUI 没有现成的 ruby 注音，可以按字排成 VStack，或者用 Core Text 的 `CTRubyAnnotation` 包一层 UIView。这是工程推断，没有验证。
- 繁简切换是纯显示偏好，存进 `StudyProgress` 或 UserDefaults。音频 key、比对和审核一律基于 `yue`。

### Gaps
- iOS 26 系统字体（PingFang HK 等）对 HKSCS 和扩展 B 粤语字的覆盖范围没有核实，需要真机检查。

---

## Q14 建议的 diu 目录结构和 schema 变更草案（从 mamma-mia 派生）

### Takeaway
沿用 mamma-mia 的布局：`App/`、`Packages/<Core>/`、`Content/`（含 `schema/`）、`Audio/`、`tools/`、`reviews/`、`docs/`、`.github/workflows/`，CLAUDE.md 用中文写给编码代理。在 Core 里新增 `Cantonese/` 和 `Tone/`，`Content/` 里新增由工具生成的数据表；字段从 `it` 换成 `yue`、`jyutping` 等；音频 key 带上粤拼。

### Cited Findings
- mamma-mia 的目录约定见 `mamma-mia/README.md` 的「目录」表和 `mamma-mia/CLAUDE.md` 的「目录」节：`project.yml`、`App/`、`App/Assets.xcassets/AppIcon.appiconset/`（一张 1024×1024、不带透明通道的 PNG）、`Packages/MammaCore/`、`Content/`（含 `schema/`、`showcards.json`、`spots.json`）、`Audio/`、`tools/`、`reviews/`、`.github/workflows/`。
- 文件命名：`lessons/Lxx.json` 的文件名要和 `unit` 一致；交叉校对报告叫 `reviews/cross-check-<单元>.md`（`mamma-mia/CLAUDE.md`）。
- 交叉校对 `mamma-mia/tools/cross_check.py`：
  - 默认模型 `gemini-4-argon`、思考档 HIGH，分批发送。
  - 返回 schema 是 `items[{id, verdict: ok|revise, issues[{field, severity: error|suggestion, problem, suggestion}], comment}]`。
  - `INSTRUCTIONS` 是中文的 7 条审查项：语法、语域、地道程度、中文翻译、发音提示、剧本选项、剧本台词。
  - 写明“不要为了凑数挑刺”“你只审文字，听不到音频”。

### Inferences
- **建议目录**：

```
Diu/
├── README.md                     # 中文：定位、装到手机、目录、个人数据
├── CLAUDE.md                     # 中文，给编码代理的约定（由 mamma-mia/CLAUDE.md 改写，见下）
├── .env.example  .gitignore      # 同 mamma-mia，增加 Diu-progress-*.json
├── project.yml                   # name: Diu；bundle io.github.li1000v2.diu；iOS 26；zh-Hans；Content/Audio 作为 folder 资源
├── App/
│   ├── DiuApp.swift  AppModel.swift
│   ├── Services/                 # SpeechService(粤语 locale 探测)、AudioService(zh-HK 兜底)、GeminiClient、KeychainStore、
│   │                             # Recorder、StreamingSpeechPlayer、ReplySpeaker、RolePlayService/Session、NetworkMonitor、PracticeAudio
│   ├── Views/                    # Today、Lessons、LessonPlayer、ReviewSession、ChallengePlayer、RolePlay(+Settings)、Speech、
│   │                             # ToneCompareView(基频曲线叠图)、Diagnostics(加：识别 locale、zh-HK 声音、字体检查)
│   └── Support/                  # AppLog、BuildInfo
├── Packages/DiuCore/
│   ├── Package.swift             # 只依赖 Foundation + swift-fsrs（固定 revision，FSRS-6）
│   ├── Sources/DiuCore/
│   │   ├── Scheduling/           # ReviewScheduler、ReviewQueue（照搬；CardKind 启用 minimalPair）
│   │   ├── Progress/             # StudyProgress（去掉 trip/spotLogs 或保留为可选；只做加法）、ProgressStore
│   │   ├── Content/              # Course、ContentBundle、ContentValidator（加粤拼对齐、字表校验）
│   │   ├── Audio/                # AudioManifest（key = voice|speed|yue|jyutping）
│   │   ├── Dialogue/             # 照搬
│   │   ├── RolePlay/             # Gemini* 照搬；RolePlayPrompt、SentenceChunker、RolePlayFeedback 粤语化
│   │   ├── Cantonese/            # Jyutping(解析/校验/声母韵母声调拆分)、Segmenter(汉字含扩展区/拉丁/数字)、
│   │   │                         # ScriptFolding(繁简+俗写表，读 Content/data)、Lexicon(字→粤拼候选)、Numbers(粤语数字/金额)
│   │   ├── Speech/               # SpeechCheck(按字对齐、同音算对、同音节异调→heardAs)、ReplyMatcher
│   │   ├── Tone/                 # PitchContour(YIN/自相关 F0、半音归一)；只对比不打分
│   │   ├── Signing/              # 从 DepartureCheck 拆出的 mobileprovision 到期解析
│   │   ├── Study/  Time/         # 照搬
│   │   └── Travel/               # 可选：有港澳/广州行程才保留（CityPack/NeedBy 改城市）
│   └── Tests/DiuCoreTests/       # Linux 上跑；新增粤拼、切分、归一、数字、对齐、声调近似、F0 的测试
├── Content/
│   ├── course.json               # frames(粤语句框) × scenes(茶餐廳/港鐵/的士/飲茶/街市…)；neededAt 改可选
│   ├── lessons/Lxx.json  dialogues/Cxx.json  roleplay.json
│   ├── data/                     # 由 tools/build_tables.py 生成并提交：hk2hans.json、variants.json(俗写→规范)、
│   │                             # readings.json(内容字+常见识别输出字→粤拼候选)
│   ├── generated/hans.json       # 简体显示形式（生成物，CI 检查是否最新）
│   └── schema/                   # lesson/course/dialogue/roleplay/audio-manifest(+ data 表)的 schema
├── Audio/                        # *.m4a + manifest.json
├── tools/
│   ├── gen_audio.py              # 粤语：可选 engine、ttsText、转写 JSON(language+transcript)、粤语归一与粤拼比对
│   ├── audio_review.py           # 粤语问题类型；声调意见在校准前只作提示；polyphone/tonePair“需人工听”
│   ├── regen_audio.py            # 照搬（两次干净才替换，最多 4 版）
│   ├── cross_check.py            # 粤语审校（见下）
│   ├── validate_content.py       # + 粤拼与汉字逐一对齐、字典读音候选、选项差音节、字表、生成物最新
│   ├── build_tables.py           # 新：从开放数据生成 Content/data/*
│   ├── calibrate_tone_screen.py  # 新：测粗筛模型判声调的准确率
│   ├── tts_voices.json           # + engine 字段；lineVoices 用于最小对立
│   └── requirements.txt          # jsonschema；按需加 OpenCC、粤拼工具（由技术清单调研员选型）
├── reviews/                      # cross-check-<单元>.md、audio-screen-*.md/.html、audio-screen-cache.json、audio-regen*.{md,json}、tone-calibration.md
├── docs/sideload.md              # 照搬并写明“3 个侧载名额已被 SideStore + MammaMia + Diu 占满”
└── .github/workflows/            # content-check、core-tests(swift:6.3-noble)、ios-build(macos-26/Xcode 26.6)、ipa（改名照搬）
```

- **lesson 句子 schema 草案**（在 `lesson.schema.json` 的基础上改）：

```json
{
  "id": "L05-01",
  "yue": "唔該，一杯凍檸茶。",
  "jyutping": "m4 goi1 jat1 bui1 dung3 ning4 caa4",
  "zh": "劳驾，来一杯冻柠檬茶。",
  "formal": "請給我一杯冰檸檬茶。",
  "frame": "mgoi",
  "scene": "chachaan",
  "status": "draft",
  "ttsText": null,
  "accept": ["凍檸茶，唔該。"],
  "note": "（中文讲解：先听音频；搭桥只讲像什么、再写哪里不像）",
  "checkedText": "唔該，一杯凍檸茶。|m4 goi1 jat1 bui1 dung3 ning4 caa4"
}
```

  以上示例内容只用来说明字段，不保证读音和译文正确，正式内容要走交叉校对。

  要更新的规则：
  - `id`、`unit` 的正则照旧（`Lxx-nn`）。
  - `jyutping` 的音节正则：声母（可空）+ 韵母 + `[1-6]`，可选 `*` 加变调。
  - `validate_content.py` 检查音节数等于汉字数，并且每个音节都在该字的字典候选里；不在候选里时报 warning，标 checked 前必须由人确认。
  - 状态非 draft 时，`checkedText` 必须等于 `yue|jyutping`。
- **dialogue、roleplay 的变更**：
  - `npc[].{yue, jyutping, zh, answer?, distractors?}`；`choices[].{yue, jyutping, correct, feedback?}`。
  - `roleplay.json` 的 `fillers` 换成粤语语气词，`name` 用粤语，`setting` 保持英文并写进粤语口语要求，`task` 用中文。
- **manifest 变更**：`entries` 键改为 `<voice>@<rev>|<n|s>|<yue>|<jyutping>`，schema 正则相应修改。`AudioManifest.key`、`tools` 里的 `audio_key()`、测试 `audioManifestKeyFormat` 三处要同步。
- **`cross_check.py` 粤语版的审查项**（改写 `INSTRUCTIONS`）：
  1. 是不是地道的香港粤语口语，有没有书面语或普通话直译；
  2. 口语字写法是否规范（港式字形，俗写如 既 / 左 / 系 / 黎 算问题）；
  3. 粤拼逐音节是否正确，包括声调、多音字在语境中的读音、文白读、变调；
  4. 港穗用词差异（巴士 / 公交、的士 / 出租车、埋單 / 結賬）要说明；
  5. 普通话翻译和书面语对照是否准确；
  6. note 和 intro 里关于发音的搭桥是否误导；
  7. 剧本选项是否“只差声调”（这种要改成点选），对错和 feedback 是否成立。

  另外：把字典查出的读音候选一起发给模型，让它在“字典候选 vs 作者写的粤拼”之间判断，不凭空生成；继续“只审文字，听不到音频”，声调对不对由音频流程和人来听。
- **CLAUDE.md 改写清单**（逐节对应 `mamma-mia/CLAUDE.md`）：
  - 「目录」「编译与测试」：照搬后改名。
  - 「示范音频」：照搬 MAX_PARALLEL=8、401/403/404 立即停、不许换项目绕过的规则，补上粤语语言判定和 `ttsText` 的说明。
  - 「开口说」：粤语 locale、按字对齐、同音算对、只差声调的选项只能点选。
  - 「在线 AI 角色扮演」：粤语提示词、标签格式、本地注音。
  - 「隐私」「免费 Apple ID 的限制」：照搬。
  - 「内容规则」：状态流水线照搬；audio_ok 的“双辅音”换成“多音字、最小对立、单字条目需人工听”；发音搭桥换成针对普通话加日语母语者的粤拼规则；Lei 和阳性换成粤语语域规则；“只教怎么问”照搬。
  - 「交叉校对」：粤语版审查项。

### Gaps
- 如果 diu 没有旅行目标，Travel 层整体删除后，“今日 / 随身”的首页信息架构要重新设计，这属于通用 app 架构调研员的范围。
- 粤语开放数据（字表、粤拼词典、繁简表）的许可证和具体选型不在本笔记范围，上面的生成表设计要等相关调研员的结论。
