# 复刻 mamma-mia，把普赣读音变成粤语捷径

做这个粤语 App，最省力也最有差异化的路线，是把 mamma-mia 的工程骨架几乎原样复制成 Diu。可以照搬的部分包括：SwiftUI + XcodeGen；只依赖 Foundation 和 swift-fsrs（FSRS-6）、能在 Linux 上测试的核心包；JSON + Schema 内容和 `draft → checked → native → audio_ok` 状态流水线；预生成音频的转写校验、AI 粗筛和人工听审；离线识别；用用户自己的 Google key 做角色扮演；GitHub Actions 出未签名 IPA，再用 SideStore 侧载。粤语专属的改造只有三处。第一，内容模型以繁体为唯一权威文本，逐字对齐粤拼，并用 CC BY 4.0 的开放字表做确定性校验。第二，识别比对、切句和音频校验都按“字/音节”而不是按“词”来做，因为 mamma-mia 现有的分词会把一整句汉字当成一个词。第三，加一套发音训练：用“普通话读音 + 赣语读音”预测粤语读音，专攻 T3/T6、T5、入声和 -m 韵尾 ([mamma-mia CLAUDE.md](https://github.com/li1000v2/mamma-mia/blob/main/CLAUDE.md))。市场恰好空着这一格：面向普通话使用者的系统课程只有多邻国一门，到 2026 年 7 月才扩到“早期 A2”；也没有任何产品把普→粤或赣→粤的读音对应当作教学方法 ([duolingodata](https://duolingodata.com/dailynews.html))。对这位学习者来说，认字几乎不成问题：约 25 个高频方言字就覆盖了书面粤文九成以上的方言字出现 ([Cifu](https://github.com/gwinterstein/Cifu))。真正要学的是每个熟字的粤音、几百个口语差异词、句末助词，以及原速听力。赣语能帮多少，取决于方言点。“古全浊声母不论平仄都读送气”是赣语的共同特征，可以补上普通话推不出“去声读 3 还是 6”的那一格。入声和 -m/-p 能迁移多少，则要看用户说的是南昌话、抚州话还是宜春话，而用户还没说明。两个最大的技术未知数要在开工第一周用真机和真实调用探明：iPhone 上的 SpeechTranscriber 有没有可用的粤语模型，mamma-mia 现用的 Gemini TTS 能不能稳定念粤语。按现有价格估算，自用的云成本每月不到 10 美元。

## 约 25 个方言字撑起九成粤文，难的是熟字的粤音

香港、广州一带是典型的双层语言社会（diglossia）。正式书写用以普通话为基础的标准书面语（SWC）；粤语白话文主要出现在社交媒体、LIHKG 论坛、小报、网络小说和广告里，用户在两者之间常常不加标注地混用 ([Lau et al. 2024](https://aclanthology.org/2024.eurali-1.4))。CanCLID 的分类器因此把文本分成粤文、官话文、官粤混杂和无特征四类。大量日常句子（如“去學校讀書”）本身就分不出粤官 ([canto-filter](https://github.com/CanCLID/canto-filter))。主流电视剧和流媒体字幕仍以 SWC 为主，粤文字幕多见于老港片和儿童片 ([CantoBlog](https://blog.cantoblog.com/2020/01/how-to-start-learning-cantonese-after.html))。所以 Diu 要教两件不同的事。一是“用粤音读 SWC”：学习者认识全部汉字，只缺读音。二是“读写粤语白话文”：要新学一批方言字和语法词。两者共用同一套读音数据，但练法不同：前者靠听辨和读音推导，后者靠口语字和句末助词。

方言字本身非常集中。对 Cifu 香港粤语词频库（LREC 2020，GPL-3.0）书面语料的统计如下：canto-filter 定义的 41 个“粤语独有字”合计只占全部字次的 **6.67%**；其中前 3 个（佢、嘅、咁）占这些字出现次数的一半，**前 13 个占 90%，前 23 个占 99%**。把“唔”和常用句末语气词也算进来，扩展集占 10.16%，前 18 个占 90%。在成人口语转写里，严格集占 **13.23%**。同一书面语料中，最高频的 1,176 个字就覆盖 95% 的字次，而这些字学习者几乎全认识 ([Cifu](https://github.com/gwinterstein/Cifu))。所以第一层方言字清单定 25 个左右就够了：佢 嘅 咁/噉 咗 啲 嚟 冇 睇 諗 喺 哋 嗰 乜 搵 嘢 啱 瞓 畀 唔 咩 㗎 喎 啦 喇 嘞。工作量真正大的是三块：300–500 个口语特有词，体标记和句末助词，以及每个熟字的粤音（含文白异读和变调）。Cifu 是 GPL-3.0，适合在 `tools/` 里做排序统计，原始数据不要打包进 App。

粤文没有官方正字法，但社区的事实标准很清楚。jyutping.org 的《粵文常見錯別字》和 rime-cantonese 都遵循“一字一语素”：嘅不写既，咗不写左，嘢不写野，嚟不写黎，畀不写俾；係（是）与喺（在）分开，噉（这样）与咁（这么）分开 ([jyutping.org 錯別字表](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/typo.mdx)；[rime-cantonese 用字原则](https://github.com/rime/rime-cantonese/wiki/%E8%A9%9E%E6%A2%9D%E7%94%A8%E5%AD%97%E5%8E%9F%E5%89%87))。真实网络文本却大量使用同音替代字。在 Cifu 书面语料里，“既”约占 ge3 书写的 **37%**，“左”约占 zo2 的 **56%**，“野”约占 je5 的 45%，“俾”约占“给”义的 **80%**，gam2 几乎一律写成“咁” ([Cifu](https://github.com/gwinterstein/Cifu))。因此标准答案采用推荐用字；判分时把这些常见异写判为“可接受的网络写法”，同时提示规范写法。阅读真实 UGC 前，可以先用 typo-corrector 一类的规则做规范化 ([typo-corrector](https://github.com/CanCLID/typo-corrector))。

编码和字体是第二个坑。核心粤字大多在 Unicode 基本区；㗎、㩒 在扩展 A，𨋢（升降机）、𡃁、𠺢 在扩展 B。GB2312 不含嘅、咗、喺、嘢、啲、嚟、哋、啱、瞓，原版 Big5 也缺这批字，要靠 Big5-HKSCS 才有 ([LSHK 字表](https://github.com/lshk-org/jyutping-table)；[Unicode Unihan](https://www.unicode.org/charts/unihan.html))。扩展 B 字在不少字体里会显示成方框 ([Launchpad bug 1270647](https://launchpad.net/bugs/1270647))。jyutping.org 的做法是自带字体，并按页面实际用字做子集化 ([jyutping.org README](https://github.com/CanCLID/jyutping.org))。

简繁问题比看上去棘手。多数粤语专用字没有简化形式，嘅、咗、啲、喺、嘢、嚟、佢、冇、畀、睇、瞓都是简繁同形。但 OpenCC 默认的 `t2s` 会把 **係→系**、**嗰→𠮶（扩展 B）**、喎→㖞、餸→𩠌，而反向的 `s2hk` 无法把“系”恢复成“係”。以上是用 OpenCC 词典的 Python 移植版实测的结果 ([OpenCC](https://github.com/BYVoid/OpenCC)；[opencc-python-reimplemented](https://pypi.org/project/opencc-python-reimplemented/))。jyutping.org 还列出一批按普通话同音合并、粤语却不同音的字，例如表 biu2/錶 biu1、只 zi2/隻 zek3、松 cung4/鬆 sung1、系·係 hai6 与喺 hai2。它的主张是：简体粤文只作初学者过渡，因为传统字更容易看出粤音规律 ([jyutping.org 簡化字](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/simplified.mdx))。结论是：**内容以繁体为唯一权威文本**，采用 rime-cantonese 使用的 OpenCC 传统字形，它可以自动转换到港、台、简体字形 ([rime-cantonese 字形原则](https://github.com/rime/rime-cantonese/wiki/%E6%9C%AC%E6%96%B9%E6%A1%88%E5%AD%97%E5%BD%A2%E9%81%B8%E7%94%A8%E5%8E%9F%E5%89%87))。简体只是构建时生成的显示视图，另配一张覆盖表：嗰、餸、喎保持原形，“係”保留原形，或转成“系”时附注 hai6。用户读简体，所以默认显示“简体字形 + 粤字原样保留”，遇到合并字时提示繁体和读音。课程中段再逐步切到香港繁体，最终目标是能读 LIHKG 原文。

读音标准同样没有官方版本。事实上的参考是 LSHK《電腦用漢字粵語拼音表》（**2.9 万多字，CC BY 4.0**，在 GitHub 维护）、CUHK《粵語審音配詞字庫》和 words.hk ([lshk-org/jyutping-table](https://github.com/lshk-org/jyutping-table))。何文汇从 1980 年代起推动依《广韵》反切的“正音”，学界由此分成“从切”和“从众”两派 ([HKU Scholars Hub](https://hub.hku.hk/handle/10722/228896))。Diu 宜采用“从众”的通行音。示范音区分 n/l、ng-/零声母、gw/g，这也是 rime-cantonese-upstream 的收录原则 ([rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream))。正音派读法只作注释。香港口语里 n 读 l 极为普遍，但梁源（2018）的感知实验显示，多数母语者仍能分辨 n/l ([梁源 2018](https://www.cuhk.edu.hk/ics/clrc/crcl_97_1/liang.pdf))。所以听力材料要有意放入懒音样本，产出仍按规范要求。

罗马化只用 LSHK 粤拼（Jyutping）。它纯 ASCII，一个符号对应一个音位；LSHK 字表、rime-cantonese、Cifu 都用它标音，主流粤语输入法也以它为主。耶鲁拼音用调号加 h 标阳调，广州话拼音方案用 ê、é、ü 等附加符号，两者都会让打字和判分变复杂 ([jyutping.org《點解要用粵拼》](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/scheme.mdx))。汉语拼音使用者的陷阱集中在“同形不同音”上。粤拼 j 相当于拼音 y（一 jat1），yu 才是 ü（豬 zyu1），gw/kw 是圆唇声母（廣 gwong2）。拼音的 ong 对应粤拼 ung，粤拼的 ong 是普通话没有的 [ɔŋ]（莊 zong1）。aa/a 是长短对立（三 saam1/心 sam1），eo/oe 是两个圆唇前元音。此外还有 -m/-p/-t/-k 韵尾和 6 个调号 ([jyutping.org《一文掌握粵拼》](https://github.com/CanCLID/jyutping.org/blob/master/src/content/learn/yue/index.mdx))。App 内部只存粤拼，耶鲁和 IPA 只做转换视图。

显示上，汉字上方用 ruby 注粤拼，数字调号紧跟音节，可切换为“只显声调”或“隐藏拼音”。按调着色时必须同时保留数字，以照顾色盲。变调词标实际读音，同时显示本调（如“女”neoi5→neoi2）。jyutping.org 的教学页就是 ruby 注音配真人录音，并用“詩史試時市事”做六调对比 ([jyutping.org learn](https://github.com/CanCLID/jyutping.org/blob/master/src/content/learn/yue/index.mdx))。答题输入分三种。一是输入粤拼：可省调号，y 可代 yu；判分时把“只错声调”单独统计，作为声调诊断。二是用系统粤拼键盘直接打汉字：iPhone 自带“粵語拼音”布局，Android 有 Gboard 粤拼 ([cantonese.hk 键盘指南](https://cantonese.hk/keyboard))。三是点选 App 内置的第一层粤字候选条。rime-cantonese 的“普通话反查”对应一条很常见的查询路径（“我认识这个字，但不知道粤音”），值得照做 ([rime-cantonese](https://github.com/rime/rime-cantonese))。港穗粤语可以互通，差异主要在口音、个别读音和词汇（如巴士/公共汽車、英语借词多少）；开源词库本身就是“港穗合一”的 ([Wikipedia: Hong Kong Cantonese](https://en.wikipedia.org/wiki/Hong_Kong_Cantonese)；[rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream))。建议发音以规范音为准，词汇以港穗共同核心为主，书写以香港推荐用字为标准答案，词条上标注【港】【穗】。侧重哪一边取决于用户学粤语之后去哪里用，这一点用户也还没说明。

## CC BY 字表加 Apache 模型足够搭底座，words.hk 只宜查阅

Diu 是自用 App，但沿用 mamma-mia 的规矩，仓库是公开的 ([mamma-mia README](https://github.com/li1000v2/mamma-mia/blob/main/README.md))。所以许可证仍然要紧：凡是打包进 `Content/` 的生成表，都会随公开仓库再分发。下表按“能否放进公开仓库、将来是否仍可商用”排出首选栈。

| 用途 | 首选 | 许可 | 用法要点 |
|---|---|---|---|
| 单字读音 | LSHK《電腦用漢字粵語拼音表》，2.9 万多字 ([GitHub](https://github.com/lshk-org/jyutping-table)) | CC BY 4.0 | 只需署名，可直接裁剪成设备端“字→粤拼候选”表 |
| 词音、文白读、变调、读音常见度 | rime-cantonese-upstream 的 `char.csv`/`word.csv` ([GitHub](https://github.com/CanCLID/rime-cantonese-upstream)) | CC BY 4.0；rime-cantonese 的 `jyut6ping3.maps` 为 ODbL 1.0 ([GitHub](https://github.com/rime/rime-cantonese)) | 词条记实际读音，并标注本调/变调、文读/白读，可直接用作多音字前端 |
| 字转粤拼（G2P）、分词 | ToJyutping ([GitHub](https://github.com/CanCLID/ToJyutping))；PyCantonese v4.0.0（2026-03）([GitHub](https://github.com/jacksonllee/pycantonese)) | ToJyutping 的 LICENSE 文件写 BSD-2，awesome 列表却写 CC BY 4.0，两者冲突；PyCantonese 为 MIT | `get_jyutping_candidates` 返回排序后的候选读音，有多个候选时标“需人工确认” |
| 繁简转换 | OpenCC ([GitHub](https://github.com/BYVoid/OpenCC)) | Apache-2.0 | 不做普粤翻译；需加粤字覆盖表 |
| 粤文判别 | canto-filter ([GitHub](https://github.com/CanCLID/canto-filter)) | 笔记未记录 | 输入须为繁体并采用推荐用字，可用来给 LLM 生成的句子把关 |
| 英文释义 | CC-Canto，2015 年 beta 时约 2.2 万条 ([cantonese.org](https://cantonese.org/download.html)) | CC BY-SA 3.0 | 改编后再分发须同样共享；与自有数据分层存放 |
| 粤语释义与例句 | words.hk 粵典 ([授权页](https://words.hk/base/hoifong/)) | 非商业开放资料授权；各处标注互相冲突 | 用于人工查阅和交叉校对，不进仓库 |
| 口语语料 | HKCanCor，约 23 万词，1997–98 年录制 ([TFDS](https://tensorflow.org/datasets/community_catalog/huggingface/hkcancor?authuser=1)) | CC BY 4.0 | 做词频和真实口语例句的种子 |
| 中古音推导 | nk2028 的 tshet-uinh-js 与 `gwongzau.js`、`putonghua.js` ([GitHub](https://github.com/nk2028/tshet-uinh-examples)) | MIT（tshet-uinh-data 的许可未核实） | 只在构建期运行，App 只读预计算结果 |
| 服务器或管线 ASR | Qwen3-ASR 1.7B/0.6B 与 ForcedAligner ([GitHub](https://github.com/QwenLM/Qwen3-ASR)) | Apache-2.0 | 可用于音频回转写和音节对齐 |
| 端侧 ASR 备选 | sherpa-onnx + SenseVoice-Small ([GitHub](https://github.com/k2-fsa/sherpa-onnx)) | Apache-2.0；SenseVoice 权重遵循 FunASR 模型许可 ([MODEL_LICENSE](https://github.com/modelscope/FunASR/blob/main/MODEL_LICENSE)) | 须署名并保留模型名；该许可还有“禁止诋毁”“可单方修订”等非标准条款 |
| 基频提取 | CREPE ([GitHub](https://github.com/marl/crepe))；Parselmouth ([GitHub](https://github.com/YannickJadoul/Parselmouth)) | CREPE 为 MIT；Parselmouth 为 GPL-3.0+ | Parselmouth 只在 `tools/` 里用，不进 App |
| 训练用语音 | Common Voice yue v25.0，**307 小时（已验证 210.67 小时），1,179 名说话人** ([MDC](https://datacollective.mozillafoundation.org/datasets/cmn29rqn9016to107eniyak65)) | CC0，但平台条款禁止再托管、禁止识别说话人 | 自用 App 不需要训练模型，暂时用不上 |

词典层的关键是：rime-cantonese-upstream 把每个读音标成“预设、常用、罕見、棄用”，并区分本调与变调、文读与白读 ([rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream))。同类数据里很少有这种结构化标注，它正好用来判断“这个字在这里该读哪个音”，以及“TTS 读错了没有”。words.hk 是口语释义和例句质量最高的开放来源，但它的条款以“非商业开放资料授权 1.0”为主，未经特别批准不得商用。它的分析页又把某个数据集标成 public domain，另一个项目则说其数据转储是 all-rights-reserved ([words.hk 授权](https://words.hk/base/hoifong/)；[words.hk analysis](https://words.hk/faiman/analysis/)；[read-path-glosses](https://github.com/leecommamark/read-path-glosses))。面对这种冲突，最稳妥的做法是只把它当人工参考书，配合 Pleco 或开源的 Jyut Dictionary 使用 ([jyut-dict](https://github.com/aaronhktan/jyut-dict))。Diu 的 MVP 不需要自建词典。

语音识别方面，2026 年开源粤语 ASR 最强的是 **Qwen3-ASR**（2026-01-29 发布，Apache-2.0），它把“粤语（香港口音）”和“粤语（广东口音）”分开支持。在 Common Voice 粤语测试集上，1.7B 版的 WER 为 **7.57**，Whisper-large-v3 为 16.23，GPT-4o-Transcribe 为 11.36 ([Qwen3-ASR](https://github.com/QwenLM/Qwen3-ASR))。需要注意，各来源的数字不能直接横比：同一个 WenetSpeech-Yue 短句测试集上，Whisper-large-v3 在 Qwen 的表里是 32.26，在 WenetSpeech-Yue 自己的 README 里却是 12.95 ([WenetSpeech-Yue](https://raw.githubusercontent.com/ASLP-lab/WenetSpeech-Yue/main/README.md))。端侧更现实的是 sherpa-onnx 里现成的 `sense-voice-zh-en-ja-ko-yue` 模型。社区评测中，SenseVoiceSmall 在“广州日常”测试集上的 CER 为 5.64%，粤语微调版 Whisper-large-v2 为 12.38% ([cantonese_asr_eval](https://github.com/AlienKevin/cantonese_asr_eval))。原版 Whisper 对粤语常常直接输出书面中文 ([HKUST Library](https://library.hkust.edu.hk/sc/?p=3750))，不适合用来检查“有没有说出嘅、咗、喺”。还有一个值得留意的模型：hon9kon9ize 的 wav2vec2bert-jyutping 直接输出带声调的粤拼，但没有公布准确率 ([GitHub](https://github.com/hon9kon9ize/wav2vec2bert-jyutping))。

语音合成方面，商用条款最清楚的是云服务。Google Cloud TTS 从 **2025-12-30 起以 Preview 状态支持 Chirp 3: HD 的 yue-HK** ([Cloud TTS release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes))，另有 yue-HK-Standard-A 至 D 四个标准声音 ([Cloud TTS voices](https://cloud.google.com/text-to-speech/docs/voices))。Chirp 3: HD 每月前 100 万字符免费，超出部分每百万字符 30 美元 ([Cloud TTS pricing](https://cloud.google.com/text-to-speech/pricing?hl=it))。Azure 有 zh-HK 的 HiuMaan、HiuGaai、WanLung 三个音色 ([json2video](https://json2video.com/ai-voices/azure/voices/zh-hk-hiumaanneural/))，Amazon Polly 有神经声线 Hiujin ([AWS Polly](https://docs.aws.amazon.com/polly/latest/dg/available-voices.html))。开源 TTS 里，GPT-SoVITS（MIT）从 V2 起支持粤语；CosyVoice 的 README 声明“仅供学术用途”；Fish-Speech 有营收门槛；MMS-TTS 为非商业许可；Edge TTS 非官方接口有违反服务条款的风险 ([GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS)；[CosyVoice](https://github.com/FunAudioLLM/CosyVoice)；[Fish-Speech LICENSE](https://github.com/fishaudio/fish-speech/blob/main/LICENSE)；[Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/2088770/are-opensource-edge-tts-free-for-commercial-use))。最关键的一点：mamma-mia 现用的 `gemini-3.8-flash-tts` 是否支持粤语，**没有证实**。检索摘要里，同系列 Flash-Lite TTS 的语言表有 Cantonese，Flash 的语言表被截断了；Cloud 版 Gemini-TTS 的列表只见 cmn ([Gemini 3.8 Flash-Lite TTS](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-lite-tts)；[Gemini-TTS](https://docs.cloud.google.com/text-to-speech/docs/gemini-tts))。

发音评估没有现成的粤语方案可买。Azure 发音评估的语言列表据称包含 zh-HK，但音素级输出只有 en-US 和 zh-CN，韵律评估只有 en-US ([Azure language learning overview](https://docs.azure.cn/en-us/ai-services/speech-service/language-learning-overview))。检索也没找到任何关于“学习者说粤语的声调错误检测”的论文 ([CUHK ICALIP 2008](https://www1.se.cuhk.edu.hk/~hccl/publications/pub/ICALIP2008_v1.pdf))。所以声调反馈只能自建：强制对齐（Qwen3-ForcedAligner，或基于 Common Voice zh-HK 训练的社区 MFA 模型 ([HKCantonese_models](https://github.com/chenchenzi/HKCantonese_models))）加基频提取。大模型方面，GPT-4o 和 Qwen-2.5-72B 在粤语 ARC-C 上已有 92–94% 的正确率 ([Yue-Benchmark](https://arxiv.org/html/2408.16756v3))；但在 HKCanto-Eval 里，GPT-4 的粤语正字法只得 **65.4%**，多数模型连同音、押韵判断都做不好 ([HKCanto-Eval](https://arxiv.org/html/2503.12440))。这决定了 Diu 的分工：LLM 负责写句子和讲解，粤拼由词典 G2P 生成；LLM 最多在“字典候选里哪个对”这种判断题上帮忙。

## 普通话使用者的中级粤语课仍是空白，多邻国只到早期 A2

粤语学习市场分成互不交叉的两半。一半是英文界面的国际产品，默认用户不识汉字，从零教起；另一半是中文界面、面向普通话使用者的产品，以多邻国为首。多邻国的粤语课只对“中文（简体）使用者”开放，用繁体字教学 ([Duolingo blog](https://blog.duolingo.com/chinese-languages))。按非官方追踪站的记录，它在 **2026-07-02 才扩到 48 个单元，达到“早期 CEFR A2”** ([duolingodata](https://duolingodata.com/dailynews.html))。有学习者学了 30 天去广州实测，结论是“听得懂司机和服务员，却说不出” ([澎湃](https://www.thepaper.cn/newsDetail_forward_30563565))。据报道，多邻国没有给方言课设商业收入目标，这一点只看到搜索摘要 ([Futu 新闻](https://news.futunn.com/en/post/1001613246/duolingo-launches-shanghainese-course-after-cantonese-no-commercial-revenue-targets))。

国产中文界面应用的内容方法高度同质：粤语歌逐字注音，港剧和 TVB 台词，旅行短语，“普通话转粤语”翻译朗读，社区里还大量流行谐音法。粤语U学院在应用宝有 167 万下载，差评集中在“收费高、内容少”和会员权益问题 ([应用宝](https://sj.qq.com/appdetail/com.huahua.yueyu))。粤语学习通年费 ¥98，主打 5000 首歌的注音 ([App Store](https://apps.apple.com/us/app/%E7%B2%A4%E8%AF%AD%E5%AD%A6%E4%B9%A0%E9%80%9A-%E8%87%AA%E5%AD%A6%E7%B2%A4%E8%AF%AD%E6%AD%8C%E6%9B%B2%E5%8F%91%E9%9F%B3%E5%AD%97%E5%85%B8/id1442054681))。粤拼歌词收录 8000 多首 ([jyut6.com](https://www.jyut6.com/))。这些产品很少教系统的语法和粤语白话文。国际产品的硬伤是“只有书面语”和“粤拼出错”。Clozemaster 2026 年的评测说，大多数“粤语课程”其实是书面标准中文配上粤语音频 ([Clozemaster](https://www.clozemaster.com/blog/best-apps-to-learn-cantonese/))。CantoneseClass101 的中高级课程偏书面语 ([perapera](https://www.perapera.org/cantoneseclass101-review/))，LingQ 的粤拼经常错或缺 ([LingQ 论坛](https://forum.lingq.com/t/cantonese-romanization-jyutping-is-often-inaccurate-or-missing/2227383))，Pimsleur 的粤语一直只有 30 课 ([mezzoguild](https://www.mezzoguild.com/online-cantonese-courses/))。2025–2026 年密集出现了一批 AI 工具，如 HelloTalk 的 Cantonese Ai、CantoAI（Pro 每月 10 美元）、cantonese.ai。它们几乎都是英文界面、以香港口语为准，没有一个利用“已会普通话”这个前提 ([CantoAI](https://canto-ai.com/)；[Cantonese Ai](https://apps.apple.com/us/app/cantonese-ai-your-ai-tutor/id6748127421))。

最清楚的空白是读音迁移。检索没有找到任何一款以“从普通话读音推导粤语读音”为核心教学法的 App，也没有任何产品利用赣语或客家话 ([知乎《借助普通话推断粤语读音》](https://zhuanlan.zhihu.com/p/702923289))。相关的东西都停留在别处：知乎和 CSDN 上的规律总结；Michelle Fullwood 带计数的交互对应表，她提出的问题是“总按最可能的对应去猜，能猜对多少” ([michellefullwood.com](https://michellefullwood.com/languages/cantonese/correspondences/))；还有 2026 年 8 月一篇提出普粤成对罗马化并开源工具链的论文 ([arXiv 2608.29170](https://arxiv.org/abs/2608.29170))。也没有产品公开过声调反馈的准确率，Preply 甚至提醒，靠自动识别学粤语声调“不是最可靠的方式” ([Preply](https://preply.com/en/blog/best-apps-to-learn-cantonese/))。没有一款应用让用户在香港和广州变体之间选择。对自用的 Diu 来说，这张竞品图不是用来找商业机会的，而是用来划定该做什么、不该做什么。词典、歌词库、短语手册这些市场上免费就有，不必重做。值得投入的是没人提供的四样东西：一是带命中率的读音推导；二是同一句话“书面语 vs 口语粤语”的双轨对照；三是原速真人听力；四是诚实的声调反馈，只画曲线、标明不确定，不给假分数。

## 平调高度、入声和 -m 尾是主战场，赣语能补普通话推不出的一格

### 普通话母语者错得最多的是 T3/T6 与 T5

粤语有 9 个调类、6 个调值。按 jyutping.org 的对照：阴平 55/53，阴上 35，阴去 33，阳平 21，阳上 13，阳去 22；入声 5/3/2 是 1/3/6 声的短促版，粤拼也记作 1/3/6 ([jyutping.org 对照表](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/table.mdx))。阴入按元音长短再分：短元音读高的 1，长元音读中的 3 ([粤语音系](https://zh.wikipedia.org/zh-hans/%E7%B2%B5%E8%AA%9E%E9%9F%B3%E7%B3%BB))。入声韵尾只成阻、不爆破 ([入声](https://baike.baidu.com/item/%E5%85%A5%E5%A3%B0/5495064))。aa/a 的长短对立在所有韵尾前都存在（街 gaai1/鸡 gai1）([知乎 十一元音](https://zhuanlan.zhihu.com/p/265020710))。对普通话者真正陌生的有三样：一是“同一调形、不同高度”的平调对立（1/3/6）和低升调 5、低降调 4；二是塞音韵尾；三是元音长短。

实证研究把难点排得很清楚。《Applied Sciences》2025 年的研究比较了母语者与高级普通话背景学习者：学习者 T1 感知最准，T5 最差；**主要混淆是 T3↔T6**，其次是 T5 与 T3/T6、T3 与 T4，比母语者高出至少 15 个百分点 ([Applied Sciences 2025](https://doi.org/10.3390/app15126590))。PACLIC 2024 让 30 名普通话者接受两天高变异性语音训练（HVPT），训练后识别显著提升，而且学习者更偏好显式视觉提示（调值字母）([PACLIC 2024](https://aclanthology.org/2024.paclic-1.121.pdf))。另一项香港研究里，32 名普通话者训练粤语平调后，识别提升了，区分却没有提升；音乐能力和音高能力能预测学习效果 ([ResearchGate](https://www.researchgate.net/publication/348671150_The_effect_of_Mandarin_listeners'_musical_and_pitch_aptitude_on_perceptual_learning_of_Cantonese_level-tones))。高变异性材料的优势只体现在低平调和中平调的产出上 ([PMC6206236](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6206236/))。也有反面线索：一项较早的研究称，普通话者区分粤语声调并不比英语者好，但只见到摘要 ([Alfieri 2014](https://as.nyu.edu/content/dam/nyu-as/linguistics/documents/Alfieri2014.pdf))。这些证据合起来，给出了 Diu 的训练顺序：先练平调高度（3/6），再练低升 5 和它与 2、3、6 的区分，再练把低降 4 压低压平，然后是入声收尾和中入的高度、aa/a 长短、-m 尾、ng- 起首、oe/eo、gw/kw。练法用多说话人、即时反馈、带调值或基频曲线的 HVPT，而且必须练产出，因为感知收益不一定迁移到区分和发音。影子跟读的系统综述（44 项研究）支持它改善可懂度、流利度和韵律，但对单个音段的效果不确定 ([Taylor & Francis 2025](https://www.tandfonline.com/doi/full/10.1080/29984475.2025.2546827))。所以跟读负责韵律，最小对立负责音段。

### 普通话读音能推出大半声调，去声不送气字是盲区

普粤对应的根据是中古音的“平上去入 × 清浊”。网上流传的“超过 90% 对应”是民间说法，没有交代样本，编号体系也混乱 ([博客园](https://www.cnblogs.com/xryyforver/archive/2008/01/29/1508862.html))。有据可查的单项统计是：粤语第 2 调的字有 **89%** 在普通话读第 3 调 ([ResearchGate](https://www.researchgate.net/figure/Major-tone-correspondences-between-Cantonese-and-Mandarin-words_tbl2_262364609))。全浊上声在普通话变去声，次浊上声留在上声 ([浊上变去](https://zh.wikipedia.org/zh-hans/%E6%BF%81%E4%B8%8A%E8%AE%8A%E5%8E%BB))；粤语古浊上则分化为白读阳上（5）和文读阳去（6）([维基教科书 粤语/语音](https://zh.wikibooks.org/zh-hans/%E7%B2%B5%E8%AA%9E/%E8%AA%9E%E9%9F%B3))。据此可以写出一张“看普通话调类和声母，预测粤语调”的表。表中规则依据音韵演变常识推导，例字为典型字，上线前须用字表批量核验并算出命中率。

| 普通话调类 | 条件（看普通话声母） | 预测粤语调（粤拼） | 例字 | 风险 |
|---|---|---|---|---|
| 阴平 | 任意 | 1 | 天 tin1、风 fung1 | 清入字也派入阴平：一 jat1、八 baat3、黑 hak1 |
| 阳平 | 送气声母，或 m/n/l/r/零声母 | 4 | 人 jan4、茶 caa4、门 mun4 | 送气的阳平几乎不会是入声 |
| 阳平 | 不送气 b/d/g/j/zh/z | 多为入声：全浊入读 6，清入读 1 或 3 | 白 baak6、读 duk6；国 gwok3、急 gap1 | 分不清全浊入还是清入 |
| 上声 | 清声母 | 2 | 好 hou2、手 sau2 | 入声来源：百 baak3、北 bak1 |
| 上声 | m/n/l/r | 5 | 马 maa5、你 nei5、老 lou5 | 个别读 2 |
| 上声 | 零声母 | 5 或 2 | 有 jau5、五 ng5；椅 ji2、碗 wun2 | 来源不同，需单记 |
| 去声 | 送气 | 3 | 去 heoi3、菜 coi3 | 少数全浊字普通话例外读送气 |
| 去声 | m/n/l/r | 6 | 面 min6、路 lou6；入声 六 luk6、月 jyut6 | 入声只需加韵尾 |
| 去声 | 不送气或擦音 | **3 或 6，普通话无法区分** | 布 bou3/步 bou6、到 dou3/道 dou6、世 sai3/事 si6 | 正是赣语能补的格子 |

声母和韵母也有成串的规则。普通话 j/q/x 有两个来源：见组读粤语 g/k/h，精组读 z/c/s。普通话韵母为 ia、iong 的字一定是团音 ([知乎 声母篇](https://zhuanlan.zhihu.com/p/702923289)；[BYVoid 尖团](https://byvoid.com/zhs/blog/jqx-ziantuan/))。其余几条（推导规则，例字待核验）：普通话 k 开口多读粤语 h（开 hoi1、看 hon3），合口多读 f（苦 fu2、快 faai3）；普通话 h 加合口时，晓母读 f（花 faa1），匣母读 w（华 waa4）；**普通话 w 开头的微母字读 m**（文 man4、万 maan6、网 mong5），这一条高频而且规则；日母 r 读 j（人 jan4）；zh/ch/sh 与 z/c/s 在粤语合并，属于正迁移的简化。韵母方面也有规律：普通话 -ao 按中古一、二、三等分化为 ou/aau/iu（好 hou2、包 baau1、小 siu2），-i 按止摄、蟹摄分为 ei/ai（机 gei1、鸡 gai1）([CSDN 韵母对应](https://blog.csdn.net/m0_50272029/article/details/147383878))。普通话完全推不出的是咸深摄的 -m/-p（三 saam1、心 sam1、十 sap6），只能靠声旁（今、金、甘、占、音、林、咸等）或保留唇音尾的赣语点。实现上，可以在构建期用 Unihan 的 kMandarin/kCantonese 和 LSHK 字表，对常用 3500 字逐条算出各规则的精确率和覆盖率，再用 tshet-uinh-js 的 `gwongzau.js` 给每个字算出“按规律应读的粤音”，与字典实际读音比对，标成“规律、例外、文白异读” ([tshet-uinh-examples](https://github.com/nk2028/tshet-uinh-examples))。App 只读预计算结果，并把规则命中率直接展示给学习者。

### 赣语加成要看方言点，以下结论都以用户的方言点为前提

用户还没说明自己说哪个地方的赣语，下面的结论按片区条件化给出。有两点要先说明：调研没能读到熊正辉《南昌方言词典》引论、《汉语方言字汇》等一手材料，各来源给出的南昌调值也互有分歧；检索也**没有找到任何“赣语者学粤语”的实证研究**。下面所有“优势”都是对比分析的推断，需要用户实测。

| 片区（代表点） | 入声 | 唇音韵尾 -m/-p | 对学粤语的含义 |
|---|---|---|---|
| 昌都片（南昌、新建、永修、修水等） | 阴入、阳入两调，阴高阳低；南昌约为 5 和 2；韵尾 -t、-k，-k 有弱化为 -ʔ 的趋势 ([赣语](https://zh.wikipedia.org/zh-hans/%E8%B4%9B%E8%AA%9E)；[南昌话](https://zh.wikipedia.org/zh-cn/%E5%8D%97%E6%98%8C%E8%AF%9D)) | 南昌已并入 -n/-t；同片的高安、靖安、奉新、安义保留 ([赣语](https://zh.wikipedia.org/zh-cn/%E8%B4%9B%E8%AA%9E)) | 能直接认出入声字；阴入（调值约 5）接近粤语短促的 1 声，阳入（约 2）接近粤语短促的 6 声；粤语中入（粤拼记 3，调值 3）是新调；南昌人要新建 -m/-p |
| 抚广片（抚州、临川、黎川、南丰、东乡、资溪、广昌等） | 笔记未查到调数 | 东乡、资溪、抚州、黎川、南丰、广昌保留 -m/-p ([赣语方言](https://zh.wikipedia.org/zh-hans/%E8%B4%9B%E8%AA%9E%E6%96%B9%E8%A8%80))；孙辈可能已不保留 ([抚州话](https://baike.baidu.com/item/%E6%8A%9A%E5%B7%9E%E8%AF%9D/9559267)) | 咸深摄字有望直接迁移，这是普通话者没有的优势 |
| 宜浏片（宜春等） | 宜春只有一个入声调；丰城、万载分阴阳；分宜、萍乡没有入声 ([宜春话](https://zh.wikipedia.org/zh-hans/%E5%AE%9C%E6%98%A5%E8%AF%9D)) | 未查到 | 能感知入声但分不出阴阳；分宜、萍乡的学习者与普通话者处境相同 |
| 鹰弋片（鹰潭、贵溪、鄱阳、余干等） | 两个入声，一般阴高阳低；余干的阴阳入靠后接鼻音的高低区分（只见百科）([鹰弋片](https://zh.wikipedia.org/zh-hans/%E9%B7%B9%E5%BC%8B%E7%89%87)) | 未查到 | 大体同昌都片 |
| 多数赣语共有 | 古全浊塞音、塞擦音不论平仄都读送气 ([江西方言](https://baike.baidu.com/item/%E6%B1%9F%E8%A5%BF%E6%96%B9%E8%A8%80/8728691)) | — | 可用于“双方言判别”：判断仄声字读粤语 6 还是 3 |

最稳健、也最有价值的一条是**全浊送气**，因为它不依赖片区内的细分。规则是：普通话上声、去声或入声字，若普通话不送气（b/d/g/j/zh/z）而赣语送气，就是古全浊字，粤语多读不送气 + 6 声（入声也读 6），古上声字还可能白读为送气 + 5 声；若两边都不送气，就是全清字，粤语读 3、2，入声读 1 或 3。举例说，步/布、道/到、在/再、旧/救、白/百这几对，按规则应是前字南昌话送气、后字不送气，对应粤语 bou6/bou3、dou6/dou3、zoi6/zoi3、gau6/gau3、baak6/baak3。这些南昌读音是按规则推出来的，需要用户用自己的方言核对。这一条正好补上上表普通话推不出的那一格。它的边界在擦音：父/富、事/世、汗/汉没有送气对立，只能看阴去和阳去。而南昌、新建、安义、永修、修水、德安等地有“送气分调”，调类分合还跟今声母送气与否有关，铜/农、再/菜的调都不同 ([南昌话](https://zh.wikipedia.org/zh-cn/%E5%8D%97%E6%98%8C%E8%AF%9D)；[颜森 江西方言分区](https://www.voicedic.com/?p=544))。所以在送气声母上，“南昌读阳去，粤语就读 6”可能失效，各来源的说法也不一致。

入声和声调形状的影响因片区而异。按引用的调值，南昌的阴入 5 和阳入 2 几乎等于粤语短促的 1 和 6，学习者可以直接借用。粤语的中入 3 是新调类：百、八、甲、脚、铁、雪这些长元音清入字，南昌人容易按本方言的高调阴入去读，需要专门训练。负迁移主要在舒声：南昌阳平记作 24、阴去记作 45 或 445，都是升调 ([知乎 南昌方言声韵调](https://zhuanlan.zhihu.com/p/30387163))，而粤语阳平 4 是低降、阴去 3 是中平，学习者容易把粤语 4 声往上挑，把 3 声读得往上走。南昌一带 n/l 相混 ([江南都市报 2024](https://jndsb.jxnews.com.cn/system/2024/09/28/020649964.shtml))，但香港口语本来就普遍 n→l，这一项可以只要求听辨、不强求产出。还有一条线索：有语言学者认为赣语与粤语之间有大量相同或相似的词 ([江西新闻网](https://jiangxi.jxnews.com.cn/system/2024/09/28/020649965.shtml))，但没有具体词表，暂不进入设计。产品上，既然方言点未知，最合理的做法是首次启动时做一个方言自测，用约 10 个字判断该开哪些模块，再把用户自己的方言读音存进本机，作为规则卡上的“赣语对照”行。这些字可以是：十/入/三/心（有无 -m/-p）、白/百/六（入声有几个调）、步/布/道/到（全浊是否送气）、北/色（入声收 -t 还是 -k）、我/牙（有无 ng-）、南/蓝（n/l）。

### 口语差异词、句末助词与课程顺序

普通话者读懂粤语书面语几乎没有障碍，门槛在两类词。一类是高频口语核心词，如佢、睇、食、行、企、嘢、乜嘢、边度、几时、唔、冇、係、喺、畀。另一类是同形异义词，如“走”在粤语是跑或离开，“行”才是走 ([words.hk 走](https://words.hk/zidin/%E8%B5%B0))。现成的学习者词表质量很差，比如 GoCantonese 的“高频词”排在前面的是“的、是、於” ([GoCantonese](https://gocantonese.com/popular-words))。Diu 应以 Cifu 和 HKCanCor 词频，加上 canto-filter 的“粤语独有词”正则为种子，自建 300–500 词的差异词表。语法上，粤普同为主谓宾分析语。差异集中在语序（佢高过你、畀本書我、行先）、体标记（咗、緊、過、住、晒、埋、返）、否定三分（唔、冇、未）和句末助词。句末助词按计数方法不同，从约 30 个到 95 个不等 ([Leung](https://linguistics.berkeley.edu/~herman/documents/CantoneseFPs_SyntaxCircle_Leung.pdf))。Sybesma 与 Li（2007）把约 40 个常用助词拆成 12 个最小语义成分 ([Lingua 2007](https://www.sciencedirect.com/science/article/abs/pii/S0024384106002130))。据此，助词可以按 l 系（啦、喇、囉）、z 系（啫、咋）、m 系（咩、嘛）、g 系（㗎、嘅）分组教，每个助词配几句“只换助词”的对比句。香港口语还普遍夹英语：一项 WhatsApp 研究中，**73.1%** 的句内转换以粤语为主体语言 ([JEAL](https://animorepository.dlsu.edu.ph/cgi/viewcontent.cgi?article=1041&context=jeal))。所以听力里要有 的士、OT、check、book 这类词。

课程顺序应与给英语者的课程截然不同。港中大雅礼中国语文研习所为普通话学生单设粤语课程轨道，并开设专门的“Sounds of Cantonese”语音课，这佐证了“按背景分轨、语音单列”的做法 ([YCCLA](https://yccla.cuhk.edu.hk/upsoverview)；[CUHK Handbook](https://rgsntl.rgs.cuhk.edu.hk/aqs_prd_applx/public/handbook/view_document.aspx?id=1588&seq=1&lang=en))。对这位学习者，起步约 12 周的建议安排是：第 1–2 周集中练音系和每天 5 分钟的 HVPT；第 2–4 周启用读音推导，先讲高精度规则，再讲普赣双方言判别；第 3–6 周学口语差异词和同形异义词；第 4–8 周学最小语法集和第一批助词；第 6–12 周转向泛听和句子挖掘，同时练懒音感知和粤英夹杂。汉字书写和大部分书面词义可以整块跳过。要警惕的是“书面腔”，即把普通话字直接读成粤音（说“他”而不说“佢”）。粤语歌可以辅助声调感知：CantoJam 用 37,877 个演唱音节检验了“0243”音高阶梯，结论是大体成立 ([CantoJam](https://github.com/candpixie/CantoJam)；[Music Theory Online 2021](https://mtosmt.org/issues/mto.21.27.1/mto.21.27.1.li.html))。但歌词多为书面语，不能当口语来源。港剧字幕多为书面语，正好拿来做“听口语、对照书面字幕”的练习，同时要防止学习者看字幕代替听。

## mamma-mia 的骨架大多可照搬，要重写的是分词、切句和音频校验

### 照搬、改造与新写的边界

建议把代码复制进 Diu 并改名为 DiuCore，不与 mamma-mia 共用一个包。理由有三：两边都是自用，改动节奏不同，而且 mamma-mia 在 12 月意大利行程后基本会冻结。DiuCore 继续遵守“只依赖 Foundation 和 swift-fsrs、不 import UIKit/SwiftUI/SwiftData”的边界，在 `swift:6.3-noble` 容器里跑测试 ([mamma-mia CLAUDE.md](https://github.com/li1000v2/mamma-mia/blob/main/CLAUDE.md))。swift-fsrs 仍固定在 main 的一个提交上，因为 FSRS-6 只在 main，最新 tag v5.0.0 还是 FSRS-5 ([Package.swift](https://github.com/li1000v2/mamma-mia/blob/main/Packages/MammaCore/Package.swift))。

| mamma-mia 部分 | Diu 处理 | 原因 |
|---|---|---|
| `project.yml`、4 个 workflow、`docs/sideload.md` | 改名照搬：Diu，`io.github.li1000v2.diu` | 与语言无关；新的 App ID 占用每周 10 个名额中的 1 个 |
| `Scheduling/`（FSRS-6，listen/recall 分开调度、同一句的卡错开出）、`Time/`、`Dialogue/`、`Study/` | 照搬 | 与语言无关 |
| `RolePlay/` 的 GeminiRequest、GeminiStream、GeminiFailure、TurnAllowance，以及 App 层的 KeychainStore、StreamingSpeechPlayer | 照搬 | 端点、SSE、错误分类、每日上限、钥匙串都通用 |
| `Progress/StudyProgress` | 改造 | 去掉或可选化 trip/spotLogs，加显示偏好；继续“只做加法” |
| `Content/`、`Audio/AudioManifest`、各 schema | 改造 | `it` 换成 `yue` + `jyutping` 等；音频 key 加粤拼 |
| `Speech/SpeechCheck`、`SpokenSentence` | 重写分词与归一化，保留对齐骨架 | 汉字的 `isLetter` 为真，整句汉字会成为一个“词” |
| `RolePlay/SentenceChunker`、`RolePlayPrompt`、`RolePlayFeedback` | 重写 | “第一个 CJK 字符就是翻译开始”在粤语里不成立；`isCJK` 只覆盖到 U+FFEF，漏掉扩展 B 以后的区段 |
| `Travel/` | 视学习目标而定 | 有港澳或广州行程才保留；`DepartureCheck` 里的签名到期解析拆成独立的 `Signing/` |
| `Cantonese/`、`Tone/` | 新写 | 粤拼解析、汉字切分（含扩展区）、繁简与俗写折叠、字→粤拼表、粤语数字；基频提取与归一化 |

表中两处“重写”都有代码依据。`SpokenSentence` 的分词条件是 `character.isLetter || character.isNumber` ([SpokenSentence.swift](https://github.com/li1000v2/mamma-mia/blob/main/Packages/MammaCore/Sources/MammaCore/Speech/SpokenSentence.swift))，于是“唔該一杯凍檸茶”会被当成一个词，`SpeechCheck` 就退化成整句全对或全错。`isCJK` 只认 `0x3000…0xFFEF` 以内的几段码位 ([SentenceChunker.swift](https://github.com/li1000v2/mamma-mia/blob/main/Packages/MammaCore/Sources/MammaCore/RolePlay/SentenceChunker.swift))，𨋢、𡃁 这类字会被漏判。

### 内容模型：繁体权威文本、逐字粤拼和双重审核锁

mamma-mia 现在的句子字段是 `{id, it, zh, frame?, scene, status, note?, checkedText?}`；状态不是 draft 时，`checkedText` 必须等于原文，这样改过的句子不能沿用旧的审核结论 ([lesson.schema.json](https://github.com/li1000v2/mamma-mia/blob/main/Content/schema/lesson.schema.json))。粤语版要把锁扩大到读音，因为读音错和字错一样需要重审。各字段如下：`yue` 是繁体书面粤语原文，口语字照写，是音频 key、交叉校对和识别比对的唯一依据；`jyutping` 逐音节带调号，音节数必须等于汉字数；`zh` 是普通话意思（简体）；可选的 `formal` 给出书面语对照，为普通话者提供双轨；`ttsText` 只发给 TTS，用同音字逼出正确读音；`accept` 列出可接受的其他说法；`checkedText` 等于 `yue` 与 `jyutping` 以竖线相连。示例如下（内容只说明字段，正式内容要走校对）：

```json
{"id": "L05-01", "yue": "唔該，一杯凍檸茶。", "jyutping": "m4 goi1 jat1 bui1 dung3 ning4 caa4",
 "zh": "劳驾，来一杯冻柠檬茶。", "formal": "請給我一杯冰檸檬茶。", "status": "draft",
 "checkedText": "唔該，一杯凍檸茶。|m4 goi1 jat1 bui1 dung3 ning4 caa4"}
```

`validate_content.py` 和 DiuCore 的 `ContentValidator` 两层都要检查以下几项：每个音节都在该字的字典候选读音里，不在时报 warning，标 checked 前必须人工确认；多音字和最小对立字自动打上 `polyphone`、`tonePair` 标签；扩展区字列表；生成的简体文件是否最新。音频 manifest 的 key 从“声线|速度|原文”改为同时包含 `yue` 与 `jyutping`，这样改了读音，文件名就会变，音频会自动重生成。`AudioManifest.key`、`tools/` 里的 `audio_key()` 和 manifest schema 的正则三处要同步修改，这只是延续 mamma-mia 已有的一致性规则。

### 音频管线：先拦“念成普通话”，再谨慎相信声调判断

mamma-mia 的音频管线建立在“承认模型哪里不可靠”之上：Gemini TTS 生成，另一个模型回转写比对，高思考档模型逐段粗筛，结果和音频哈希绑定缓存，可疑录音最多重生成 4 版、复筛两次都干净才替换，模型判不准的类别（双辅音）一律交给人听 ([mamma-mia CLAUDE.md](https://github.com/li1000v2/mamma-mia/blob/main/CLAUDE.md))。实测成本很低：第一周 208 段的粗筛花费约 **0.83 美元** ([audio-screen-week1.md](https://github.com/li1000v2/mamma-mia/blob/main/reviews/audio-screen-week1.md))。一轮重生成处理了 127 条录音，换掉 82 条，42 条判为误报保留，最后没有仍可疑的 ([audio-regen.md](https://github.com/li1000v2/mamma-mia/blob/main/reviews/audio-regen.md))。整套形状可以直接复用，粤语版需要做四处改动。

第一，防止“念成普通话”。mamma-mia 只发原文、不带指令，因为实测指令会被念出来，语言完全靠模型自动判断 ([gen_audio.py](https://github.com/li1000v2/mamma-mia/blob/main/tools/gen_audio.py))。繁体汉字很可能被当成普通话念，“你好”“多謝”这类短句风险最大。对策是让回转写模型输出带语言标签的 JSON（`language` 为 yue、cmn 或 other），凡是不是 yue 就判失败重试。同时在 `tts_voices.json` 里给每个声线加一个 `engine` 字段，粤语念不好的条目退到 Chirp 3: HD 的 yue-HK。代价是在线对话的声音和示范音频不再是同一个声线。成本不构成约束：2,000 句 × 12 字 × 2 声线约 4.8 万字符，远在 Chirp 3: HD 每月 100 万字符的免费额度内 ([Cloud TTS pricing](https://cloud.google.com/text-to-speech/pricing?hl=it))。

第二，比对改成按音节。归一化流程是：NFKC，去标点，繁简折叠，俗写折叠（左→咗、系→係、D→啲、既→嘅，只用于比对），粤语数字归一（兩/二、廿/二十、三十五蚊）。然后用字典把两边都转成粤拼，逐音节比较；同音不同字也算对。局限要写清楚：转写模型自带的语言模型会把轻微的声调错误“脑补”成预期的字，所以这一层主要拦得住念成普通话、漏读、多读和明显读错，拦不住细微的声调错误。

第三，AI 粗筛增加 `mandarin_reading`、`tone`、`polyphone`、`final_stop`、`lazy_sound` 五类问题，并且**先校准、再信任**。mamma-mia 实测发现，粗筛模型判断双辅音时“会自信地判错”，于是用正则把这类意见过滤掉 ([audio_review.py](https://github.com/li1000v2/mamma-mia/blob/main/tools/audio_review.py))。粤语声调很可能是同一类风险。所以要先用一组读音已知的片段测出粗筛模型判声调的准确率，这组片段包括单音节和双音节的最小对立，以及用 `ttsText` 故意制造的错调版本；准确率不够，声调意见就只当提示。多音字（行 hang4/hong4、重 cung5/zung6/cung4、樂 lok6/ngok6）、最小对立组和单字条目，必须有人听过才能标 audio_ok。学习者早期分辨不了声调，所以母语者抽检（`native`）在粤语版里的分量要比意大利语版重得多。单音节示范最好用真人录音，自用规模大约是 300–500 条核心句，加上各声调对立音节。

第四，继承并收紧调用纪律。2026-10-16，mamma-mia 一天发出数千次请求、最高 40 路并发，之后 Google Cloud 项目被 Trust & Safety 以违反服务条款为由停用。此后所有工具的并发都压在 `MAX_PARALLEL = 8` 以内，遇到 401/403/404 立即停止，并且不许换项目或换账号绕过 ([mamma-mia CLAUDE.md](https://github.com/li1000v2/mamma-mia/blob/main/CLAUDE.md))。Diu 还应加上每分钟请求数上限和单次运行的总请求预算。开工前也要先确认 Diu 用哪个 Google Cloud 项目：如果还是被停用的那个，要先申诉恢复。

### 识别与声调反馈：按字对齐，同音算对，只画不评

Apple 端侧识别对粤语的支持只有间接证据。一份 2026-10-21 的 macOS 27.0.1 报告显示，`SpeechTranscriber.supportedLocales` 里有 yue-CN 和 zh-HK，但 `installedLocales` 只有 zh-CN 和 zh-TW ([FluidVoice #1202](https://github.com/altic-dev/FluidVoice/issues/1202))。模拟器上 `supportedLocales` 返回空数组，必须真机测试 ([Apple Developer Forums 802969](https://developer.apple.com/forums/thread/802969))。旧的 SFSpeechRecognizer 列表里也有 yue-CN 和 zh-HK ([MBS plugins](https://mbsplugins.eu/SpeechRecognitionSupportedLocales.shtml))。所以 `SpeechService` 要把“找最接近 it_IT 的 locale”换成按顺序探测：先用 `supportedLocale(equivalentTo:)` 试 yue-CN，再试 zh-HK，结果写进诊断页并允许手动切换。降级链依次是：SpeechTranscriber → DictationTranscriber → SFSpeechRecognizer → 内置 sherpa-onnx SenseVoice → 只保留录音对比和自评。注意不能凭 locale 代码猜：zh-HK 未必就是粤语模型。

判定层改为按字对齐。每个汉字算一个 token，CJK 判定覆盖到扩展 B–H，拉丁字母串和数字串各算一个 token。经过繁简与俗写折叠后，字相同或粤拼相同都算匹配；声母、韵母相同、只有声调不同的，记成 `heardAs`，界面上显示为“『試 si3』听成了『市 si5』”这类声调提示。mamma-mia 的判定四档（clear、mostly ≥ 60%、unclear、silent）和“识别只是提示、只有自评改变卡片状态”的原则照搬 ([SpeechCheck.swift](https://github.com/li1000v2/mamma-mia/blob/main/Packages/MammaCore/Sources/MammaCore/Speech/SpeechCheck.swift))。挑战选项的内容约定要改：mamma-mia 要求选项之间“至少差一个听得出的词”，粤语版改为“至少差一个声母或韵母不同的音节”，只差声调的选项只能点选，因为识别器可能把它们“纠正”成同一个字。

“只判断对方能不能听懂、不打发音分”对粤语只成立一半。好的一面是，粤语里声调决定词义，识别器听错调往往就会听成别的字。不足的一面是，整句里的语言模型会凭上下文把错调纠正回来，所以 clear 不代表声调对。补救办法都不打分。一是单字或单词的“说出来”练习，此时上下文最弱，`heardAs` 最能暴露声调问题。二是基频曲线叠图：在 DiuCore 里用纯 Swift 写 YIN 基频提取，可在 Linux 上测试，Accelerate 加速版放在 App 层；先让用户读“詩史試時市事”做 10 秒校准，再把曲线换算成相对说话人平均音高的半音，映射到五度刻度上，与 `tools/` 预计算的参考曲线叠在一起显示。16 种声调归一化方法的比较中，“相对说话人平均音高的半音变换”效果最好 ([Zhang, PACLIC 2018](https://preview.aclanthology.org/moar-dois/Y18-1095.pdf))；对粤语单字调做 z-score 归一化，可以把说话人之间的方差降低约 20 倍 ([Rose, SST 2022](https://sst2022.com/declination-adjusted-normalisation-of-cantonese-citation-tones/))。平调的判断依赖相对音高，整体上移两个半音就可能把中平听成低平 ([PolyU](https://www.polyu.edu.hk/cbs/rclcn/images/publications/202312_ChenSi_AMLap_10099_BeiLi.pdf))，所以曲线一定要在说话人自己的音域里看。

卡片设计上，每句话拆成几张独立调度的卡：听辨、开口回忆、普→粤口语对译（他→佢、在→喺、没有→冇、看→睇）、字→粤拼、助词完形填空。swift-fsrs 用 FSRS-6 默认参数即可。srs-benchmark 在约 3.5 亿条复习上测得 FSRS-6 的 log loss 为 0.3460 ([srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark))，而复习量少于约 1000 次时，个性化参数的收益有限 ([imprimo](https://imprimo.app/blog/fsrs-vs-sm2-why-your-flashcard-app-uses-outdated-algorithm))。mamma-mia 的 `CardKind.minimalPair` 至今只有枚举声明、没有被用到 ([ReviewScheduler.swift](https://github.com/li1000v2/mamma-mia/blob/main/Packages/MammaCore/Sources/MammaCore/Scheduling/ReviewScheduler.swift))。我的建议是分两层用它。会造成误解的具体词对（如 買 maai5/賣 maai6）做成 `minimalPair` 卡进 FSRS；“3 声对 6 声”这类声调范畴辨别则做成每天 5 分钟、多说话人、按正确率自适应挑难对的 HVPT 练习，不进 FSRS，只记统计。原因是 HVPT 的证据来自大量多变异刺激加即时反馈的训练，而大量近似卡塞进 FSRS 会互相干扰，让调度失真。

### 角色扮演、字体与侧载名额

在线角色扮演的工程部分可以原样复用：一回合是录音 → 离线识别（同时播语气词）→ 文本模型流式回复（6 秒没出字就换备用模型）→ 按句切开 → TTS 流式合成、边收边播；key 存在钥匙串里且只限本机，每天默认 40 回合，对话文字不写日志，结束后的反馈卡把纠正的句子转成复习卡 ([mamma-mia CLAUDE.md](https://github.com/li1000v2/mamma-mia/blob/main/CLAUDE.md))。要重写的是语言部分。提示词改成“香港粤语口语，用港式繁体口语字书写，不用書面語、們、的、是、在、沒有”。长度上限从“每回合 15 个词”改为“约 25 个字”。输出改成带标签的“粵：/普：”两行，或者用 JSON，不再靠“遇到汉字就是翻译”来切分。粤拼不让 LLM 生成，在设备上查表现算，并标明“自动注音，可能有误”。反馈卡提示词里要写明学习者是普通话母语者，常见错误是书面语直译、量词、句末助词和双宾语语序。兜底语音显式请求 `AVSpeechSynthesisVoice(language: "zh-HK")`。但 iOS 18 起有个 bug：“朗读内容”里的中文口语设置会覆盖 App 的请求，开发者报告到 iOS 26 仍然存在 ([Apple Developer Forums 772348](https://developer.apple.com/forums/thread/772348))，所以首次使用时要提示用户去检查这个设置。

字体方面，`validate_content.py` 列出所有大于 U+FFFF 的码位，在真机上截图检查。缺字就用 fonttools 从 Noto Sans HK 裁出只含这些字的子集字体放进 bundle。Noto Sans HK 覆盖部分扩展 B，但不是全部 ([Fontsource](https://fontsource.org/fonts/noto-sans-hk/about))。SwiftUI 没有现成的 ruby 注音，可以按字排 VStack，或用 Core Text 的 `CTRubyAnnotation` 包一层视图，后者尚未验证。侧载名额是硬约束：免费 Apple ID 最多同时装 3 个侧载 App（SideStore 自己算一个），签名 7 天过期 ([sideload.md](https://github.com/li1000v2/mamma-mia/blob/main/docs/sideload.md))，SideStore、MammaMia、Diu 正好占满。意大利行程在 12 月，行程结束前两个 App 只能共存；之后卸掉 MammaMia 前要先导出进度。同理 Diu 只能有一个 target，不加小组件等扩展，不用 APNs、iCloud、App Groups，持久化格式只做加法改动。

## 六周 MVP 先探针后功能，读音推导提前进第一版

复用 mamma-mia 骨架后，MVP 可以压到 4–6 周，聚焦“会听、会认粤拼声调、能开口说核心句”。和单纯照搬架构相比，我在顺序上做了一处调整：读音推导的**预计算规则标签**提前进 MVP。它全在 `tools/` 离线完成，只占字卡背面的一小块界面，却是这位学习者独有的加速器，也是整个市场的空白。完整的规律浏览页和赣语对照页留到第二阶段。AI 角色扮演放在第二阶段，是因为它同时依赖识别和 TTS 两项探针的结论。

| 阶段 | 时长 | 交付 | 进入下一阶段的条件 |
|---|---|---|---|
| 第 0 周：技术探针 | 2–3 天 | 真机探测粤语识别 locale 和输出字形，列出 zh-HK 声音，检查 𨋢、𡃁、𠺢 是否显示为方框；试 Gemini TTS 念粤语，试 Chirp 3: HD yue-HK 的声线和 `<phoneme alphabet="jyutping">`；用户完成方言自测 | 结论写进 `docs/`，选定识别主路径、TTS 引擎，以及启用哪些赣语模块 |
| 第 1–2 周：内容与核心包 | 2 周 | DiuCore 改名；`Cantonese/` 模块（粤拼解析、切分、折叠、字表、数字）；新 schema；`build_tables.py`（LSHK + rime-upstream → `Content/data/`）、`annotate.py`（ToJyutping + OpenCC + canto-filter）、粤语版 `validate_content.py`；常用字规则命中率报告；粤拼入门 8–10 课，情景句 150–300 句 | 全部句子通过粤拼字典校验和 canto-filter，状态到 checked |
| 第 3–4 周：App 功能 | 2 周 | 课程页；复习（听辨、回忆、普→粤对译、字→粤拼、助词完形）；按调着色、慢速播放；点字查粤拼；开口说按字比对；基频曲线 v0；每天 5 分钟 HVPT | 音频通过粤语版粗筛，核心句由母语者抽检 |
| 第 5–6 周：打磨 | 1–2 周 | 诊断页（识别模型、声音、字体）、备份导入导出、CI 出 IPA、签名到期提醒 | 真机连续使用一周无数据丢失 |
| 第 7–12 周：第二阶段 | 约 6 周 | AI 角色扮演（粤语提示词、标签格式、本地注音、反馈卡）；tshet-uinh 规律浏览和“赣语对照”行；真人录音的最小对立自适应训练 | 声调粗筛校准报告通过；Gemini 写口语字稳定 |
| 3 个月后 | — | 逐音节声调判定（规则 + DTW）、分级白话文阅读、内嵌粤拼输入（参考 TypeDuck）([TypeDuck](https://github.com/TypeDuck-HK/TypeDuck-Web)) | — |

内容层还有一个开放问题要用户拍板：学粤语的目的是港澳或广州出行、看港剧和 TVB、和人交流，还是纯兴趣。它决定 mamma-mia 的旅行层（行程、用到日、出示卡）要不要保留，也决定香港还是广州侧重。mamma-mia 的“句框 × 场景”课程结构不用改，换成粤语句框（唔該…、我想要…、…幾多錢？、…喺邊度？、有冇…？、可唔可以…？）和场景（茶餐廳、港鐵、的士、飲茶、街市、問路）即可。发音讲解沿用 mamma-mia 的“像什么，再写哪里不像，最后以音频为准”的搭桥格式 ([mamma-mia CLAUDE.md](https://github.com/li1000v2/mamma-mia/blob/main/CLAUDE.md))，搭桥材料从日语假名换成拼音和用户的赣语。如果用户会日语假名（mamma-mia 的学习者画像里会 ([cross_check.py](https://github.com/li1000v2/mamma-mia/blob/main/tools/cross_check.py))），讲入声“急停”时也可以借促音っ，并写明哪里不像。

## 开工前要在真机和真实调用中验证的事项

下表各项都没有一手证实，决定路线的放在前面。按 mamma-mia“先测再信”的习惯，每一项都应在 `docs/` 或 `reviews/` 里留下实测记录。

| 待验证事项 | 现有证据 | 验证方法 | 不通过时的退路 |
|---|---|---|---|
| iPhone（iOS 26.x）上 SpeechTranscriber/DictationTranscriber 对 yue-CN、zh-HK 的支持、安装与下载 | 只有 macOS 报告：supported 里有，installed 里没有 ([FluidVoice #1202](https://github.com/altic-dev/FluidVoice/issues/1202)) | 真机调用 `supportedLocale(equivalentTo:)`、`AssetInventory` 下载并 reserve，写进诊断页 | SFSpeechRecognizer → sherpa-onnx SenseVoice → 只录音对比 |
| 粤语识别输出的是繁体还是简体、口语字还是书面语，对初学者错调的容忍度 | 无 | 用 30 句含嘅、咗、喺的读句，以及故意读错调的版本测试 | 折叠表兼容两种字形；声调只信基频曲线 |
| sherpa-onnx SenseVoice int8 在 iPhone 上的体积、内存和延迟 | 无官方数字 | 集成试跑 | 首次启动时下载，或改用云端回转写 |
| `gemini-3.8-flash-tts` 能否稳定念粤语（尤其是短句），`speechConfig` 能否固定语言 | Flash-Lite 的语言表有 Cantonese，Flash 未见 ([Gemini 3.8 Flash-Lite TTS](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-lite-tts)) | 生成 50 句，用带语言标签的回转写统计 yue 比例 | `engine` 字段切到 Chirp 3: HD yue-HK |
| Chirp 3: HD yue-HK 是否已 GA、声线名称（是否有 Kore/Charon）、能否用 `<phoneme alphabet="jyutping">` | 2025-12-30 起 Preview ([release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes))；Jyutping 音素只见于 Assistant 文档 ([Google SSML](https://developers.google.com/assistant/conversational/ssml-phonemes)) | 列出声线；用多音字句做 SSML 试验 | 用 `ttsText` 同音字替换；Azure zh-HK 作第二声线 |
| Gemini 回转写粤语时会不会改写成书面语，语言标签是否可靠 | 只有 CUHK 项目“要靠提示词保留口语风格”的旁证 ([CUHK Library](https://dsprojects.lib.cuhk.edu.hk/en/projects/cuhk-60th-oralhistory/cantonese-transcription/)) | 已知文本回转写，统计口语字保留率 | 改用不同家族的 ASR（Qwen3-ASR 或 SenseVoice） |
| 粗筛模型（Argon）判断粤语声调的准确率 | mamma-mia 实测它对双辅音“自信地判错” ([audio_review.py](https://github.com/li1000v2/mamma-mia/blob/main/tools/audio_review.py)) | 写 `calibrate_tone_screen.py`，用已知读音和故意错调的片段测试 | 声调意见只当提示，交给人和母语者听 |
| Gemini（Argon/Flash）在角色扮演中写港式口语字是否稳定，会不会混入书面语或简体 | 无独立评测；HKCanto-Eval 显示通用模型正字法薄弱 ([HKCanto-Eval](https://arxiv.org/html/2503.12440)) | 100 回合对话过 canto-filter，统计 cantonese/mixed 比例 | 提示词加约束，或换模型；对输出做规则修正 |
| iOS 26 的 zh-HK 声音列表、质量档，以及“朗读内容”覆盖 bug | 论坛称 iOS 26 仍存在 ([772348](https://developer.apple.com/forums/thread/772348)) | 诊断页列出已装声音，切换系统设置复测 | 引导用户改设置；全部改用预生成音频 |
| 系统字体对 𨋢、𡃁、𠺢、𠮶、㗎 的覆盖，以及 ruby 排版 | iOS 18 起 PingFang 资源可能未下载 ([Apple Forums](https://origin-devforums.apple.com/forums/profile/rcombs/replies)) | 真机截图；试 `CTRubyAnnotation` | 打包 Noto Sans HK 子集；改用 VStack 逐字排 |
| 大陆拼音输入法能否打出嘅、咗、喺 | 无可引用资料 | 真机试用常用输入法 | App 内粤字候选条，引导安装粤拼键盘 |
| SideStore、LocalDevVPN 在最新 iOS 26.x 上能否安装和续签，3 个名额如何分配 | mamma-mia 文档只写了自检步骤 ([sideload.md](https://github.com/li1000v2/mamma-mia/blob/main/docs/sideload.md)) | 装一次并立即续签一次 | 推迟 Diu 上机，或行程后卸掉 MammaMia |
| Diu 使用的 Google Cloud 项目状态 | 原项目 2026-10-16 被停用，没有恢复记录 ([CLAUDE.md](https://github.com/li1000v2/mamma-mia/blob/main/CLAUDE.md)) | 测试连接，看 GeminiFailure 分类结果 | 按规则申诉，不新开项目绕过 |
| 普→粤规则的实际命中率、`gwongzau.js` 的输出格式和准确率 | 只有民间“90%”和单项 89% 统计 ([ResearchGate](https://www.researchgate.net/figure/Major-tone-correspondences-between-Cantonese-and-Mandarin-words_tbl2_262364609)) | 用 LSHK + Unihan 对常用 3500 字计算 | 只展示命中率高的规则，其余逐字记 |
| 用户方言点的实际表现：入声几调、有无 -m/-p、全浊是否送气、是否送气分调、入声收 -t 还是 -k（如“北、色”）、有无 ng-、n/l 是否相混 | 只有百科和二手描写，没有学习者研究 ([赣语](https://zh.wikipedia.org/zh-cn/%E8%B4%9B%E8%AA%9E)) | 首次启动的 10 字方言自测，用户录下自己的方言读音 | 关闭对应的赣语模块，退回普通话规则 |

还有几项不需要真机，但同样没有核实，在把数据放进公开仓库之前要查清：ToJyutping 的许可证（LICENSE 文件写 BSD-2，awesome 列表写 CC BY 4.0）；words.hk 各类标注各自的适用范围；tshet-uinh-data、WenetSpeech-Yue、CantoneseLLM v2 权重的许可证；OpenCC 官方 C++ 版处理嗰、喎、餸是否与 Python 移植版一致；swift-fsrs 是否已发布带 FSRS-6 的正式 tag ([swift-fsrs](https://github.com/open-spaced-repetition/swift-fsrs))。

## 结论

这个项目最反直觉的一点是：一个能读写中文的人学粤语，书写系统几乎是最轻的部分。25 个左右的方言字、一张俗写折叠表、一份繁简覆盖表就能处理掉；真正的工作量在“熟字的粤音”和“原速听力”。所以 Diu 的核心与其说是一门课程，不如说是一台“熟字读音机”：先预测（普通话规则，加上赣语送气和入声线索），再验证（字典粤拼和真人或 TTS 音频），把例外交给 FSRS 记忆，声调交给不打分的曲线和 HVPT。赣语的价值高度集中在一个具体格子上，即普通话无法判断的去声、入声字“读 3 还是 6”，而且价值大小随方言点变化。因此赣语模块应该当作可证伪的假设来做：自测、记录命中率、无效就关掉。不能当作卖点预设成立。

mamma-mia 的工程哲学“在模型不可靠的地方用流程绕开”，搬到粤语后反而更重要。意大利语里需要人工听审的只是双辅音；在粤语里，声调、多音字、文白读和“被念成普通话”都属于这类风险，而学习者本人在入门期恰恰听不出声调错误。这意味着两个与 mamma-mia 不同的取舍：母语者抽检和真人录音的权重要明显提高；粤拼必须由字典确定性生成，再由第二个模型和人复核，不能交给任何单个模型。只要第 0 周的探针证实 iPhone 端侧识别和 Gemini TTS 至少有一个能用于粤语，其余路线都已有开放数据和现成代码支撑。
