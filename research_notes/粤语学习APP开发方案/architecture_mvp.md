# 粤语学习 App：技术架构、关键功能实现、成本与 MVP 路线（2026-10 调研笔记）

> 调研日期：2026-10-24。目标用户：读简体字的普通话 + 赣语母语者，学粤语口语 + 粤语白话字。
> 协调者补充的前提：大概率是**自用 iOS app**，照搬用户已有项目 mamma-mia（意大利语旅行学习 app）的架构：iOS 26+、SwiftUI、XcodeGen；可在 Linux 上测试的 Swift 包（MammaCore，只依赖 swift-fsrs / FSRS-6）；内容为 JSON + JSON Schema；离线预生成音频（Gemini/Google TTS）+ 自动转写校验；Apple SpeechAnalyzer 离线识别；用用户自己的 Google key 做在线 AI 角色扮演；无 Mac，GitHub Actions 出未签名 IPA，SideStore 侧载。因此本笔记以原生 iOS/Swift 路线为主，通用平台比较和大陆分发只简述。
> 数据集/词典/语料/许可证的清单由另一位研究员负责，这里只讲“怎么搭、怎么接”。
> 网络限制说明：本次调研环境无法直接抓取 learn.microsoft.com、azure.microsoft.com、aws.amazon.com、ai.google.dev、arxiv.org、help.aliyun.com 等页面，相关结论来自搜索摘要（已标注），价格类数字务必上线前在官方页面复核。

## Q1. 平台/技术栈选择（以原生 iOS/Swift 为主），含 CJK 生僻字渲染与分发

### Takeaway
对“自用 + 侧载 + 已有 mamma-mia 代码库”的场景，最优解是**直接复刻 mamma-mia 的原生 SwiftUI 架构**（把 MammaCore 换成粤语版核心包），跨平台框架（Flutter/RN/PWA）带来的收益几乎为零；需要额外处理的只有两件事：粤语端侧语音（Apple 对粤语的支持需真机运行时探测，备选 sherpa-onnx SenseVoice）和 Extension B 生僻字字体回退（需真机测试，必要时打包子集字体）。

### Cited Findings
**mamma-mia 现有架构（可直接复用的约束与经验）**
- App 只放界面与 iOS 专属能力；核心包 MammaCore 只依赖 Foundation 和 swift-fsrs，禁止 import UIKit/SwiftUI/SwiftData，以便在 Linux 上测试；CI 用 `swift:6.3-noble` 容器跑测试，ios-build 在 macos-26 + Xcode 26.6 编译，部署目标 iOS 26 — 本地文件 `/home/user/li1000v2/mamma-mia/CLAUDE.md`
- 免费 Apple ID 侧载限制：只有一个 app target、不加任何扩展（扩展也占 3 个侧载名额）、不用 APNs/iCloud/App Groups，免费签名 7 天需续签；持久化格式只做加法改动以保住每周重装后的学习记录 — `/home/user/li1000v2/mamma-mia/CLAUDE.md`
- 识别链路：SpeechAnalyzer + SpeechTranscriber（it_IT），设备不支持则退到 DictationTranscriber，再不行只保留录音对比和自评；模型经 AssetInventory 下载 — `/home/user/li1000v2/mamma-mia/CLAUDE.md`
- 一个运维教训：2026-10-16 一天数千次 Gemini 请求、最高 40 路并发后，Google Cloud 项目被 Trust & Safety 以“违反服务条款”停用（HTTP 403），之后并发上限定为 8 路 — `/home/user/li1000v2/mamma-mia/CLAUDE.md`

**iOS 端侧粤语 ASR 选项**
- SpeechAnalyzer/SpeechTranscriber 是 iOS 26 新 API（WWDC25 session 277）— [Apple WWDC25](https://developer.apple.com/videos/play/wwdc2025/277/)。第三方资料称 SpeechTranscriber 只支持“约 10–20 个”或“约 30 个”最常用 locale，DictationTranscriber 支持更多 locale 和变体但精度较低，旧 SFSpeechRecognizer 覆盖大多数 locale — [nitro-speech 文档](https://github.com/NotGeorgeMessier/nitro-speech/blob/main/docs/features/supported-locales.md)；[swiftcrafted 指南](https://swiftcrafted.dev/article/speechanalyzer-speechtranscriber-ios-26-swift)。**没有找到任何来源确认 SpeechTranscriber 支持 yue-CN / zh-HK。**
- 判断支持时应使用 `SpeechTranscriber.supportedLocale(equivalentTo:)` 而不是手工比较 BCP-47 字符串（zh-Hant-TW vs zh-TW 之类会踩坑）— [DEV Community](https://dev.to/simple_memo/ios-26-speechanalyzer-what-i-learned-wiring-it-to-a-mic-f7p)；iOS 26 模拟器上 `supportedLocales` 返回空数组，必须真机测 — [Apple Developer Forums](https://developer.apple.com/forums/thread/802969)
- SFSpeechRecognizer 支持的 locale 与键盘听写相同 — [Apple 文档 supportedLocales()](https://developer.apple.com/tutorials/data/documentation/speech/sfspeechrecognizer/supportedlocales().md)；第三方示例输出列出 yue-CN 和 zh-HK — [MBS plugins](https://mbsplugins.eu/SpeechRecognitionSupportedLocales.shtml)；WWDC 2019 幻灯片列出端侧识别的中文为普通话和粤语（较旧）— [WWDC19 Session 256 PDF](https://devstreaming-cdn.apple.com/videos/wwdc/2019/256p7m9z4yst71ai/256/256_advances_in_speech_recognition.pdf?dl=1)；`supportsOnDeviceRecognition` 是实例属性 — [Apple SFSpeechRecognizer](https://developer.apple.com/documentation/speech/sfspeechrecognizer)；有开发者遇到端侧语言资源缺失错误，需在设置中启用 Siri/下载模型后才可用 — [Apple Developer Forums](https://developer.apple.com/forums/thread/703770)
- sherpa-onnx（Apache-2.0）支持 iOS/Android/macOS/WebAssembly 等，提供 Swift、Kotlin、Dart 等 API，功能含流式/非流式 ASR、TTS、VAD、关键词、说话人识别；有中英日韩粤模型 `sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17`，以及中英粤 Paraformer-large 实时模型 — [k2-fsa/sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx)；更新日志提到把 ASLP-lab 的 WSYue-ASR `u2pp_conformer_yue` 导出到 sherpa-onnx — [sherpa_onnx_ios changelog 镜像](https://pub-web-s1.flutter-io.cn/packages/sherpa_onnx_ios/changelog)
- WhisperKit（MIT，Argmax）在 Apple 芯片上跑 Whisper，推荐 iOS 用压缩版 large-v3-turbo（626MB）；同一 SDK 还有 SpeakerKit（iOS 16+）与 TTSKit（iOS 18+，跑 Qwen3-TTS）；README 未提粤语 — [argmaxinc/WhisperKit](https://github.com/argmaxinc/WhisperKit)
- SenseVoice 论文称在粤语上 SenseVoice-Large 明显优于 Whisper-Large-V3 — [FunAudioLLM 论文](https://arxiv.org/pdf/2407.04051)；一篇 2026 博客给出粤语 CER SenseVoice 7.09% vs Whisper large-v3 10.41%（测试集未说明，未核实）— [convertaudiototext 博客](https://convertaudiototext.com/de/blog/best-transcription-for-asian-languages)

**iOS 系统粤语 TTS（作为无预生成音频时的兜底）**
- AVSpeechSynthesizer 支持 zh-HK，有开发者按 utterance 指定 Sin-Ji（粤语）声线，同一合成器可排队粤语和普通话句子 — [Apple Developer Forums](https://developer.apple.com/forums/thread/651832)
- iOS 18 起有 bug：辅助功能“朗读内容 → 声音 → 中文”的设置会覆盖代码里显式指定的 zh-HK/zh-TW；有评论称 iOS 26 仍存在 — [Apple Developer Forums 788218](https://developer.apple.com/forums/thread/788218)；[772348](https://developer.apple.com/forums/thread/772348)
- `speechVoices()` 只返回已安装的声线，没有 API 列出可下载但未安装的增强/高级声线；默认质量明显差于增强/高级 — [Apple Developer Forums 758460](https://developer.apple.com/forums/thread/758460)

**CJK Extension B 生僻字渲染**
- Extension B 区块为 U+20000–U+2A6DF，约 42,720 个码位；部分粤语用字在 Ext B（HKSCS 混合了粤语用字与香港用字）— [Wikipedia](https://en.wikipedia.org/wiki/CJK_Unified_Ideographs_Extension_B)；[Unicode 邮件列表 2002](https://unicode.org/mail-arch/unicode-ml/y2002-m05/0121.html)
- Noto Sans HK 支持 55 个 Unicode 区块共 44,806 个字符，含 Ext B（但非全覆盖）— [Fontsource](https://fontsource.org/fonts/noto-sans-hk/about)；Noto CJK 家族对 Plane 2 只做“满足中日标准所需”的有限覆盖 — [FreeBSD port 描述](https://cgit.freebsd.org/ports/plain/x11-fonts/noto-serif-hk/pkg-descr)
- iOS 18/macOS 15 起系统 PingFang 不再是标准 TTC 文件；有报告称 PingFang 资源未下载时字符显示为豆腐块 — [Apple Developer Forums (rcombs)](https://origin-devforums.apple.com/forums/profile/rcombs/replies)；PingFang HK 的汉字覆盖少于 PingFang SC（macOS 旧资料）— [Tor Project issue](https://gitlab.torproject.org/tpo/team/-/issues/45)；W3C clreq 系统字体对照表（2026-07-21 核对）— [W3C clreq](https://w3c.github.io/clreq/others/system-font-mappings.html)

**跨平台方案（简述）与大陆分发（简述）**
- sherpa-onnx 官方支持 Flutter（含 Web）、Tauri；React Native 只有第三方封装 — [k2-fsa/sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx)
- 工信部《关于开展移动互联网应用程序备案工作的通知》（信管〔2023〕105号）：境内提供互联网信息服务的 App/小程序须备案，新 App 先备案后上线，存量 App 2024-03 底前补办；单机不联网的 App 可不备案 — [澎湃新闻](https://www.thepaper.cn/newsDetail_forward_24155262)；[安全内参](https://www.secrss.com/articles/57609)
- Apple App Store Connect：部分 App 在中国大陆上架须有与工信部备案一致的 ICP 备案号 — [App Store Connect Help](https://developer.apple.com/help/app-store-connect/reference/app-information/)；论坛称不面向大陆用户的开发者可不填 — [Apple Developer Forums](https://developer.apple.com/forums/thread/743661)
- 微信小程序：未上架的 2023-09-01 起须先备案；早年资料称个人主体可开发教育等 9 个一级类目，但直播类组件只对非个人主体开放 — [知晓程序](https://www.ifanr.com/minapp/921668)；[腾讯云文档](https://cloud.tencent.com/document/product/1639/129300)

### Inferences
- **推荐栈（自用 iOS）**：XcodeGen + SwiftUI App；`CantoCore`（或直接把 MammaCore 抽象成语言无关核心）= FSRS 调度 + 内容解析 + 粤拼工具 + 识别结果比对 + 声调评分算法（纯 Swift + Foundation，Linux 可测）；内容 JSON + JSON Schema；`tools/` Python 管线负责 TTS 预生成、转写校验、粤拼标注、参考音高曲线预计算、简繁转换。和 mamma-mia 唯一的结构性差异是：声调评分/音高提取最好也放进核心包（纯数值算法，Linux 可测），只有 AVAudioEngine 采集与 Accelerate 加速放 App 层（Accelerate 在 Linux 不可用，核心包需写纯 Swift 回退实现或用协议注入）。
- **端侧 ASR 三级回退**（照搬 mamma-mia 的思路）：① `SpeechTranscriber.supportedLocale(equivalentTo: Locale(identifier: "yue-CN"))` / `zh-HK` 真机探测；② DictationTranscriber 或旧 SFSpeechRecognizer（zh-HK/yue-CN 在列表中，`supportsOnDeviceRecognition` 运行时读取）；③ 内置 sherpa-onnx + SenseVoice int8 模型（体积约数百 MB 级，需确认；可做成“首次启动下载”以控制 IPA 大小——但侧载 IPA 本身没有商店大小限制）。WhisperKit 作为第四选项（模型更大，粤语表现据 SenseVoice 论文不如 SenseVoice）。
- **字体策略**：常用粤语字（佢、嘅、咗、啲、冇、嚟、喺、嘢、睇、搵）都在基本区/扩展 A，系统字体能显示；真正风险是少量 Ext B 字（如用户例举的 𨋢、𡃁，以及赣语的 𠮶）。做法：内容管线扫描所有 JSON，列出 > U+FFFF 的码位 → 真机截图测试是否豆腐 → 若有缺字，用 fonttools 从 Noto Sans CJK HK / Source Han Sans HC 生成**仅含这些字的子集字体**（几十 KB），在 Info.plist `UIAppFonts` 注册，并通过 `UIFontDescriptor` 的 cascadeList（或 SwiftUI 中对含生僻字的 Text 单独指定字体）做回退。代码层注意 Ext B 字在 UTF-16 中是代理对：点击查词、高亮、粤拼对齐一律按 Swift `Character`/Unicode scalar 计数，避免 NSRange/UTF-16 偏移错位。
- **简繁**：粤语资源（词典、rime-cantonese、words.hk 等）以繁体为主，而用户读简体。建议内容以繁体为“源文本”（便于用粤语工具链做 g2p/分类），构建时用 OpenCC 生成简体显示文本并存两份；UI 默认显示简体、可切换繁体。
- **跨平台方案**：只有在将来要给他人用、要上 Android/大陆渠道时才值得考虑 Flutter（sherpa-onnx 有官方 Flutter 支持）；PWA 在 iOS 上做录音 + 端侧 ASR + 后台音频都更弱。自用场景不建议偏离原生。
- **大陆分发**：自用侧载不涉及备案；若将来上架大陆 App Store 或做小程序，需要 ICP 备案（意味着大陆主体/服务器），且联网 + 语音数据会触发更多合规要求——这会显著改变后端选型（见 Q7）。

### Gaps
- SpeechTranscriber/DictationTranscriber 是否支持粤语、粤语识别输出的是口语字（佢/嘅/咗）还是书面语、输出繁体还是简体：未找到来源，必须在 iOS 26 真机上实测。
- sherpa-onnx SenseVoice int8 模型在 iPhone 上的实际体积、内存与延迟：未找到官方数字。
- iOS 26 系统字体对 𨋢、𡃁、𠮶 等具体 Ext B 字的覆盖：未找到来源，需真机测试。
- iOS 是否提供粤语“高级/增强”声线及其名称：论坛资料不足。

## Q2. 间隔重复：FSRS（Swift 实现）vs SM-2，适合粤语的卡片类型

### Takeaway
继续用 swift-fsrs（FSRS-6，需钉在 main 的提交上），无需考虑 SM-2；粤语的特殊性不在调度算法，而在**卡片设计**：把“听辨”“开口回忆”“普→粤口语对译”“字→粤拼+声调”拆成独立调度的卡，声调最小对立对（minimal pairs）做成不进 FSRS 的自适应感知练习更合适。

### Cited Findings
- swift-fsrs（MIT，约 107 star）面向 FSRS-6.0：`FSRS(parameters: .init(w: FSRSDefaults.defaultWv6))` 启用 21 参数的 FSRS-6，FSRS-5 用 19 参数；API 为 `FSRSDefaults().createEmptyCard()`、`try v6.next(card:now:grade:)`；README 未提优化器 — [open-spaced-repetition/swift-fsrs](https://github.com/open-spaced-repetition/swift-fsrs)
- mamma-mia 把 swift-fsrs 固定在 main 的一个提交，因为 FSRS-6 只在 main 上，最新 tag v5.0.0 还是 FSRS-5 — `/home/user/li1000v2/mamma-mia/CLAUDE.md`
- ts-fsrs（MIT，约 831 star）v6 默认已是 FSRS-7，参数训练由 `@open-spaced-repetition/binding` 负责 — [open-spaced-repetition/ts-fsrs](https://github.com/open-spaced-repetition/ts-fsrs)
- srs-benchmark（约 1 万份 Anki 集合、约 3.5 亿条复习）：不含同日复习时 FSRS-6 Log Loss 0.3460、RMSE 0.0653；最佳 FSRS 变体 FSRS-7（recency, 100 epochs）0.3363；RWKV/GRU/LSTM 等神经网络更好；页面文字未列 SM-2 的数值（对比以图片形式给出）— [srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark)
- 第三方总结称 FSRS 在约 99.5% 的用户上比 SM-2 预测更准（二手资料，且 SM-2 本身不输出概率、需额外公式，有人认为对 SM-2 不公平）— [mindomax](https://www.mindomax.com/fsrs-vs-sm2-spaced-repetition-algorithm)；[Anki 文档 FAQ](https://docs.ankiweb.net/faqs/what-spaced-repetition-algorithm.md)
- mamma-mia 的卡片规则：每句默认两张卡（听辨 listen、回忆 recall）各自调度，同一句的两张卡不连着出；只有复习自评会更新 FSRS 状态，识别结果只作提示 — `/home/user/li1000v2/mamma-mia/CLAUDE.md`

### Inferences
- **调度**：沿用 swift-fsrs FSRS-6 默认参数即可；自用单人数据量小，参数优化价值有限（等累计上千条复习后再考虑导出日志用 Python fsrs-optimizer 训练，再把 21 个参数写回 App）。
- **粤语卡片类型建议**（每张独立调度，同源句子错开出现）：
  1. **听辨卡**（音频 → 意思/简体字）：最高优先，粤语口语的核心障碍是听。
  2. **开口回忆卡**（普通话意思 → 说粤语，录音 + 识别提示 + 自评）：对应 mamma-mia 的 recall。
  3. **普→粤口语对译卡**（“他在哪里？” → “佢喺边度？”）：针对普通话母语者最有价值，显式训练 他→佢、的→嘅、在→喺、没有→冇、东西→嘢、看→睇、了→咗 等高频替换；可只对“含高频差异词”的句子生成。
  4. **字/词 → 粤拼 + 声调卡**（看“嘢”说出 je5）：偏读写，适合白话字学习；与 Q6 的“读音预测器”联动（显示规律/例外）。
  5. **完形填空卡**（喺 / 嘅 / 咗 等虚词和句末助词 啦、喎、咩、噃）：训练白话文语法。
  6. **声调最小对立对**（如 si1–si6 诗史试时市事）：更适合做成**不进 FSRS 的感知训练**（按正确率自适应选择难区分的对：2/5 声、3/6 声、4/6 声等），把练习结果只作为统计；把它塞进 FSRS 会因为大量近似卡互相干扰而失真。
- **数据模型**：卡片 id = `句子id + 卡型`，FSRS 状态与内容分离，内容改动（如修正粤拼）不应重置卡片状态，除非原文改变（与 mamma-mia 的 `checkedText` 规则同理）。

### Gaps
- swift-fsrs 是否已支持 FSRS-7、最新提交与 tag 状态：页面未显示，需在仓库提交历史中确认。
- 未找到针对“语音/声调类卡片”与 FSRS 交互效果的研究。

## Q3. 发音与声调反馈：实时音高提取、说话人归一化、可视化、ASR 句子检查、强制对齐

### Takeaway
MVP 可行的声调反馈 = **端侧 YIN 音高提取（Accelerate 自写或移植）→ 按说话人做半音/z-score 归一化 → 映射到五度值 → 与管线预计算的参考曲线叠加显示**；“说这句话”检查用端侧 ASR，比较时**把识别结果和目标句都转成粤拼再按音节比对**（容忍粤语用字差异）。逐音节自动打分（强制对齐 + 声调分类）没有现成的开源粤语 CAPT 方案，放到后期。

### Cited Findings
**音高提取**
- pitchfinder（JS）集合多种算法，作者认为 YIN 是精度与速度的最佳平衡，但偶尔会给出离谱值；pitchy（JS）用 McLeod Pitch Method 适合实时 — [pitchfinder](https://github.com/peterkhayes/pitchfinder)；[pitchy](https://github.com/ianprime0509/pitchy)
- Swift 库 Beethoven 基于 AVAudioEngine 与 YIN 等策略，但最新版本 4.0.2 是 2018 年 1 月，可能已过时 — [Swift Package Registry: Beethoven](https://swiftpackageregistry.com/vadymmarkov/Beethoven)
- YIN 用累积均值归一化差分函数减少八度错误；取第一个低于阈值的谷而不是最深的谷可避免低八度错误；实践中先做 1 kHz 低通、加静音阈值 — [Rochester 课程报告](https://hajim.rochester.edu/ece/sites/zduan/teaching/ece472/projects/2018/BenjaminShafran_JoshuaMiller_BrentIkei_PitchCorrection_FinalReport.pdf)；逐帧检测器不做跨帧平滑，需自行加平滑 — [pitch-estimate crate](https://docs.rs/pitch-estimate/0.1.0)
- SwiftF0（MIT）：约 14,386 参数的神经网络音高估计器，范围约 46.9–2,094 Hz，单核 CPU 约 180 倍实时，流式需 176 ms 前瞻；在 19 个跟踪器的 Pitch Detection Benchmark 中 F1 最高（与 RMVPE 统计持平）；提供 ONNX 模型，README 未说明 iOS — [lars76/swift-f0](https://github.com/lars76/swift-f0)；[SwiftF0 论文](https://arxiv.org/pdf/2508.18440)
- 开源的 Mandarin 声调练习工具：MSU Tone Perfect 相关项目列表有 “Open Source Mandarin Tone Practice App by Cory Gallagher”（仅标题，仓库未核实）— [MSU Tone Perfect Related](https://tone.lib.msu.edu/related)；Shinytone 是面向研究者的浏览器端音高提取 + 曲线聚类 + 可视化工具 — [Shinytone poster](https://chenzixu.rbind.io/files/shinytone_poster.pdf)；商用的 ToneDetector 显示示范和用户的声调曲线 — [ToneDetector Lite](https://mwm.ai/ja/apps/tonedetector-lite/550854681)；Migaku 的 tone trainer 是订阅功能（听单音节选声调）— [Migaku blog](https://migaku.com/blog/chinese/chinese-language-tones)

**说话人归一化**
- 16 种声调归一化方法比较中，最好的是“相对每个说话人平均音高（Hz）的半音变换”— [Zhang, PACLIC 32 (2018)](https://preview.aclanthology.org/moar-dois/Y18-1095.pdf)
- 10 位说话人的粤语单字调，基线 z-score 归一化把说话人间方差降约 20 倍（指数 21.3），再用平调斜率校正 F0 下倾可再提升约 11%（23.6）— [Rose, SST 2022](https://sst2022.com/declination-adjusted-normalisation-of-cantonese-citation-tones/)；对数 z-score（LZ）用于儿童粤语声调 — [Lei, TAL 2006](https://www.isca-archive.org/tal_2006/lei06_tal.pdf)
- 粤语平调 T1 (55/53)、T3 (33)、T6 (22)；上下文整体上移两个半音时，听者可能把中平调听成低平调（说明平调判断依赖相对音高）— [PolyU 幻灯片](https://www.polyu.edu.hk/cbs/rclcn/images/publications/202312_ChenSi_AMLap_10099_BeiLi.pdf)

**ASR 句子检查**
- Whisper 与 Wav2Vec2 在粤语音节与声调信息上仍有困难 — [PMC 论文](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12298586/)；Whisper large-v3 新增了粤语语言 token — [openai/whisper-large-v3](https://huggingface.co/openai/whisper-large-v3)
- 端侧选项与精度对比见 Q1（SenseVoice 在粤语上优于 Whisper-Large-V3）— [FunAudioLLM](https://arxiv.org/pdf/2407.04051)
- 云端：Google Cloud Speech-to-Text 有香港粤语模型 `yue-Hant-HK`，有数据集用它做初稿再人工校正 — [Cantonese ASR datasets survey (arXiv 2201.02419)](https://arxiv.org/pdf/2201.02419)；Alibaba 有 `qwen3-asr-flash-2026-02-10` — [OpenRouter](https://openrouter.ai/qwen/qwen3-asr-flash-2026-02-10)
- Azure 发音评估（pronunciation assessment）支持语言列表中是否含 zh-HK：未能确认（官方概览只指向语言支持页）— [Microsoft Learn: language learning overview](https://learn.microsoft.com/en-my/Azure/ai-services/speech-service/language-learning-overview)

**强制对齐**
- MFA 没有官方粤语模型；社区仓库 HKCantonese_models 提供基于 Common Voice zh-HK 15.0 的两个声学模型（约 10 小时 train 集 / 约 106.5 小时 validated 集、2,325 位说话人）和 4,800+ 条词典；词典音素集必须与声学模型一致 — [chenchenzi/HKCantonese_models](https://github.com/chenchenzi/HKCantonese_models)
- Interspeech 2025：在香港粤语自然访谈上评估 MFA，元音对齐尚可，但粤语特有特征（如入声）和连读仍是主要难点 — [So et al., Interspeech 2025](https://www.isca-archive.org/interspeech_2025/so25_interspeech.pdf)
- torchaudio `MMS_FA`（MMS 项目，23,000 小时、1,100+ 语言训练）需要先把文本归一化/罗马化，罗马化方法按语言而定、教程不覆盖中文 — [torchaudio 多语言强制对齐教程](https://docs.pytorch.org/audio/2.8/tutorials/forced_alignment_for_multilingual_data_tutorial.html)

**CAPT 文献**
- CUHK 的粤语相关 CAPT 工作主要是“粤语母语者学英语”的音段错误检测（扩展词典 + ASR，与人工评判一致率 >84%）— [Wang, Feng & Meng, ICALIP 2008](https://www1.se.cuhk.edu.hk/~hccl/publications/pub/ICALIP2008_v1.pdf)；[Lee et al., Interspeech 2015](https://www.isca-archive.org/interspeech_2015/lee15b_interspeech.pdf)。未找到“学习者说粤语的声调错误检测”的论文。
- 新近粤语 ASR 研究：CantoASR（韵律感知 ASR + 大音频语言模型协作）— [arXiv 2511.04139](https://arxiv.org/pdf/2511.04139)；WenetSpeech-Yue 大规模粤语语料 — [arXiv 2509.03959](https://arxiv.org/pdf/2509.03959)

### Inferences
- **端侧音高实现（Swift）**：AVAudioEngine input tap（16 kHz 单声道，hop 10 ms，窗 40 ms）→ YIN（差分函数可用 vDSP 卷积/点积加速；核心算法写成纯 Swift 放核心包，Linux 测试用合成正弦/录音样本）→ 置信度 + 能量门限去清音 → 中值滤波 + 八度跳变修正 → 输出 (t, f0) 序列。SwiftF0 精度更好且模型极小，但需要 ONNX Runtime（若已集成 sherpa-onnx，ONNX Runtime 已在包里，可复用）；MVP 先用 YIN。
- **归一化与五度映射**：先做 10 秒“校准”——让用户读 6 个声调的 si 音节（诗史试时市事）或自然读几句，求说话人 F0 的均值与范围（取 5%/95% 分位）；显示时以半音为单位相对说话人均值（Zhang 2018 支持此法），再线性映射到 1–5 的五度刻度：`T = 1 + 4 × (st − st_min)/(st_max − st_min)`。参考曲线同样按参考说话人自身范围映射，两者在同一五度坐标系叠加，就能消除男女声差异。
- **可视化**：每个音节一个小格子（参考音高线 + 用户线），音节边界对参考音频在管线里预先算好（MFA 社区模型 / MMS_FA，罗马化可直接用去掉声调数字的粤拼）；用户录音的音节边界用 DTW 把用户能量/音高包络对齐到参考上（2–6 个音节的短句足够可靠）。先只“画出来 + 文字提示”（如“这里你读成了 33，示范是 35 升调”），不打分。
- **声调判定（后期）**：对单音节/双音节，可用规则（起点高度、终点高度、斜率、时长——入声短促）把归一化曲线分到 6 类，再与目标调比较；粤语 T2(35)/T5(23) 与 T3/T6 平调高度差是普通话者最难点，规则判定需要宽容区间并显示“置信度低”。
- **ASR 句子检查的比对单元**：识别出的文字（可能是繁体、可能用“系/係、D/啲、咁/噉”等不同写法）→ OpenCC 统一 → ToJyutping 转粤拼 → 与目标句粤拼逐音节对齐（编辑距离），分别报告“音节对不对”和“声调对不对”（ASR 有语言模型会“纠正”声调，所以 ASR 通过不代表声调对，声调以音高曲线为准）。这与 mamma-mia “只判断对方能不能听懂、不打发音分”的哲学一致。

### Gaps
- 没有找到经过验证的、开源的粤语声调 CAPT 系统或学习者粤语声调错误语料；逐音节自动打分的准确度无据可查。
- MFA 社区粤语模型、MMS_FA 对 TTS 合成音频的对齐质量未见评测。
- “Open Source Mandarin Tone Practice App”是否有公开仓库未核实。
- 五度值 T 值法（石锋）的原始文献本次未检索到，上面的映射公式是常用做法的推断。

## Q4. 音频内容管线：粤语 TTS 选型与成本、预生成与转写校验、多音字处理、真人录音

### Takeaway
预生成管线的首选是 **Google Cloud TTS Chirp 3: HD 的 yue-HK（2025-12-30 起 Preview）**——与 mamma-mia 现用的 Gemini TTS 同一套声线名（Kore、Charon 等）、成本可以忽略（每月 100 万字符免费，超出 $30/百万字符）；Azure zh-HK（HiuMaan 等）做第二声线/交叉校验。Gemini TTS 是否支持粤语**未证实**，需要实测。多音字的主要防线不是 SSML，而是**“生成 → 回转写 → 转粤拼 → 与标注粤拼逐音节比对 → 不一致就重试/人工”**。

### Cited Findings
**云 TTS 支持与价格（均需上线前复核）**
- Google Chirp 3: HD 语言表含 Chinese (Hong Kong) `yue-HK`，标注 Preview — [Chirp 3: HD voices](https://docs.cloud.google.com/text-to-speech/docs/chirp3-hd)；release notes：yue-HK 合成于 2025-12-30 进入 Preview — [Cloud TTS release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes)；Chirp 3: HD 有 30 个命名声线（Achernar、Kore、Puck、Zephyr 等），未按语言细分 — 同上
- Chirp 3: HD 价格：每月前 100 万字符免费，之后 US$0.00003/字符（US$30/百万字符）；Instant custom voice US$60/百万字符无免费额度；按字符计费，空格换行和 SSML 标签（除 mark）都计入 — [Cloud TTS pricing（意大利语页）](https://cloud.google.com/text-to-speech/pricing?hl=it)
- Gemini 3.8 Flash TTS（mamma-mia 当前所用 `gemini-3.8-flash-tts`）官方页称支持 130 种语言（Flash-Lite 101 种），并提到区域口音、方言与 IPA 发音替换，但摘录中未见粤语 — [Gemini 3.8 Flash TTS](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-tts?hl=zh-cn)；Cloud 版 Gemini-TTS 文档可见列表含 cmn-CN、cmn-tw（Preview），未见粤语 — [Gemini-TTS](https://docs.cloud.google.com/text-to-speech/docs/gemini-tts)；mamma-mia 配置：`ttsModel: gemini-3.8-flash-tts`、`checkModel: gemini-3.8-flash`、声线 Kore/Charon — `/home/user/li1000v2/mamma-mia/tools/tts_voices.json`
- Azure：中国区价格页（2026-07 更新）Neural ¥95.4/百万字符（含税），每月免费 50 万字符 — [Azure 中国定价](https://www.azure.cn/en-us/pricing/details/cognitive-services/)；全球区第三方称约 $15/百万字符（未能从官方页核实）— [speechactors](https://speechactors.com/article/microsoft-azure-pricing-and-plans)；zh-HK 有 HiuMaanNeural 等声线 — [json2video voice 页](https://json2video.com/ai-voices/azure/voices/zh-hk-hiumaanneural/)
- Amazon Polly：Hiujin 是女声粤语（yue-CN）神经声线 — [Polly neural voices](https://docs.amazonaws.cn/en_us/polly/latest/dg/neural-voices.html)；第三方目录列神经声线 $16/百万字符 — [GetApp](https://www.getapp.co.uk/software/2050357/amazon-polly)；用户报告 Hiujin 只有 neural、无 standard — [3CX 论坛](https://www.3cx.com/community/threads/amazon-polly-tts-does-not-have-cantonese-voice.130292/)
- Alibaba 托管的 qwen3-tts-flash 有 Rocky、Kiki 两个粤语声线（托管 API，不是开放权重）；开源 Qwen3-TTS（Apache-2.0，0.6B/1.7B）的公开说话人表中未见粤语 — [pyvideotrans Qwen-TTS](https://doc.pyvideotrans.com/qwen-tts)；[NYU Shanghai RITS](https://rits.shanghai.nyu.edu/ai/%F0%9F%93%A2-qwen3%E2%80%91tts-open%E2%80%91source-text%E2%80%91to%E2%80%91speech-tts-family/)

**多音字：SSML phoneme**
- Google（Assistant 文档）对 Cantonese (Hong Kong) 的 `<phoneme>` 要求 alphabet 设为 `jyutping`、声调数字放在音节末尾 — [Google SSML phonemes](https://developers.google.com/assistant/conversational/ssml-phonemes)
- Azure SSML 发音页摘录只提 `ipa` 和 `sapi` 字母表；`ph` 中含不认识的音素时返回 HTTP 400 — [Azure SSML 发音（docs.azure.cn）](https://docs.azure.cn/it-it/ai-services/speech-service/speech-synthesis-markup-pronunciation)；二手资料称 zh-HK 属于支持 SAPI 音素集的 8 个 locale 之一 — [suikawiki](https://wiki.suikawiki.org/n/alphabet$30968?format=xml)。**未找到 Azure 支持 Jyutping 字母表的证据。**

**g2p 与校验工具**
- ToJyutping（BSD-2-Clause；Python 包 `ToJyutping`，npm 包 `to-jyutping`）：README 称准确率 99%（未说明测法）；多字词条优先于单字读音（上堂 soeng5 tong4）；`get_jyutping_candidates` 返回按概率排序的候选读音；`customize()` 可覆盖词条；`g2p()` 输出声母/韵母/声调 ID 供 TTS/ASR 训练 — [CanCLID/ToJyutping](https://github.com/CanCLID/ToJyutping)
- PyCantonese（MIT，约 421 star）：语料检索、粤拼解析与转换、分词、词性标注；v4.0.0（2026-03）引入 Rustling 依赖；捆绑数据各有许可证（HKCanCor CC BY、rime-cantonese CC BY 4.0、Common Voice MPL 2.0 等）— [jacksonllee/pycantonese](https://github.com/jacksonllee/pycantonese)

**离线 TTS**
- sherpa-onnx 1.12.12 修复了 Cantonese VITS TTS 并不再为其插入空白 token — [sherpa_onnx_ios changelog 镜像](https://pub-web-s1.flutter-io.cn/packages/sherpa_onnx_ios/changelog)；第三方 HF Space 代码加载名为 `vits-cantonese-hf-xiaomaiiwn` 的 sherpa-onnx 格式粤语 VITS 模型 — [HF Space model.py](https://www.huggingface.co/spaces/deep-div/Text-to-Speech-TTS/blob/main/model.py)

### Inferences
- **成本量级（2026-10 价格，Chirp 3 HD）**：2,000 句 × 平均 12 字 × 2 声线 ≈ 4.8 万字符；即使 1 万句 × 15 字 × 3 声线 ≈ 45 万字符，也在每月 100 万免费额度内。慢速版用 ffmpeg `atempo` 从原速派生（mamma-mia 做法），不额外计费。→ **TTS 成本不是约束，质量（多音字、声调准确、口语字读法）才是。**
- **推荐管线（改 mamma-mia 的 `gen_audio.py`）**：
  1. 内容 JSON 每句存：`yue`（繁体源文本）、`yueSimp`（OpenCC 生成）、`jyutping`（逐字/逐音节数组，ToJyutping 生成后人工确认，状态机 draft → checked）、`zh`（普通话对照）。
  2. TTS：Chirp 3 HD `yue-HK`（声线名按 Google 的 `<locale>-Chirp3-HD-<Name>` 规律，沿用 Kore/Charon 便于与意语项目一致——命名规律需实测确认）；第二声线可用 Azure zh-HK，提供不同音色、也可作交叉验证。
  3. 回转写：用与 TTS **不同家族**的 ASR（Google STT `yue-Hant-HK`、qwen3-asr-flash 或本地 SenseVoice）转写 → OpenCC + ToJyutping → 与标注粤拼逐音节比对（先比声母韵母；声调另用音高曲线规则粗检）。不一致 → 重试最多 N 次 → 仍不一致写进 `reviews/`，人工听。
  4. 多音字修复手段（按优先级）：a) 先确认 TTS 在该上下文的读法，多数情况下上下文足够；b) Google 若在 Chirp 3 HD 上支持 `<phoneme alphabet="jyutping">` 就用（Assistant 文档有此字母表，Cloud TTS/Chirp 3 HD 是否支持需实测）；c) Azure 用 `sapi` 音素集（需查 zh-HK 的 SAPI 音素表，Jyutping 不直接可用）；d) 最后手段：在送 TTS 的文本里用同音无歧义字替换（只影响音频，不影响显示文本），并在 manifest 里记录。
  5. 参考音高曲线：同一管线用 parselmouth/pYIN 对每段原速音频算 F0，配合 MFA/MMS 的音节边界，写成 `Audio/pitch/<hash>.json` 随包分发，App 端只做用户侧计算。
- **声调最小对立对与单音节音频**：TTS 念单个字时声调/读音最不可靠（无上下文），建议单音节练习优先用真人录音（另一位研究员负责盘点可用的开放录音及许可证），或用 TTS 念“含该字的短词”再切出。
- **离线兜底**：没有预生成音频的句子（如 AI 对话生成的新句）先用 AVSpeechSynthesizer zh-HK（注意 iOS 18+ 的“朗读内容”设置覆盖 bug），或内置 sherpa-onnx 粤语 VITS。
- **真人录音**：自用可找一位母语者录 300–500 条核心句 + 6×N 个声调对立音节；作为 TTS 的“金标准”抽检（对应 mamma-mia 的 native 状态）。

### Gaps
- Gemini TTS（2.5/3.8）是否支持粤语：未证实；需用 yue/yue-HK 实测（若可用，就能完全沿用 mamma-mia 的 Gemini 管线与 key）。
- Chirp 3 HD yue-HK 是否已 GA、哪些声线可用、是否支持 SSML `<phoneme>` 与 Jyutping：未证实。
- Azure 全球区 2026 年 Neural TTS 美元价格与 zh-HK SSML 音素支持细节：官方页本次无法访问。
- Polly 美国区官方价格：官方页无法访问，只有第三方数字。
- `vits-cantonese-hf-xiaomaiiwn` 的音质与许可证：未核实。

## Q5. 用 LLM 生成内容与做对话伙伴：生成/转写/标注流程、质量控制、实时语音模型、成本

### Takeaway
LLM 适合生成分级口语对话、把普通话句子改写成地道粤语口语、写给普通话者的讲解，但**粤拼必须由词典型 g2p（ToJyutping/PyCantonese）生成、LLM 只做复核**；“是不是粤语口语”可用 canto-filter 这类规则分类器自动把关。对话伙伴延续 mamma-mia 的“录音 → 端侧识别 → 文本 LLM → 分句 TTS 流式播放”管线最稳；端到端实时语音模型中只有 Alibaba Qwen Omni 系列明确标注支持粤语，Gemini Live / OpenAI Realtime 的粤语支持未证实。成本极低：内容生成一次性几美元，对话每天几美分。

### Cited Findings
**粤语能力评测**
- Jiang et al.（Findings of NAACL 2025）：从英语/普通话翻译并经双语审核的 4 套粤语基准（事实生成、数学逻辑、复杂推理、常识），评测 35 个开源/闭源模型 — [ACL Anthology 2025.findings-naacl.253](https://preview.aclanthology.org/dashboard/2025.findings-naacl.253)
- HKCanto-Eval：粤语理解基准，兼顾英语与书面中文的跨语言评测，融入香港文化语言细节；作者认为以往翻译型基准偏向美国文化或大陆考试 — [arXiv 2503.12440](https://arxiv.org/abs/2503.12440v1)
- canto-filter（CanCLID）把文本分为 cantonese / mandarin / mixed / neutral 四类，基于粤语与官话特征词判别，偏重 precision；假设输入为繁体（简体需先用 OpenCC 转换）且采用推荐用字（如写“比本书我”会被判 neutral，“畀本书我”才判 cantonese）；更精细的 6 类判别可用 `cantonesedetect` — [CanCLID/canto-filter](https://github.com/CanCLID/canto-filter)

**实时/语音模型的粤语支持**
- Alibaba Qwen3-Omni-Flash 文档：11 种输出语言 + 8 种中文方言，实时与文件模式均可用，语言表标注支持粤语；实时语音对话指向 Qwen3.5-Omni Realtime（WebSocket）— [Alibaba Model Studio: Qwen-Omni](https://help.aliyun.com/en/model-studio/qwen-omni)；The Decoder 报道 Qwen3.5-Omni 语音识别覆盖 74 种语言、39 种中文方言，Fleurs 粤语 1.95（对比模型 13.40）— [The Decoder](https://the-decoder.com/?p=33841)
- Gemini Live API：“24 种支持语言”，native audio 模型 30 个 HD 声线、可无配置切换语言；推荐 GA 模型 `gemini-live-2.5-flash-native-audio`；未找到含粤语的语言清单 — [Vertex AI Live API](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/live-api)
- OpenAI：gpt-realtime（2025-08）可在一句话内切换语言，但未见语言清单；GPT-Realtime-Translate 输入 70+ 种语言、输出 13 种 — [OpenAI 官方博文](https://openai.com/vi-VN/index/advancing-voice-intelligence-with-new-models-in-the-api/)；集成商称 OpenAI Realtime 多语言“不完全可靠” — [Vapi 社区](https://support.vapi.ai/t/33185864/realtime-gpt-transcriber)

**价格（每百万 token；均标注日期/来源）**
- Anthropic（官方定价，skill 缓存 2026-10-06）：Claude Haiku 5.5 $0.10 入 / $0.50 出（提示 ≤100K token；超出为 $0.50/$2.50）；Claude Sonnet 5.5 $2 / $10；Claude Opus 5.5 $4 / $20；Batch API 五折 — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- Google（第三方汇总 2026）：Gemini 2.5 Flash $0.30 / $2.50；Gemini 2.5 Flash-Lite $0.10 / $0.40；另有表格列 gemini-3.1-flash-lite $0.25 / $1.50 — [morphllm](https://www.morphllm.com/gemini-api-pricing)
- OpenAI（第三方计算器）：gpt-realtime $4 / $16（未区分音频 token 价）；gpt-realtime-mini $0.60 / $2.40 — [getmaxim gpt-realtime](https://www.getmaxim.ai/bifrost/llm-cost-calculator/provider/openai/model/gpt-realtime)；[gpt-realtime-mini](https://www.getmaxim.ai/bifrost/llm-cost-calculator/provider/openai/model/gpt-realtime-mini-2025-12-15)

**mamma-mia 的对话管线（可直接迁移）**
- 一回合：录音 → 离线识别（同时播语气词）→ 文本模型流式（6 秒没出字或 429/5xx 就换备用模型）→ 回复按句切开 → 每句送 TTS 流式合成、边收边播（24 kHz 16-bit PCM）；key 存钥匙串；每日回合上限默认 40；对话文字不写日志；结束后反馈卡（最多 3 处纠正 + 1 个新表达）生成新卡 — `/home/user/li1000v2/mamma-mia/CLAUDE.md`

### Inferences
- **内容生成工作流（建议）**：
  1. 人定大纲（场景 × 句框 × 目标词/语法点，难度分级）。
  2. LLM 生成：给出“只用香港/广州日常口语、用推荐用字（畀/嘅/咗/喺/啲）、不要书面语”等约束，输出 JSON（繁体句、普通话对照、给普通话者的讲解、关键差异词）。用 Claude Sonnet 5.5/Opus 5.5 或 Gemini（沿用用户 key）均可；生成后再用**另一家模型**交叉审校（mamma-mia 的 `cross_check.py` 模式）。
  3. 自动检查：canto-filter 判为 cantonese/mixed 才通过（neutral 句子允许但标记）；ToJyutping 标注粤拼（`get_jyutping_candidates` 有多个候选时标记“需人工确认多音字”）；简体由 OpenCC 生成；JSON Schema 校验；与已有句子去重。
  4. 人工/母语者抽检后才标 checked/native；改原文即退回 draft（`checkedText` 规则照搬）。
- **为什么不让 LLM 直接出粤拼**：LLM 在逐字罗马化上易出现声调数字错误、把普通话读音迁移（如韵尾 -p/-t/-k 丢失），而词典型 g2p 在已收录词上近乎确定；让 LLM 只做“候选读音哪个对”的判断题更可靠（推断，未见专门评测）。
- **对话伙伴**：
  - MVP 后的第一版：沿用 mamma-mia 管线，把识别换成 zh-HK/yue（或 sherpa-onnx SenseVoice），文本模型用用户的 Gemini key（或 Claude），TTS 换成粤语（Chirp 3 HD yue-HK 走 Cloud TTS 流式，或回退 AVSpeechSynthesizer zh-HK）。回复要求两行：粤语（繁体）+ 普通话翻译，App 用 OpenCC（Swift 侧需要一个 OpenCC 实现或预先让模型同时输出简体）显示简体。
  - 端到端实时语音：Qwen3.5-Omni Realtime 是目前唯一明确标注粤语的选项，但需要阿里云账号，且数据出境/隐私条款需另看；Gemini Live 是否能稳定说粤语要实测（若可用，与用户已有 Google key 最契合）。
- **单用户成本估算（推断）**：
  - 内容生成：300 次请求 × (3k 入 + 2k 出) ≈ 0.9M 入 + 0.6M 出 → Sonnet 5.5 ≈ $1.8 + $6 = $7.8，Batch 五折约 $3.9；用 Opus 5.5 约 $15.6（Batch 约 $7.8）。一次性成本。
  - 对话：每天 40 回合 × 2k 输入（含历史，开启 prompt caching 可再降）+ 150 输出：Haiku 5.5 ≈ 8 万入 $0.008 + 6 千出 $0.003 ≈ $0.01/天；Sonnet 5.5 ≈ $0.16 + $0.06 ≈ $0.22/天；TTS 每天约 1,600 字符（超免费额度后约 $0.05）。→ 每月不到 $1–7。

### Gaps
- 未拿到 Jiang et al. 与 HKCanto-Eval 中各模型（GPT/Claude/Gemini/Qwen/DeepSeek）的具体分数；也没有针对“口语粤语生成地道程度”的公开评测。
- Gemini Live、OpenAI gpt-realtime 是否支持粤语输入/输出：未证实。
- DeepSeek、Qwen-Plus 的 2026 官方 API 价格：未找到可靠来源。
- Qwen Omni Realtime 的粤语音色质量与价格：官方页本次无法访问。

## Q6. “普通话 → 粤语读音预测器”：用中古音（切韵/广韵）数据展示规律与例外

### Takeaway
可行且有现成开源基础：nk2028 的 **tshet-uinh-js（MIT）+ tshet-uinh-examples 里的 `gwongzau.js`（推導廣州話）与 `putonghua.js`（推導普通話）**，可在构建期为每个字算出“按规律应读的粤语音”，与词典实际粤拼比较，标出“规律 / 例外 / 文白异读”，App 只读预计算结果。

### Cited Findings
- nk2028 组织有 65 个仓库：tshet-uinh-js（TypeScript，MIT，切韵音系 JS 库）、tshet-uinh-deriver（MIT，在线推导工具，含“推导”出的历史与现代音系）、tshet-uinh-data（古代韵书机读数据）、tshet-uinh-examples（MIT，推导示例脚本）、ToMiddleChinese（Python，给汉字加中古音）、opencc-js（约 361 star）、yindian（字音查询）等 — [github.com/nk2028](https://github.com/nk2028)
- tshet-uinh-examples 的 21 个推导脚本中包括 `gwongzau.js`（推導廣州話 Extrapolated Cantonese）、`putonghua.js`（推導普通話）、`zaonhe.js`（上海话）、`taibu.js`（大埔客家话），以及 Karlgren/王力/潘悟云/unt/Baxter 等拟音与 TUPA 转写 — [nk2028/tshet-uinh-examples](https://github.com/nk2028/tshet-uinh-examples)
- 学术背景：有论文假设粤语源自“夜下”音系并逆推，涉及全浊清化、咝音系列与韵母差异 — [NCCU 论文摘要](https://nccur.lib.nccu.edu.tw/handle/140.119/143716)

### Inferences
- **构建期计算（Node 脚本，放 `tools/`）**：对内容中出现的每个字 → tshet-uinh-js 查音韵地位（一字多地位时全部保留）→ 跑 `gwongzau.js` 得“推导粤音”、跑 `putonghua.js` 得“推导普通话音” → 与实际粤拼（rime-cantonese/词典，由另一研究员确认许可证）和实际拼音比较 → 生成 `Content/readings.json`：`{字, 实际粤拼, 推导粤拼, 是否规律, 规则标签}`。
- **给学习者看的规则（举例方向）**：普通话声调 + 声母清浊 → 粤语阴阳调（古清声母 → 阴调 1/2/3，古浊声母 → 阳调 4/5/6）；普通话无入声，但古入声字在粤语保留 -p/-t/-k（如“十 sap6、日 jat6、六 luk6”）；普通话 j/q/x 来自古见组和精组 → 粤语分别多为 g/k/h 与 z/c/s（“见 gin3” vs “箭 zin3”）。界面上把同规则的字聚成“家族”，例外单独列出（文白异读如“生 saang1/sang1”另起一类）。
- **赣语加成（推断，未查证）**：用户的赣语母语若保留入声或古浊声母痕迹，可在规则卡中加一行“赣语对照”，但需用户自己验证其方言点的读音，不建议自动化。
- **App 端**：读音预测器作为“字卡背面的解释”和独立的“规律浏览”页即可，不需在 iOS 里跑 JS（如果想在线推导，可用 JavaScriptCore 加载 tshet-uinh-js，但没必要）。

### Gaps
- `gwongzau.js` 的输出格式（是否为粤拼、IPA 或自定义记法）、推导准确率（与实际广州话读音的吻合率）未核实。
- tshet-uinh-data 的许可证在组织页未显示，需查看仓库。
- 普通话→粤语“直接映射”（不经中古音）的规则库是否有开源实现：未找到。

## Q7. 后端、同步、鉴权、分析与隐私（PIPL / PDPO）

### Takeaway
自用 iOS 版**不需要后端**：学习进度存本地 JSON + 导出/导入备份（mamma-mia 现状），云 API key 存钥匙串；若将来要多设备同步或给别人用，再上 Supabase（海外）或腾讯云开发 CloudBase（大陆）。录音默认只在本机，发往云端识别/对话前给出明确提示——面向大陆用户时语音可能属敏感个人信息，需单独同意。

### Cited Findings
- mamma-mia 的做法：行程/打卡等可选字段与学习进度同一个文件、跟随导出备份；照片存 Application Support 不进备份；对话内容、行程字段不写日志；免费 Apple ID 下不用 iCloud — `/home/user/li1000v2/mamma-mia/CLAUDE.md`
- Supabase（第三方 2026 汇总，2026-09-30 对照官方页）：Pro $25/月/组织，含 8 GB 数据库、250 GB 出流量、10 万 MAU；Free 为 5 万 MAU、500 MB 数据库、5 GB 出流量、最多 2 个项目，闲置一周会暂停；Pro 计算实例不缩容到零 — [makerkit](https://makerkit.dev/blog/saas/supabase-pricing)；[activepieces](https://www.activepieces.com/blog/supabase-pricing-free-tier-limits-pro-costs-egress)
- 腾讯云开发 CloudBase：官方定价页强调先用免费层验证、按资源点与套餐计费，具体价格以购买页为准；有活动称体验版可分享免费续期 6 个月或 4.9 元升级（时效不明）— [CloudBase pricing](https://docs.cloudbase.net/en/pricing)；[腾讯云开发者社区](https://cloud.tencent.com/developer/team/tcb)
- PIPL：生物识别信息与不满 14 周岁未成年人的个人信息属于敏感个人信息；处理敏感个人信息需“单独同意”并告知必要性与影响；处理未满 14 周岁者信息需监护人同意 — [IAPP](https://iapp.org/news/a/top-5-operational-impacts-of-chinas-pipl-part-1-scope-key-definitions-and-lawful-handling-of-personal-information)；[Bloomberg Law PIPL FAQ](https://pro.bloomberglaw.com/brief/china-personal-information-protection-law-pipl-faqs/)；配套的《个人信息安全规范》建议对敏感信息设置独立同意机制 — [Conventus Law](https://conventuslaw.com/report/feature-article-on-chinas-personal-information-3/)

### Inferences
- **自用版数据设计**：`StudyProgress` 单文件（FSRS 卡状态 + 复习日志 + 声调练习统计 + 设置），只做加法改动；录音只存最近 N 条供“对比回放”，默认不进备份；参考音高 JSON 随包。
- **同步需求出现时**：免费 Apple ID 无 iCloud，可选 a) 手动导出到“文件”App；b) Supabase Free（注意闲置一周暂停，对“偶尔同步”的自用场景不友好）；c) 直接把备份文件推到用户自己的私有 GitHub 仓库/对象存储。
- **语音隐私**：声纹/语音是否被认定为 PIPL 下的“生物识别信息”取决于用途（用于识别身份时更明确）；保守做法是把“上传录音到云端 ASR/LLM”做成显式开关 + 首次单独同意说明，并在日志中不记录识别文本（mamma-mia 已如此）。香港 PDPO 方面本次未检索到针对语音的具体条文。
- **Firebase**：常见说法是 Google 服务在大陆不可直连，因此面向大陆用户时应改用 CloudBase/LeanCloud 一类国内 BaaS（本次未检索到权威来源，仅作背景）。

### Gaps
- CloudBase 个人版 2026 年的确切价格与额度：未找到。
- 香港 PDPO 对录音数据的具体要求、PIPL 是否明确把“语音”列为生物识别信息：未找到权威原文。
- Firebase 在大陆可用性的权威来源：未检索。

## Q8. 变现与成本（自用为主，兼顾将来公开发布）

### Takeaway
自用版的月度云成本在**几美元以内**（TTS 预生成基本落在免费额度、对话每天几美分、无后端）；真正的“成本”是 Apple 免费签名每 7 天续签和内容审校的人力。若将来公开发布，需要付费开发者账号与后端，订阅制是语言学习 App 的常见模式，但本次没有检索到 2026 年可靠的行业定价/转化基准。

### Cited Findings
- Chirp 3 HD 每月前 100 万字符免费，之后 $30/百万字符 — [Cloud TTS pricing](https://cloud.google.com/text-to-speech/pricing?hl=it)
- Azure Neural TTS 每月 50 万字符免费，中国区 ¥95.4/百万字符 — [Azure 中国定价](https://www.azure.cn/en-us/pricing/details/cognitive-services/)
- Claude Haiku 5.5 $0.10/$0.50、Sonnet 5.5 $2/$10 每百万 token，Batch 五折 — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- Supabase Pro $25/月起，小型生产应用第三方估计 $35–75/月 — [makerkit](https://makerkit.dev/blog/saas/supabase-pricing)
- 免费 Apple ID：签名 7 天、3 个侧载名额、不能用 APNs/iCloud — `/home/user/li1000v2/mamma-mia/CLAUDE.md`

### Inferences
- **自用版月成本（推断）**：TTS 0（免费额度内）；LLM 内容生成一次性 $4–16；AI 对话 $0.3–7/月（看模型）；云 ASR（若不用端侧）按分钟计费、量很小；后端 0。合计通常 < $10/月。
- **公开版（若有）**：Apple Developer Program 年费（金额本次未检索）+ 后端 $25–75/月 + 按活跃用户计的 TTS/LLM；免费层放 Jyutping 入门与基础课，订阅解锁 AI 对话与声调评分是自然的切分（因为这两项有边际成本）。

### Gaps
- 2026 年语言学习 App 订阅价格与转化率基准（如 Duolingo、RevenueCat 报告）：本次未检索。
- Google STT `yue-Hant-HK`、Azure STT 的 2026 年每分钟价格：未检索到官方数字。

## Q9. MVP 范围（4–8 周）与分阶段路线图

### Takeaway
MVP 应聚焦“**会听、会认粤拼声调、能开口说核心句**”：粤拼与声调入门 + 情景句课程（含普→粤对译）+ FSRS 复习（听辨/回忆/对译）+ 点字查粤拼的阅读器 + 端侧识别提示 + 只画不打分的声调曲线。AI 对话、逐音节声调评分、读音预测器、社区内容放到后续阶段。复用 mamma-mia 的工程骨架可把时间压到 4–6 周。

### Cited Findings
- 可复用的 mamma-mia 组件：XcodeGen 工程、MammaCore（FSRS、内容解析、识别比对、RolePlay 请求/SSE 解析/错误分类）、`gen_audio.py`/`audio_review.py`/`cross_check.py`/`validate_content.py` 工具、CI 出 IPA 流程、诊断与备份页 — `/home/user/li1000v2/mamma-mia/CLAUDE.md`
- 端侧粤语识别可用 sherpa-onnx SenseVoice（中英日韩粤）— [k2-fsa/sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx)
- 粤语 TTS 预生成可用 Chirp 3 HD yue-HK（Preview）— [Chirp 3: HD](https://docs.cloud.google.com/text-to-speech/docs/chirp3-hd)
- g2p/校验：ToJyutping、PyCantonese、canto-filter — [ToJyutping](https://github.com/CanCLID/ToJyutping)；[PyCantonese](https://github.com/jacksonllee/pycantonese)；[canto-filter](https://github.com/CanCLID/canto-filter)
- 中古音推导：tshet-uinh-js + gwongzau.js — [tshet-uinh-examples](https://github.com/nk2028/tshet-uinh-examples)

### Inferences
**第 0 周（2–3 天）：技术探针（决定后续路线的实测）**
- 真机跑：`SpeechTranscriber`/`DictationTranscriber`/`SFSpeechRecognizer` 对 yue-CN、zh-HK 的支持与端侧可用性、输出用字；AVSpeechSynthesizer zh-HK 声线列表；𨋢/𡃁/𠮶 等字是否豆腐。
- 管线跑：Gemini 3.8 Flash TTS 能否念粤语；Chirp 3 HD yue-HK 声线列表与 SSML `<phoneme alphabet="jyutping">` 是否生效；Azure zh-HK 试听。
- 结论写进仓库 `docs/`，决定 ASR 主路径（Apple vs sherpa-onnx）和 TTS 主声线。

**第 1–2 周：内容与核心包**
- 新建/改造核心包：粤拼解析与校验（声母/韵母/声调、入声判定）、内容模型（繁体源文本、简体、逐音节粤拼、普通话对照、讲解）、FSRS 卡型（listen / recall / zh2yue / char2jyutping）。
- 工具：`annotate.py`（ToJyutping + OpenCC + canto-filter + 多音字候选报告）、改 `gen_audio.py` 为粤语 TTS + 回转写粤拼比对、`validate_content.py` 加粤拼与 Ext B 字体覆盖检查。
- 内容：粤拼与声调入门 8–10 课（对比拼音：z/c/s、j=拼音 y、gw/kw、aa/a、oe/eo、yu、-m/-p/-t/-k 韵尾、6 调/9 调）；情景句 150–300 句（问路、点餐、购物、交通、寒暄），每句带“普通话怎么说 → 粤语口语怎么说”的差异词。

**第 3–4 周：App 功能**
- 课程页、复习页（听辨/回忆/对译/字→粤拼）、粤拼声调着色（6 色）、慢速播放。
- 阅读器：点字/词 → 粤拼 + 释义（离线 SQLite 或 JSON 词表，来源与许可证由另一研究员确认）、简繁切换。
- 开口说：录音 → 端侧识别 → 粤拼级比对提示（不改 FSRS 状态）。
- 声调曲线 v0：YIN + 说话人校准 + 五度坐标叠加参考曲线（参考曲线由管线预计算）。

**第 5–6 周（缓冲 / 打磨）**：母语者抽检核心句、诊断页（识别模型、音频清点）、备份导入导出、CI 出 IPA。

**第 2 阶段（第 7–12 周）**
- AI 角色扮演（沿用 mamma-mia 管线：识别 → Gemini/Claude 文本 → 粤语 TTS 流式；每日上限；反馈卡生成新卡）。
- 读音预测器（tshet-uinh 推导 → 规律/例外浏览、字卡背面解释）。
- 声调最小对立对自适应感知训练（真人录音优先）。

**第 3 阶段（3 个月后）**
- 逐音节声调判定（规则分类 + DTW；再考虑小模型）、粤语白话文阅读材料（分级短文、歌词/影视台词需注意版权）、粤语输入（可借鉴 TypeDuck/rime-cantonese）。
- 若公开发布：付费开发者账号、后端同步、隐私合规（大陆则需 ICP 备案与国内 BaaS）。

### Gaps
- 工期估计基于 mamma-mia 的复用程度推断，未有实测数据。
- 入门课的教学顺序（先声调还是先韵母）缺少针对普通话母语者的实证研究，本次未检索。

## Q10. 值得参考的开源项目（已核实存在性）

### Takeaway
最值得读代码的是：**Jyut Dictionary**（离线粤语词典的 SQLite 构建与搜索）、**sherpa-onnx**（端侧粤语 ASR/TTS 的 Swift 接入）、**swift-fsrs / srs-benchmark**（调度）、**ToJyutping / PyCantonese / canto-filter**（文本处理与质检）、**nk2028 tshet-uinh 系列**（读音预测）；LibreLingo 已归档，只适合参考课程数据格式。

### Cited Findings
- Jyut Dictionary（aaronhktan/jyut-dict，MIT，约 161 star）：Windows/macOS/Linux 离线粤语词典，Qt 6.8+ / C++，Python 脚本构建 SQLite `dict.db`（`parse-set.py` 处理 CEDICT + CC-CANTO 等），宣称 20 万+ 词条、10 万例句（CEDICT、CC-CANTO、words.hk、Unihan、开放词典、Tatoeba），支持繁/简、粤拼、拼音、英文即时搜索 — [aaronhktan/jyut-dict](https://github.com/aaronhktan/jyut-dict)
- LibreLingo（kantord/LibreLingo，AGPL-3.0，约 2.6k star）：社区语言学习平台，Svelte + PouchDB，YAML 课程 → JSON；**已归档、作者不再维护**，指向 Codeberg 上的 LibreLingoRelive 等分支 — [kantord/LibreLingo](https://github.com/kantord/LibreLingo)
- sherpa-onnx（Apache-2.0）— [k2-fsa/sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx)；WhisperKit（MIT）— [argmaxinc/WhisperKit](https://github.com/argmaxinc/WhisperKit)
- swift-fsrs（MIT）、ts-fsrs（MIT）、srs-benchmark — [swift-fsrs](https://github.com/open-spaced-repetition/swift-fsrs)；[ts-fsrs](https://github.com/open-spaced-repetition/ts-fsrs)；[srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark)
- ToJyutping（BSD-2）、PyCantonese（MIT）、canto-filter — [ToJyutping](https://github.com/CanCLID/ToJyutping)；[PyCantonese](https://github.com/jacksonllee/pycantonese)；[canto-filter](https://github.com/CanCLID/canto-filter)
- nk2028：tshet-uinh-js、tshet-uinh-examples（含 gwongzau.js）、opencc-js — [nk2028](https://github.com/nk2028)；[tshet-uinh-examples](https://github.com/nk2028/tshet-uinh-examples)
- HKCantonese_models（MFA 粤语声学模型）— [chenchenzi/HKCantonese_models](https://github.com/chenchenzi/HKCantonese_models)
- SwiftF0（MIT，神经音高估计）— [lars76/swift-f0](https://github.com/lars76/swift-f0)；pitchfinder / pitchy（JS 音高检测）— [pitchfinder](https://github.com/peterkhayes/pitchfinder)；[pitchy](https://github.com/ianprime0509/pitchy)
- TypeDuck（粤语输入法 Web 版）仓库存在 — [TypeDuck-HK/TypeDuck-Web](https://github.com/TypeDuck-HK/TypeDuck-Web)
- 商用参考：Migaku（订阅制 tone trainer）— [Migaku blog](https://migaku.com/blog/chinese/chinese-language-tones)；ToneDetector — [ToneDetector Lite](https://mwm.ai/ja/apps/tonedetector-lite/550854681)

### Inferences
- 读 Jyut Dictionary 的 `parse-*.py` 学习多源词典合并成 SQLite 的 schema（词条/读音/释义/例句多表），对 iOS 端用 GRDB 或系统 SQLite 做离线查词很有参考价值；但它是 Qt 桌面程序，UI 代码对 SwiftUI 无参考意义。
- mamma-mia 本身已经是比 LibreLingo/Anki 更贴合的架构模板；Anki/AnkiDroid 的价值主要在“卡片模板 + 笔记/卡片分离”的数据模型（一条笔记生成多张卡），与本方案“一句多卡型”一致。
- “Lingva”实为 Google 翻译的替代前端（非学习 App），与本项目关系不大（基于背景知识，本次未核实）。

### Gaps
- “CantoneseTools”、“Language Reactor”、“Lingva”、AnkiDroid 的仓库/产品细节本次未核实。
- 未找到开源的、带声调曲线反馈的粤语学习 App；“Open Source Mandarin Tone Practice App” 的仓库未核实。
