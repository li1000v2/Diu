# 粤语开放数据、NLP 与语音技术资源清单（含授权）——面向粤语学习 App（可能商用）

> 调研日期：2026-10-24。说明：本次调研环境中 huggingface.co、arxiv.org、learn.microsoft.com、docs.cloud.google.com、docs.aws.amazon.com、words.hk、cantonese.org、kaifangcidian.com 等域名无法直接抓取（DNS 失败），相关事实主要来自搜索引擎摘要（附原始 URL）与可访问的 GitHub 页面；凡未能打开一手页面核实者均已标注。调研后期 WebSearch 配额耗尽，部分问题（HKGAI、OpenSubtitles、Google STT 语言表、HKUST/CUSENT 语料授权等）未能继续检索，列入 Gaps。

## 1. 词典 / 词库（Dictionaries & Lexicons）

### Takeaway
可直接用于商用 App 的「字音/词音」数据首选 **rime-cantonese / rime-cantonese-upstream（CC BY 4.0）+ LSHK 粤拼字表（CC BY 4.0）**，带文白读、变调标注；英文释义可用 **CC-Canto / CC-CEDICT 粤拼（CC BY-SA 3.0，需传染式共享）**；最丰富的粤语释义+例句来源 **words.hk 粤典** 采用「非商业开放资料授权」，商用须另行取得授权；CUHK《粤语审音配词字库》未见开放授权。

### Cited Findings
**rime-cantonese（粤拼输入法方案，CanCLID 维护）**
- 授权：主体「共享创意-署名-4.0 国际」（CC BY 4.0），其中 `jyut6ping3.maps` 为「开放资料库授权-1.0」（ODbL 1.0）；粤拼遵循 LSHK 标准，IPA 版本基于 Bauer & Benedict《Modern Cantonese Phonology》(1997) — [GitHub rime/rime-cantonese](https://github.com/rime/rime-cantonese)
- 上游词库已迁至 `CanCLID/rime-cantonese-upstream`；Wiki 首页最后编辑于 2025-11-08（仍在维护）— [rime-cantonese Wiki](https://github.com/rime/rime-cantonese/wiki)
- rime-cantonese-upstream：CC BY 4.0；包含 `char.csv`（字、粤拼、读音等级 预设/常用/罕見/棄用、本调/变调、文读/白读、注释）、`variant.csv`、`word.csv`、`fixed_expressions.csv`（成语/歇后语等）、`phrase_fragment.csv`、`trending.csv`；词条记录变调后的实际读音，用 OpenCC 字形；字音主要来源为 LSHK 粤拼字表，参考 Unihan 12.0 kCantonese、《粤语审音配词字库》、《广州话正音字典》，词条参考粤典、冚唪唥粤文、《实用广州话分类词典》、A Dictionary of Cantonese Slang 等；页面未给出词条数 — [GitHub CanCLID/rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream)
- PyCantonese 打包了 rime-cantonese 2021.05.16 版用于分词和字转粤拼，该数据为 CC BY 4.0 — [PyPI pycantonese](https://pypi.org/project/pycantonese)

**words.hk 粤典**
- 官方「开放资料授权协议」页：部分内容以 Non-Commercial Open Data License 1.0 发布，版权人为香港辞书有限公司；只有标记为开放资料的内容适用；允许再利用、修改、再分发，但需保留版权声明、署名贡献者、不得再授权，**未经特别批准不得商用** — [words.hk 開放資料授權協議](https://words.hk/base/hoifong/)（经搜索摘要，未能直接打开）
- 第三方聚合站 jyutjyu.com 同样列为 Non-Commercial Open Data License 1.0，商用需单独授权，称收入低于地区中位数 3 倍的小型个人商户可能豁免 — [jyutjyu.com About](https://jyutjyu.com/en/about)
- 冲突：words.hk 分析页（faiman/analysis）的数据集标注「public domain」— [words.hk analysis](https://words.hk/faiman/analysis/)；而 Read Path 项目称 words.hk 数据转储标注为 all-rights-reserved — [GitHub leecommamark/read-path-glosses](https://github.com/leecommamark/read-path-glosses)
- 完整 CSV 数据需通过 `words.hk/faiman/request_data/` 表单申请；解析工具 wordshk-tools 代码为 MIT（不涵盖数据）— [GitHub AlienKevin/wordshk-tools](https://github.com/AlienKevin/wordshk-tools)
- 下游 Yomitan 词典声明沿用 Non-Commercial Open Data License 1.0 — [GitHub MarvNC/wordshk-yomitan](https://github.com/MarvNC/wordshk-yomitan)
- words.hk Android App（F-Droid）代码为 MIT，最新列出 3.5.0 beta，需 Android 10+ — [F-Droid words.hk](https://f-droid.org/en/packages/hk.words.wordshk/)

**CC-Canto / CC-CEDICT 粤拼（Pleco，cantonese.org）**
- 三个文件（CC-Canto、CC-CEDICT 的粤语读音等）均以 **CC BY-SA 3.0** 发布；版权 © 2015-16 Pleco Software；采用 CC-CEDICT 格式，在拼音后用 `{}` 附粤拼；官方自称 CC-Canto 释义格式「very rough」 — [cantonese.org Download](https://cantonese.org/download.html)（经搜索摘要）
- 设计为配合 CC-CEDICT 使用：只为 CC-CEDICT 缺失或释义差异大的粤语词建立条目；2015 年 beta 时约 22,000 条 — [Chinese-Forums 公告](https://chinese-forums.com/forums/topic/49837-cc-canto-new-open-source-cantonese-english-dictionary)
- 羊羊粤语 shyyp.net 亦按 CC BY-SA 3.0 署名使用 cantonese.org 英文释义 — [shyyp.net License](https://shyyp.net/license)

**LSHK 粤语拼音字表（jyutping-table）**
- LSHK 粤拼工作组维护，提供 29,000+ 字的粤拼（TSV `list.tsv` + JSON，旧版在 `versions`，含 2023 版），原仅 PDF；**CC BY 4.0**；不定期更新 — [GitHub lshk-org/jyutping-table](https://github.com/lshk-org/jyutping-table)

**Unihan kCantonese**
- 历史上 kCantonese 部分来源（LSHK Jyutping Phrase Box）有致谢要求，5.2.0 beta 因「病毒式」许可顾虑移除该来源，影响不到半打字 — [Unicode 邮件列表 2009](https://www.unicode.org/mail-arch/unicode-ml/y2009-m06/0079.html)
- 2021 年 UTC 文件建议 Unicode 14.0 接受 26,698 个 kCantonese 值，另 3,470 个待粤语专家核查 — [UTC L2/21-015](https://www.unicode.org/L2/L2021/21015-cjk-unihan-group-utc166.pdf)

**CUHK《粤语审音配词字库》**
- 列于中大人文电算研究中心网站，未找到开放授权或使用条款 — [CUHK humanum](https://humanum.arts.cuhk.edu.hk/web/)；[字库页面](https://humanum.arts.cuhk.edu.hk/Lexis/Canton2/popup.html)
- PolyU 记录称其基于该中心两个早期数据库构建，新版增加字音、兼容多套拼音 — [PolyU OER](https://oer.lib.polyu.edu.hk/concern/works/7p88cg60s?locale=en)

**Wiktionary / kaikki.org（wiktextract）**
- wiktextract 代码 MIT；kaikki.org 每隔几天更新一次；README 未声明抽取数据授权，也未提及粤拼字段 — [GitHub tatuylonen/wiktextract](https://github.com/tatuylonen/wiktextract)
- 英文维基词典镜像显示词条文本为 CC BY-SA，媒体文件可能另有条款 — [Kiwix Wiktionary 镜像](https://kaweah.freedombox.rocks/kiwix/wiktionary_en_all_maxi_2023-04/jik6)

**ToJyutping（字转粤拼/G2P）**
- 功能：汉字转粤拼与 IPA，返回带排序的候选读音，支持自定义词条，提供 `g2p` 函数供 TTS/ASR；自称准确率 99%（未说明测法）；Python（PyPI）+ npm `to-jyutping`；多音字依赖上下文（例：上堂、分數、到）— [GitHub CanCLID/ToJyutping](https://github.com/CanCLID/ToJyutping)
- LICENSE 文件为 **BSD 2-Clause**（© 2024 粤语计算语言学基础建设组）— [ToJyutping LICENSE](https://github.com/CanCLID/ToJyutping/blob/main/LICENSE)；冲突：awesome-cantonese-nlp 列其为 CC-BY-4.0 — [awesome-cantonese-nlp](https://github.com/CanCLID/awesome-cantonese-nlp)

**频率词表**
- Cifu：「A frequency lexicon for Hong Kong Cantonese」，GPL-3.0 — [awesome-cantonese-nlp](https://github.com/CanCLID/awesome-cantonese-nlp)（原库 https://github.com/gwinterstein/Cifu）

### Inferences
- 商用 App 的「发音底座」：LSHK 字表（单字）+ rime-cantonese-upstream（词/短语、文白读、变调）均为 CC BY 4.0，只需署名，最适合嵌入；ToJyutping（BSD-2）/PyCantonese（MIT）可作运行时 G2P，但其内置数据源自 rime-cantonese（CC BY 4.0），署名义务随数据传递。
- CC-Canto/CC-CEDICT 为 CC BY-SA 3.0：若 App 内对其改编（如合并释义、修正）再分发数据库，改编后的词典数据需按 BY-SA 共享；仅在 App 中展示一般可行，但应把词典数据作为独立模块并公开其衍生版本以降低风险（非法律意见）。
- words.hk 是粤语口语释义/例句质量最高的开放来源，但对商业 App 而言应视为「需付费/谈判授权」资源；分析页的 public domain 标注可能只适用于特定统计数据集，不宜外推到全部词典内容。
- rime-cantonese-upstream 的「文读/白读」「本调/变调」字段可直接用于处理 TTS/G2P 中的 文白异读 与 多音字 问题，这是同类数据中少见的结构化标注。

### Gaps
- 开放词典（kaifangcidian.com）粤语词典：站点无法访问、搜索无结果，授权与数据可下载性未核实。
- CantoDict：未检索到当前状态与授权（推测数据未以开放授权发布，且站点维护停滞，**未核实**）。
- 粤普对照词表（如「粤普对照」类）：未找到具备明确授权的开放数据集。
- rime-cantonese / LSHK 字表的确切词条规模（词数）未从页面获得。
- kaikki.org 粤语（Cantonese）抽取文件是否包含粤拼字段及其授权声明（通常为 CC BY-SA 4.0 + GFDL，**未核实**）。
- words.hk 音频的授权情况未找到。

## 2. 语料库（Corpora）

### Takeaway
开放口语转写语料规模都很小（HKCanCor 约 23 万词，CC BY 4.0；UD 粤语树库约 1 千句，CC BY-SA 4.0）；大规模书面粤语文本主要来自论坛/网页抓取（LIHKG、维基），授权链条复杂；现成粤-普平行语料多为非商业（CC BY-NC-SA）或授权不明。

### Cited Findings
- **HKCanCor**：约 230,000 词、10,801 个 utterance；1997-03 至 1998-08 录制的自发对话（51 篇）+ 电台节目（42 篇）+ 1 篇独白；分词、POS、LSHK 粤拼标注；**CC BY 4.0** — [TFDS HKCanCor 目录](https://tensorflow.org/datasets/community_catalog/huggingface/hkcancor?authuser=1)；[HF hkcancor](https://huggingface.co/datasets/hkcancor)
- PyCantonese：MIT；内置 HKCanCor（格式大幅改写，原数据 CC BY）；最新 v4.0.0 于 2026-03 发布（维护活跃）；功能含分词、粤拼解析/转换、POS — [GitHub jacksonllee/pycantonese](https://github.com/jacksonllee/pycantonese)；[PyPI](https://pypi.org/project/pycantonese)
- **MYCanCor（马来西亚粤语语料）**：约 20 小时视频、56 个场景、2–4 人自然对话（主要在霹雳州录制）；HKSCS 繁体转写，CHAT 格式，46 个 POS 标签；下载 200GB 需注册且标「to appear」；授权标签写「CC BY」但链接指向 CC BY-SA 4.0（不一致） — [GitHub liesenf/MYCanCor](https://github.com/liesenf/MYCanCor)
- **粤语维基（zh-yue）**：Meta 列表显示约 151,502 篇条目（未注明日期）；2022-03 快照为 122,726 篇；dump 位于 dumps.wikimedia.org/zh_yuewiki/ — [Meta List of Wikipedias](https://meta.wikimedia.org/wiki/List_of_Wikipedias)；[Wayback 2022](https://webcf.waybackmachine.org/web/20220302191019/https://meta.wikimedia.org/wiki/List_of_Wikipedias)；[zh_yuewiki dumps](https://dumps.wikimedia.org/zh_yuewiki/)
- **LIHKG / 港式网络语料**：SolarisCipher/hk_content_corpus 声明 CC BY 4.0，含 LIHKG 子集，共 9.44GB，由 8 个香港粤语/繁中来源合并，源于一篇 2025 年 ACM 关于双层语言（diglossia）的论文 — [HF hk_content_corpus README](https://huggingface.co/datasets/SolarisCipher/hk_content_corpus/blob/main/README.md)；LIHKG 爬虫 — [GitHub ayaka14732/lihkg-scraper](https://github.com/ayaka14732/lihkg-scraper)
- YueTung 论文语料来源含 LIHKG、OpenRice、粤语维基、Common Crawl，总量逾 20 亿 token（未说明授权）— [arXiv 2503.03702](https://arxiv.org/html/2503.03702v1)
- **粤-普/粤-书面中文平行语料**：botisan-ai/cantonese-mandarin-translations 为 **CC BY-NC-SA 4.0** — [HF discussion #7](https://huggingface.co/datasets/botisan-ai/cantonese-mandarin-translations/discussions/7/files)；raptorkwok/cantonese-traditional-chinese-parallel-corpus 规模 100K–1M，授权栏为空 — [HF raptorkwok](https://huggingface.co/datasets/raptorkwok/cantonese-traditional-chinese-parallel-corpus/tree/8a0e88c8ca24bb1813a1b62505e15e97f9e45592/default/validation)；相关论文称构建了 13 万对粤语-书面中文平行句 — [Lingnan Scholars](https://scholars.ln.edu.hk/en/publications/cantonese-to-written-chinese-translation-via-huggingface-translat/)；无监督普-粤 MT 论文构建约 100 万粤语句子（未说明是否公开）— [arXiv 2301.03971](https://arxiv.org/pdf/2301.03971)
- **UD_Cantonese-HK 树库**：1,004 句、13,918 词，仅测试集；来源为城大学生拍摄的三部短片字幕 + 2016-10-12 立法会主席选举会议片段；**CC BY-SA 4.0**；v2.16 发布于 2025-09-12 — [GitHub UD_Cantonese-HK](https://github.com/UniversalDependencies/UD_Cantonese-HK)；[UD 页面](https://universaldependencies.org/treebanks/yue_hk)
- **Tatoeba**：文本 CC BY（部分 CC0），音频由录音者自选授权（CC BY / BY-SA / BY-NC 或无公开授权），2018 年时最常见是 CC BY-NC 4.0，未选授权者默认不允许站外使用 — [Wikipedia: Tatoeba](https://en.wikipedia.org/wiki/Tatoeba)；[Tatoeba wiki 2019](https://en.wiki.tatoeba.org/history/show-diff-between/132/2295/2313)
- 其他历史语料（19 世纪粤语基督教文献、Candbase、Cantag、香港二十世纪中期粤语语料、早期粤语圣经数据库等），awesome 列表均未注明授权 — [awesome-cantonese-nlp](https://github.com/CanCLID/awesome-cantonese-nlp)
- 商业语料：Nexdata 粤语文本样本（付费，宣称 8,200 万条）— [HF Nexdata Cantonese_Script_Data](https://huggingface.co/datasets/Nexdata/Cantonese_Script_Data/blob/main/README.md)
- hon9kon9ize 维护 `hk-corpus-pipeline`（香港网页语料构建管线，2026-05 更新）— [GitHub hon9kon9ize](https://github.com/hon9kon9ize)；CantoneseLLM v2 论文称将公开「13 年繁体中文 Common Crawl 数据集」— [arXiv 2609.06970](https://arxiv.org/pdf/2609.06970)

### Inferences
- 对学习 App 来说，HKCanCor（CC BY 4.0）是唯一兼具口语、分词、粤拼、POS 且可商用的经典语料，适合做「真实口语例句」与词频；但年代（1997–98）较旧、规模小。
- LIHKG 等论坛数据即使上传者声明 CC BY 4.0，原始帖子的著作权/平台条款并未因此被清理，商用（尤其在 App 中直接展示原文例句）风险高，更适合仅用于模型训练评估。
- 现成粤-普平行语料商用可用性差（NC 或不明），商业产品可能需自建（例如用 LLM 生成 + 人工校对）。

### Gaps
- OpenSubtitles zh_HK / 粤语字幕语料的规模与授权（未检索到）。
- Tatoeba 粤语（yue）句子数与带音频句数（未检索到当前统计）。
- 粤语维基 dump 的文件大小、最新日期（Wikipedia 文本通常为 CC BY-SA 4.0 + GFDL，**未在本次核实**）。
- hon9kon9ize 在 HF 上的数据集（如翻译、指令数据）具体授权——HF 无法访问。

## 3. NLP 工具（分词、G2P、繁简转换、普→粤翻译）

### Takeaway
PyCantonese（MIT，2026-03 仍发新版）与 ToJyutping（BSD-2）构成可商用的分词 + 粤拼 G2P 核心；OpenCC（Apache-2.0）只做繁简与港标字形，不做普粤翻译；普→粤翻译的成熟方案主要是商用 API（Google 翻译 2024 年起支持粤语、百度翻译）或 LLM，开源 NLLB 为非商业授权。

### Cited Findings
- PyCantonese：MIT；v4.0.0（2026-03）；分词、POS、粤拼转换、解析 — [GitHub pycantonese](https://github.com/jacksonllee/pycantonese)
- ToJyutping：BSD-2-Clause，Python + npm，提供 g2p，多音字需上下文/自定义覆盖；自定义短词不会覆盖内置长词 — [GitHub ToJyutping](https://github.com/CanCLID/ToJyutping)；[LICENSE](https://github.com/CanCLID/ToJyutping/blob/main/LICENSE)
- OpenCC：Apache License 2.0；最新提及版本 1.4.2；提供 `s2hk`、`hk2s`、`t2hk`、`hk2t`，含香港词组的 `s2hkp`/`hk2sp` 仍在开发；明确「不支持普通话与粤语等语言间翻译」— [GitHub BYVoid/OpenCC](https://github.com/BYVoid/OpenCC)
- s2hk 以《香港小学学习字词表》为标准；opencc-js 称 HK 词组词典「still developing and currently small; use with care」— [opencc.byvoid.com 1.3.2](https://opencc.byvoid.com/1.3.2/)；[opencc-js README](https://cdn.jsdelivr.net/npm/opencc-js@1.4.2/README.md)
- OpenCC 的 jieba 插件（`s2hk_jieba.json` 等）为实验性外部 C++ 插件，ABI 可能变化 — [opencc.byvoid.com 1.3.x](https://opencc.byvoid.com/1.3.0/)
- 百度翻译 API 支持粤语 — [awesome-cantonese-nlp](https://github.com/CanCLID/awesome-cantonese-nlp)
- Google 翻译于 2024-06-27 新增 110 种语言，含粤语（借助 PaLM 2）— [9to5Google](https://9to5google.com/2024/06/27/google-translate-new-languages-2/)；[iClarified](https://www.iclarified.com/94068/google-translate-adds-110-new-languages)
- NLLB-200 所有模型为 **CC-BY-NC 4.0**（非商业）— [fairseq NLLB README](https://github.com/facebookresearch/fairseq/blob/nllb/README.md)
- HKCanto-Eval 的任务中包含 `zh_to_jyutping` G2P 与粤/书面中文/英文互译，可用作 G2P/翻译评测基准 — [GitHub hon9kon9ize/hkeval2025](https://github.com/hon9kon9ize/hkeval2025)
- 微调多语模型做低资源方言翻译时，方言特有词上易出现严重「语义漂移」（作者称「Probability Trap」）— [LoResLM 2026](https://aclanthology.org/2026.loreslm-1.41)

### Inferences
- 推荐管线：OpenCC（字形统一为港繁）→ PyCantonese/ToJyutping（分词 + 粤拼）→ 自定义词表覆盖高频多音字；分词质量受 rime-cantonese 词表覆盖度影响，可用 words.hk（若获授权）或自建词表补充。
- 普→粤「翻译」在学习 App 中常用于生成练习句，建议用 LLM + 人工审校，NLLB 因 NC 授权不宜商用。

### Gaps
- jieba + 粤语词典、CKIP 等其它分词器对粤语的效果与授权（未检索到可靠来源；CKIP 一般面向台湾华语，**未核实**）。
- g2pW 是否有粤语版（GPT-SoVITS 页面只称 G2PW 用于中文 TTS）。
- HF 上普→粤翻译模型（如 mt5/bart 类）的授权——检索配额耗尽、HF 无法访问。

## 4. 语音数据（Speech Data）

### Takeaway
可商用（CC0）的最大开放粤语朗读语料是 Mozilla Common Voice yue（v25.0：307 小时，其中约 211 小时已验证）+ zh-HK；规模最大的 WenetSpeech-Yue（2.18 万小时，2025-09）授权未在 README 说明；MDCC（73.6 小时）仅限研究；发音示范音频（Forvo、Tatoeba）多数有 NC 或限制性条款。

### Cited Findings
- **Common Voice yue v25.0（2026-03）**：279,321 条、307.36 小时（已验证 210.67 小时）、1,179 名说话人、28,727 个文本句；**CC0-1.0**，但 Mozilla Data Collective 数据表规定禁止重新托管/再分享、禁止识别说话人身份 — [MDC CV 25.0 Cantonese](https://datacollective.mozillafoundation.org/datasets/cmn29rqn9016to107eniyak65)
- v24.0（2025-12-05）：307.7 小时 / 已验证 210.9 小时，1,174 人 — [MDC CV 24.0](https://datacollective.mozillafoundation.org/datasets/cmj8u3q2b00v9nxxborfkm824)；v23.0：309 / 203 小时 — [MDC CV 23.0](https://datacollective.mozillafoundation.org/datasets/cmflnuzw81ibb5hby3l6fnj3c)
- zh-HK locale v25.0：CC0-1.0，MP3，3.42 GB（小时数未知）— [MDC 数据集列表](https://datacollective.mozillafoundation.org/datasets?page=2)；早期 zh-HK 仅约 60 小时（37 已验证），且混杂书面语与口语，因此社区申请独立 yue 代码 — [Mozilla Discourse](https://discourse.mozilla.org/t/requesting-the-cantonese-language-yue/69928)；alvanlii 模型训练用了 CV16 zh-HK train 138 小时 — [alvanlii/whisper-small-cantonese 卡片](https://huggingface.co/alvanlii/whisper-small-cantonese/blob/8e56c47d26dae849fb45a6b519bddc17e065c949/README.md)
- **WenetSpeech-Yue**：21,800+ 小时，10 个领域（故事、娱乐、戏剧、文化、vlog、评论、教育、播客、新闻等），JSON 元数据含转写置信度、说话人、SNR、DNSMOS、年龄、性别、字级时间戳；转写由 SenseVoice/TeleASR/Whisper 投票 + Qwen3-4B 修正；2025-09-08 发布，2026-01-07 修正元数据；论文 arXiv 2509.03959；README 未写授权 — [WenetSpeech-Yue README](https://raw.githubusercontent.com/ASLP-lab/WenetSpeech-Yue/main/README.md)
- **MDCC**：73.6 小时香港有声书朗读语音（train 57.5 / val 5.05 / test 11.1 小时）— [arXiv 2201.02419](https://arxiv.org/abs/2201.02419v2)；需签署 MDCC_LICENSE 邮件申请，.wav「research purposes only」，2024-02-01 释出 wav — [GitHub HLTCHKUST/cantonese-asr](https://github.com/HLTCHKUST/cantonese-asr)
- **CantoMap**：香港粤语 Map Task 语料，768 分钟（约 12.8 小时）、40 名说话人，汉字 + 罗马字转写，ELAN 对齐；论文 PDF 页脚为 ELRA CC BY-NC（未必等同数据授权）— [LREC 2020](https://aclanthology.org/2020.lrec-1.355)
- **KeSpeech** 不含粤语（仅普通话及 8 种官话次方言），学术免费 — [NeurIPS 2021](https://neurips.cc/virtual/2021/36900)；[VoxKitchen 目录](https://voxkitchen.readthedocs.io/en/latest/datasets/kespeech/)
- Nexdata 方言语料共 25,000 小时，含粤语（商业）— [arXiv 2512.07005](https://arxiv.org/pdf/2512.07005)
- **Forvo**：2019 年底移除 CC 条款，改为严格限制复制/修改/再分发的自定义许可；API 收费（非商业 $2/年）— [Wikipedia: Forvo](https://en.wikipedia.org/wiki/Forvo)；早期授权说法冲突（CC BY-SA 3.0）— [itsfoss 社区](https://itsfoss.community/t/review-about-lingua-libre-a-foss-alternative-to-forvo-com/8066)
- **Lingua Libre**：内容 CC BY-SA 4.0，由 Wikimédia France 运营 — [Wikipedia: Lingua Libre](https://www.wikipedia.com/wiki/Lingua_Libre)
- 社区评测使用的测试集包括 Guangzhou Daily Use、Guangzhou Cabin（指令）、粤英混说、三国演义说书、words.hk 句子经 AWS Polly 合成等 — [GitHub AlienKevin/cantonese_asr_eval](https://github.com/AlienKevin/cantonese_asr_eval)

### Inferences
- 训练/微调可商用 ASR：Common Voice yue + zh-HK（CC0）是最稳妥的基础；但 MDC 条款禁止再托管，App 不能直接打包分发原始数据集（训练出的模型不受此限，需自行确认条款）。
- Common Voice 文本句多为书面/朗读风格，zh-HK 早期混杂书面语——对口语粤语识别与发音评估可能存在领域偏差。
- 学习 App 的「标准示范音频」最好自录（雇母语配音）或用可商用 TTS 生成，Forvo/Tatoeba/LinguaLibre 都有 NC 或 SA 约束。

### Gaps
- WenetSpeech-Yue 的授权（HF 数据集卡无法访问；**未核实**，可能为非商业研究用途）。
- HKUST Cantonese、CUSENT、CU Corpora、Babel Cantonese 的授权与获取方式（arXiv 2201.02419 无法访问）。
- MagicData 广州粤语开源子集是否存在及授权（未找到）。
- Wiktionary/Commons 粤语读音音频规模；words.hk 音频授权。

## 5. 语音识别 ASR（开源 / 云端 / 端侧）

### Takeaway
2026 年开源粤语 ASR 最强的是 **Qwen3-ASR（Apache-2.0，2026-01）**，在 CV-yue、WenetSpeech-Yue 上优于 GPT-4o-Transcribe、Gemini-2.5-Pro、Whisper-large-v3；端侧可选 **SenseVoice-Small（sherpa-onnx/GGUF）** 或 WenetSpeech-Yue 的 Conformer-Yue（~130M）；原版 Whisper 对粤语倾向输出书面中文，需用粤语微调版。

### Cited Findings
**开源模型与 CER/WER**
- Qwen3-ASR（0.6B/1.7B + ForcedAligner-0.6B）于 2026-01-29 发布，支持 30 种语言 + 22 种汉语方言，含「粤语（香港口音）」「粤语（广东口音）」；代码 Apache-2.0；页面表格（标注为 WER）：Fleurs-yue：GPT-4o-Transcribe 4.98 / Gemini-2.5-Pro 9.43 / Doubao-ASR 4.98 / Whisper-large-v3 9.18 / Qwen3-ASR-0.6B 5.79 / **1.7B 3.98**；CV-yue：11.36 / 18.76 / 13.20 / 16.23 / 9.50 / **7.57**；WenetSpeech-Yue short：15.62 / 25.19 / 9.74 / 32.26 / 7.54 / **5.82**；long：25.29 / 11.23 / 11.40 / 46.64 / 9.92 / **8.85** — [GitHub QwenLM/Qwen3-ASR](https://github.com/QwenLM/Qwen3-ASR)
- 阿里云博客称 Qwen3-ASR 1.7B/0.6B 与 ForcedAligner 权重以 Apache 2.0 开源 — [Alibaba Cloud Blog](https://www.alibabacloud.com/blog/qwen3-asr-%26-qwen3-forcedaligner-is-now-open-sourced-robust-streaming-and-multilingual_602843)
- 基于 Qwen3-ASR-0.6B 蒸馏的 Luigi/asr-324m-apache（Apache-2.0）在 200 条 FLEURS 粤语上 CER 22.54（样本小）— [HF Luigi/asr-324m-apache](https://huggingface.co/Luigi/asr-324m-apache)
- WenetSpeech-Yue 的 WSYue-eval（Short 0–10s / Long 10–30s）：Conformer-LLM-Yue 4.73/7.91；Conformer-Yue（~130M）5.05/8.89；Whisper-m-Yue（~769M）5.05/8.05；SenseVoice-s-Yue（~234M）5.23/8.63；SenseVoice-small 6.69/9.95；Whisper-large-v3 12.95/26.86；Whisper-medium 80.82/50.96；Paraformer 73.64/89.00；Qwen2.5-Omni-3B 67.95/88.46 — [WenetSpeech-Yue README](https://raw.githubusercontent.com/ASLP-lab/WenetSpeech-Yue/main/README.md)
  - 冲突：同一 WenetSpeech-Yue 测试上，Qwen3-ASR 页面给 Whisper-large-v3 为 32.26/46.64，与 WenetSpeech-Yue README 的 12.95/26.86 差异大（文本归一化/解码设置可能不同）。
- 社区评测（文本转简体、去标点后 CER）：CV17 test：SenseVoiceSmall 7.70% / Scrya whisper-large-v2-cantonese 5.55% / alvanlii whisper-small-cantonese 8.00%；广州日常：5.64 / 12.38 / 14.60；广州车载指令：7.45 / 14.22 / 17.83；粤英混说：9.05 / 20.85 / 25.57；说书：14.67 / 18.54 / 25.06；words.hk 合成句：10.58 / 14.20 / 15.05 — [GitHub AlienKevin/cantonese_asr_eval](https://github.com/AlienKevin/cantonese_asr_eval)
- alvanlii/whisper-small-cantonese：CV16 yue test 归一化 CER 7.93（保留标点 9.72）— [HF 模型卡](https://huggingface.co/alvanlii/whisper-small-cantonese/blob/8e56c47d26dae849fb45a6b519bddc17e065c949/README.md)
- khleeloo/whisper-large-v3-cantonese 卡片中「Whisper large v3」行：CV15 zh-HK 10.8、CV15 yue 16（未明确是否为基线）— [Friendli 模型页](https://friendli.ai/models/khleeloo/whisper-large-v3-cantonese)
- Whisper large-v3（2023-11-06）新增粤语语言 token（`yue`），训练数据 100 万小时弱标注 + 400 万小时伪标注 — [openai/whisper Discussion #1762](https://github.com/openai/whisper/discussions/1762)
- Whisper 对粤语常只输出书面中文；HKUST 图书馆指南称 large-v3 可直接把粤语语音转为书面中文，建议先得到粤语文本再用 LLM 转书面语 — [Migaku 社区](https://migaku.com/community/m/1366805594614403123)；[HKUST Library](https://library.hkust.edu.hk/sc/?p=3750)
- SenseVoice-Small：支持 `yue`；代码 MIT，权重遵循 FunASR 模型许可；维护者在 issue 中澄清遵守模型许可前提下官方权重可商用、微调衍生权重可不公开，但需遵守 2.2 节署名/保留模型名 — [GitHub FunAudioLLM/SenseVoice](https://github.com/FunAudioLLM/SenseVoice)
- FunASR 模型许可：可使用、复制、修改、分享；须注明来源与作者并保留模型名；「仅供参考与学习」免责；禁止「无理诋毁」否则自动丧失授权；协议可单方修订；管辖法律条款仍为占位符 — [FunASR MODEL_LICENSE](https://github.com/modelscope/FunASR/blob/main/MODEL_LICENSE)
- hon9kon9ize/wav2vec2bert-jyutping：Wav2Vec2-BERT 2.0 微调，直接输出带声调的粤拼（如 maa4 maa1 go3），用 CV17 粤语训练，未报告准确率与授权 — [GitHub hon9kon9ize/wav2vec2bert-jyutping](https://github.com/hon9kon9ize/wav2vec2bert-jyutping)
- CantoASR（2025-11）：LoRA 微调 Whisper 提升声调辨别，再用音频语言模型修正 — [arXiv 2511.04139](https://arxiv.org/html/2511.04139v1)

**端侧**
- sherpa-onnx：Apache-2.0；支持 Android、iOS、HarmonyOS、桌面、WebAssembly、Flutter；提供 `sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17` 与 Paraformer-large 中英粤实时识别 — [GitHub k2-fsa/sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx)；1.12.12 changelog 引入 WSYue `u2pp_conformer_yue` — [sherpa_onnx pub 镜像](https://pub-web-s1.flutter-io.cn/packages/sherpa_onnx)
- SenseVoice 端侧：funasr-onnx（可量化）、libtorch、llama.cpp/GGUF（内置 FSMN-VAD）、SenseVoice.cpp（3–8 bit 量化）— [GitHub SenseVoice](https://github.com/FunAudioLLM/SenseVoice)
- Qwen3-ASR-0.6B 有 MLX 8bit 版本（Apple Silicon）— [HF majentik/Qwen3-ASR-0.6B-MLX-8bit](https://huggingface.co/majentik/Qwen3-ASR-0.6B-MLX-8bit)

**云端**
- 腾讯云：官方文档称 `16k_zh_en_2.0`（中英大模型 2.0）可识别粤语及多种方言并支持说话人分离；`16k_multi_lang` 不含粤语 — [腾讯云 ASR API 文档](https://cloud.tencent.com/document/api/1093/37823)；第三方整理称有 `16k_yue` 粤语引擎 — [ClawHub tencentcloud-asr](https://clawhub.ai/Stardusten/tencentcloud-asr)
- 讯飞：旧版 SDK 听写参数 `accent` 可选普通话/粤语 — [npm xfy-node](https://www.npmjs.com/package/xfy-node)；方言试用需在控制台设置识别语种，星火短音频接口宣称支持 202 种方言（需核实）— [GitCode 教程](https://gitcode.csdn.net/69de1d2d0a2f6a37c59faada.html)
- 云端 SOTA 对比（GPT-4o-Transcribe、Gemini-2.5-Pro、Doubao-ASR）见上方 Qwen3-ASR 表 — [GitHub QwenLM/Qwen3-ASR](https://github.com/QwenLM/Qwen3-ASR)

### Inferences
- 商用自托管首选 Qwen3-ASR-1.7B（服务器）/0.6B（端侧或 MLX），授权干净（Apache-2.0）且粤语指标最好；SenseVoice-Small 速度快、生态成熟（sherpa-onnx 移动端），但 FunASR 模型许可有「署名 + 不得诋毁 + 单方修订」等非标准条款，需法务评估。
- 学习 App 场景中 ASR 应输出「粤语口语字」而非书面语：原版 Whisper 不合适；Qwen3-ASR/SenseVoice/粤语微调 Whisper 更合适，但仍需人工评估口语字（嘅、咗、喺、冇）输出比例。
- 直接输出带声调粤拼的模型（wav2vec2bert-jyutping）更适合做发音/声调反馈，而非文本识别。
- CER 数字受文本归一化（繁简转换、标点、口语字/书面字映射）影响极大，各来源不可直接横比；需在自有测试集上复测。

### Gaps
- Azure STT（zh-HK、yue-CN）、Google STT（yue-Hant-HK，chirp_2/chirp_3）粤语支持与 CER：官方页面无法访问，未核实。
- 阿里云 NLS / 百炼 粤语识别模型名、讯飞当前 WebAPI 粤语参数；各云端粤语 CER 均无公开数据。
- whisper.cpp 授权（通常 MIT，**未核实**）及粤语微调模型的 GGML 转换可用性。
- Qwen3-ASR 粤语指标是否按 CER 计算（页面标 WER）。

## 6. 语音合成 TTS（云端 / 系统内置 / 开源）

### Takeaway
商用最稳妥的是云端：Azure（zh-HK 晓曼/晓佳/云龙等 + yue-CN 云松）、Google（yue-HK，Chirp 3 HD 预览）、Amazon Polly（Hiujin，仅 neural）、阿里云百炼部分音色支持粤语；开源中 GPT-SoVITS（MIT）与 CosyVoice（Apache-2.0 代码）支持粤语，Fish-Speech 有营收门槛，MMS-TTS 为非商业；Edge TTS 非官方接口有服务条款风险。所有 TTS 的多音字/文白读需要前端粤拼校正。

### Cited Findings
**云端**
- Azure：zh-HK-HiuMaanNeural、zh-HK-HiuGaaiNeural（女声）、zh-HK-WanLungNeural（男声，支持 SSML）— [JSON2Video HiuMaan](https://json2video.com/ai-voices/azure/voices/zh-hk-hiumaanneural/)；[HiuGaai](https://json2video.com/ai-voices/azure/voices/zh-hk-hiugaaineural/)；[WanLung](https://json2video.com/ai-voices/azure/voices/zh-hk-wanlungneural/)；yue-CN-YunSongNeural（简体粤语男声）— [HF azure_utils.py](https://huggingface.co/spaces/ttt246/develop/blob/main/azure_utils.py)
- Azure 粤语 TTS 的 viseme（口型）长度与音频不匹配（2024-08 用户报告，英/普正常）— [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/1866973/visemes-dont-match-speech-for-cantonese-azure-text)；旧标准音色（Danny、TracyRUS）已弃用 — [Azure 迁移文档](https://learn.microsoft.com/vi-vn/azure/ai-services/speech-service/how-to-migrate-to-prebuilt-neural-voice)
- Google Cloud TTS：2025-12-30 起 Chirp 3: HD 以 Preview 支持 yue-HK — [Cloud TTS release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes)；[Chirp 3 HD 文档](https://docs.cloud.google.com/text-to-speech/docs/chirp3-hd?authuser=1)；用户称 yue-HK 约 34 个音色（yue-HK-Chirp3-HD-Achernar … yue-HK-Standard-D），且 Vertex AI Studio 下拉框缺少粤语（未经官方证实）— [Google Developer Forum](https://discuss.google.dev/t/text-to-speech-missing-language-chinese-cantonese-yue-hk/344555)
- Amazon Polly：Hiujin，女声，yue-CN，仅 Neural（无 Standard/Generative/Long-form），24 kHz — [AWS Polly Available voices](https://docs.aws.amazon.com/polly/latest/dg/available-voices.html)；[AWS 中国 Neural voices](https://docs.amazonaws.cn/en_us/polly/latest/dg/NTTS-main.html)；[3CX 社区](https://www.3cx.com/community/threads/amazon-polly-tts-does-not-have-cantonese-voice.130292/post-620311)
- 阿里云百炼：部分 TTS 模型的部分音色支持粤语，另一款 17 音色模型覆盖含粤语在内的方言（不支持复刻）— [阿里云 多模态音色列表](https://help.aliyun.com/zh/model-studio/multimodal-timbre-list)
- Edge TTS（rany2/edge-tts，调用 Edge 在线朗读服务）：仓库含 GPL-3.0 文件；微软 Q&A 版主称无有效 Azure 订阅的商用可能违反服务条款 — [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/2088770/are-opensource-edge-tts-free-for-commercial-use)；[GitHub rany2/edge-tts](https://github.com/rany2/edge-tts)；第三方 npm 包亦建议商用改用 Azure — [npm msedge-tts](https://npmjs.com/package/msedge-tts)

**系统内置**
- iOS：辅助功能 → 朗读内容 → 声音 → 中文 → 中文（香港）可选 Sin-Ji 音色 — [UNH TADS Lab](https://sites.usnh.edu/tadslab/cantonese-text-to-speech-tts-using-coughdrop)；iOS 18 起 Siri 设为粤语时请求 zh-CN 会读成粤语，iOS 26 仍有报告 — [Apple Developer Forums](https://developer.apple.com/forums/thread/772348)
- Android：仅找到旧报道称 Google Assistant 支持粤语，并为 Actions 提供一男一女合成音 — [Voicebot.ai](https://voicebot.ai/?p=16231)

**开源**
- CosyVoice：代码 Apache-2.0；Fun-CosyVoice3-0.5B-2512（2025-12）宣称 18+ 种方言/口音，列有「Guangdong」；README 称内容「仅供学术用途」；无粤语评测 — [GitHub FunAudioLLM/CosyVoice](https://github.com/FunAudioLLM/CosyVoice)；CosyVoice 2.0 方言列表包含粤语（fork）— [GitHub ring-3/cosyvoice](https://github.com/ring-3/cosyvoice)
- WenetSpeech-Yue 发布 CosyVoice2-Yue、CosyVoice2-Yue-ZoengJyutGaai、Llasa-1B-Yue-Updated（2025-11-15）— [WenetSpeech-Yue README](https://raw.githubusercontent.com/ASLP-lab/WenetSpeech-Yue/main/README.md)
- GPT-SoVITS：MIT；V2 起支持粤语（数据集代码 `yue`），后续版本 V3/V4/V2Pro/V5 — [GitHub RVC-Boss/GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS)
- Fish-Speech：代码与权重为 Fish Audio Community License（S2-Pro 权重为 Research License）；商用仅在年收入 < 50 万美元时免费，超过需 30 天内通知并在 90 天后另签许可；以 API/SDK/语音复刻服务对外提供需另授权；禁止用模型或输出训练/蒸馏其他生成式语音模型；须展示「Built with Fish Audio」— [Fish-Speech README](https://github.com/fishaudio/fish-speech)；[LICENSE](https://github.com/fishaudio/fish-speech/blob/main/LICENSE)
- Meta MMS-TTS：mms-tts 系列为 CC-BY-NC-4.0（yue 单独仓库未核实，推断相同）— [HF facebook/mms-tts](https://huggingface.co/facebook/mms-tts/tree/main)
- Qwen3-TTS（2026-01-22，0.6B/1.7B，Apache-2.0 代码）：官方语言列表不含粤语 — [GitHub QwenLM/Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS)；hon9kon9ize 有针对港式粤语口音的微调工程，提到「快语速崩坏」、乱码、提前结束等问题，无量化指标 — [GitHub qwen3-tts-cantonese-accent](https://github.com/hon9kon9ize/qwen3-tts-cantonese-accent)
- sherpa-onnx changelog 有粤语 VITS 音色的修复记录（模型名未知）— [sherpa_onnx pub 镜像](https://pub-web-s1.flutter-io.cn/packages/sherpa_onnx)

**多音字 / 文白异读**
- rime-cantonese-upstream 字表区分 预设/常用/罕見 读音、本调/变调、文读/白读，可作 TTS 前端 — [GitHub rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream)
- ToJyutping 指出多音字（上堂、分數、到）无上下文时有歧义，需要自定义覆盖 — [GitHub ToJyutping](https://github.com/CanCLID/ToJyutping)

### Inferences
- 学习 App 推荐：示范音用 Azure（音色多、支持 SSML `<phoneme>` 可能用于纠音，**粤语 phoneme 支持未核实**）或 Google Chirp 3 HD（预览期，生产慎用）；必要时以粤拼前端 + 人工词表纠正多音字。
- 若需离线/自托管且商用：GPT-SoVITS（MIT）或 CosyVoice（Apache-2.0 代码，但注意 README「学术用途」声明与权重卡条款）；Fish-Speech 仅适合小规模且不对外提供 API 的场景；MMS 非商用。
- Edge TTS 不应用于商业产品。
- iOS 内置 Sin-Ji 可做零成本离线朗读，但 iOS 18+ 的语种路由 bug 可能导致普粤混读，需测试。

### Gaps
- Azure zh-HK/yue-CN 完整音色列表（XiaoMin 等）及粤语 SSML phoneme（jyutping）支持，官方页面不可达。
- Google yue-HK 确切音色名单与 Neural2/WaveNet 情况；Android 内置 Google TTS 粤语音色现状。
- CosyVoice2/3、CosyVoice2-Yue、Llasa-Yue 权重授权（HF/ModelScope 不可达）。
- 各 TTS 声调准确率的量化评测（未找到）；Bert-VITS2 粤语、HF「Cantonese TTS」模型清单未检索。

## 7. 发音 / 声调评估（Pronunciation & Tone Assessment）

### Takeaway
现成商用 API 中只有 Azure Pronunciation Assessment 据称支持 zh-HK（仅基础分数，无音素名/韵律），SpeechSuper 未见粤语；粤语声调评估基本需要自建：以带声调粤拼识别模型 + 强制对齐 + 基频（F0）提取实现，F0 工具 CREPE（MIT）可商用，Parselmouth/Praat 为 GPL-3.0。

### Cited Findings
- Azure 发音评估文档的 GitHub 副本将 zh-HK 列为 Chinese (Cantonese, Traditional)（本次直接抓取该文件 404，可能已迁移）— [MicrosoftDocs azure-ai-docs（搜索结果）](https://github.com/MicrosoftDocs/azure-ai-docs/blob/main/articles/ai-services/speech-service/includes/language-support/pronunciation-assessment.md)
- 2024-08 Azure 发音评估 33 种语言 GA，各语言在全部 STT 区域可用 — [Azure Speech release notes](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/releasenotes)
- 音素名输出仅 en-US（IPA）与 en-US/zh-CN（SAPI）；内容与韵律评估仅 en-US — [Azure language learning overview](https://docs.azure.cn/en-us/ai-services/speech-service/language-learning-overview)（经搜索摘要）
- SpeechSuper 官方样例仓库列出英、中、德、法、俄、韩、日、西语，未提粤语；其「升降调」分析为句末语调而非词汇声调 — [GitHub SpeechSuper-API-Samples](https://github.com/speechsuper/SpeechSuper-API-Samples)
- 学术：CantoASR 用 LoRA 微调 Whisper 增强声调辨别（2025-11）— [arXiv 2511.04139](https://arxiv.org/html/2511.04139v1)；MDPI《Symmetry》2025 音素感知增强粤语 ASR（视粤语为 9 声）— [MDPI Symmetry 17(9):1478](https://www.mdpi.com/2073-8994/17/9/1478/xml)；KIT 硕士论文帧级粤语声调识别 LSTM 最佳，帧错误率 63.47%（7 类）— [KIT ISL](https://isl.iar.kit.edu/english/MA_FlorianDessloch.php)
- hon9kon9ize/wav2vec2bert-jyutping 直接输出带声调粤拼，可作「读音/声调识别」基础 — [GitHub](https://github.com/hon9kon9ize/wav2vec2bert-jyutping)
- Qwen3-ForcedAligner-0.6B（Apache-2.0）可做时间对齐 — [Alibaba Cloud Blog](https://www.alibabacloud.com/blog/qwen3-asr-%26-qwen3-forcedaligner-is-now-open-sourced-robust-streaming-and-multilingual_602843)；WenetSpeech-Yue 提供字级时间戳 — [README](https://raw.githubusercontent.com/ASLP-lab/WenetSpeech-Yue/main/README.md)
- Parselmouth：GPL-3.0+（Praat 本体 GPL-2.0+，依赖 GPL-3.0+）— [GitHub YannickJadoul/Parselmouth](https://github.com/YannickJadoul/Parselmouth)
- CREPE：MIT；模型容量 tiny/small/medium/large/full 可选 — [GitHub marl/crepe](https://github.com/marl/crepe)

### Inferences
- 推荐方案：①ASR/对齐得到每个音节时间段 → ②CREPE-tiny（或 pYIN）提取 F0 → ③按说话人基频范围归一化（如 5 度标调/z-score）与 6 声模板比对 → ④可视化声调曲线；同时用 wav2vec2bert-jyutping 类模型做「识别出的声调 vs 目标声调」判定。Azure zh-HK 可作为整体准确度/流利度打分的补充。
- Parselmouth 为 GPL：若在闭源 App 客户端打包会触发 copyleft，可只在服务器端使用或改用 CREPE/pYIN。
- 粤语变调（如 tone sandhi 、口语高升变调）需要用 rime-cantonese-upstream 的「变调」标注避免误判。

### Gaps
- Azure zh-HK 发音评估的实际粒度（音节/声调级？）与准确性未核实；Speechace 是否支持粤语未找到。
- 2024–2026 年面向二语学习者的粤语 CAPT/声调错误检测论文（检索未命中）。
- librosa pYIN 的授权（通常 ISC，**未核实**）、torchcrepe 授权。

## 8. 大语言模型（LLM）与粤语

### Takeaway
基准显示 GPT-4o 与 Qwen-2.5-72B 在 Yue-ARC-C 上达 92–94%，HKCanto-Eval 结论是闭源 > 开源但粤语特有语言/文化知识普遍薄弱；粤语专用模型以 hon9kon9ize 的 CantoneseLLM（v1.0 基于 Qwen2.5，v2 于 2026-09 基于 Qwen3）为代表；「漂移到书面语/普通话」现象有间接证据（推理语言保持源模型语言），但未找到直接量化书面语漂移的论文。

### Cited Findings
- Yue-Benchmark（Jiang et al., NAACL 2025 Findings）：评测 35 个模型，题目由英/普翻译并人工复核；Yue-ARC-C 零样本/五样本：Qwen-2.5-72B 92.74/92.91，GPT-4o 91.97/94.45；Qwen-2.5-7B 约 82–83%；早期 Qwen-7B 仅约 11–15% — [arXiv 2408.16756](https://arxiv.org/html/2408.16756v3)；[ACL Anthology](https://preview.aclanthology.org/dashboard/2025.findings-naacl.253)
- YueTung-7b：Yue-GSM8K 零样本 84.65%、五样本 86.46%，Qwen-2.5-7b 为 63.84%/44.20%，并称超过 GPT-4/GPT-4o — [arXiv 2503.03702](https://arxiv.org/pdf/2503.03702)
- HKCanto-Eval（Cheng et al., CoNLL 2025）：含 14,042 道 MMLU 粤语翻译题及香港文化、DSE、香港法律、语音（phonetics）、专业类，另有摘要、情感、翻译、zh_to_jyutping；结论：闭源普遍优于开源，粤语特有语言与文化知识仍显著不足 — [arXiv 2503.12440](https://arxiv.org/abs/2503.12440v1)；[HF Papers](https://huggingface.co/papers/2503.12440)；[GitHub hkeval2025](https://github.com/hon9kon9ize/hkeval2025)（仓库未见 LICENSE）
- CantoneseLLMChat v1.0（7B/32B/72B，基于 Qwen 2.5）：在约 6 亿（原文「600 million」）香港新闻与粤语网站数据上继续预训练，75,000 条指令（其中 45,000 条粤语，由其他 LLM 生成并人工核对）— [HF CantoneseLLMChat-v1.0-7B](https://huggingface.co/hon9kon9ize/CantoneseLLMChat-v1.0-7B)；72B 称为 HK-Eval 最佳开源模型，HK Culture 75.4%、Cantonese Linguistics 59.6%；第三方页标注 Apache-2.0 — [Featherless 72B](https://featherless.ai/models/hon9kon9ize/CantoneseLLMChat-v1.0-72B)；[Featherless 7B](https://featherless.ai/models/hon9kon9ize/CantoneseLLMChat-v1.0-7B)；32B — [HF 32B](https://huggingface.co/hon9kon9ize/CantoneseLLMChat-v1.0-32B)
- CantoneseLLM v2（arXiv 2609.06970，2026-09-07）：基于 Qwen3 8B 与 30B-A3B，7.84 亿 token 粤语/香港相关 CPT + chat-vector 合并 + SFT + DPO + RLVR；发现 chat-vector 合并保留了源模型的推理语言，少量粤语推理 SFT 会大幅缩短/去除推理链，RLVR 以「粤语 + 繁体」作为乘性约束恢复粤语对齐；30B-A3B 在 HKCanto-Eval 得 73.16；Qwen3 8B base 63.07、30B-A3B base 67.69；计划公开权重、训练环境与 13 年繁中 Common Crawl 数据 — [arXiv 2609.06970](https://arxiv.org/pdf/2609.06970)；[HF Papers](https://huggingface.co/papers/2609.06970)
- hon9kon9ize 开源 6 个 NeMo-Gym RL 环境（任务 × 语言 × 格式乘性奖励，MIT，2026-08）及粤语 Chat UI — [GitHub hon9kon9ize](https://github.com/hon9kon9ize)；[cantonese-nemo-gym-environments](https://github.com/hon9kon9ize/cantonese-nemo-gym-environments)
- 小模型：hon9kon9ize/ERNIE-4.5-0.3B-Yue — [HF](https://huggingface.co/hon9kon9ize/ERNIE-4.5-0.3B-Yue)
- 跨语言基准（粤/日/土）：GPT-4o 粤语题正确率 78%，GPT-4 74% — [arXiv 2511.10664](https://arxiv.org/pdf/2511.10664)
- 书面语/普通话漂移的间接证据：Whisper 类模型输出书面中文 — [HKUST Library](https://library.hkust.edu.hk/sc/?p=3750)；方言翻译微调存在语义漂移 — [LoResLM 2026](https://aclanthology.org/2026.loreslm-1.41)；v2 论文「donor model reasoning language」保留问题 — [arXiv 2609.06970](https://arxiv.org/pdf/2609.06970)

### Inferences
- 选型：内容生成（例句、对话、讲解）可用前沿闭源模型（GPT-4o 级别已在 Yue 基准 90%+），但须在提示中强制「香港口语粤语 + 繁体 + 口语字（嘅/咗/喺/啲）」并做自动检测（例如统计书面语标志词「的/了/在/是」与粤语标志字比例）后再人工抽检。
- 自托管/可微调：CantoneseLLM v2（Qwen3 基座）是 2026 年最新开源粤语模型，但需确认 Qwen3 / Qwen2.5 基座许可（Qwen2.5-72B 等部分尺寸使用 Qwen 自定义许可，**未在本次核实**）。
- 基准题多由英/普翻译而来，测的是「理解」，不等于「地道口语生成」；学习 App 需建立自己的生成质量评测集。

### Gaps
- GPT-5/Claude/Gemini 2.5+/DeepSeek 在 HKCanto-Eval 上的逐模型分数（arXiv 不可达，GitHub 仓库无结果表）。
- 直接量化 LLM 粤语生成「书面语化 / 普通话词汇混入」的研究（未检索到）。
- HKGAI（香港生成式 AI 研发中心）V1 模型、SeaLLM 粤语能力（搜索配额耗尽未检索）。
- CantoneseLLM v2 权重的确切授权。

## 9. 商用授权影响（Licensing Implications）

### Takeaway
可放心商用（仅需署名或无义务）：Common Voice（CC0，但禁再托管）、HKCanCor/LSHK/rime-cantonese（CC BY 4.0）、PyCantonese（MIT）、ToJyutping（BSD-2）、OpenCC/sherpa-onnx/Qwen3-ASR（Apache-2.0）、GPT-SoVITS（MIT）；需传染共享：CC-Canto/CC-CEDICT、UD 树库、维基/维基词典、Lingua Libre（BY-SA）；禁止或限制商用：words.hk（NC，需授权）、MDCC（研究用）、MMS-TTS/NLLB（NC）、botisan 平行语料（NC-SA）、Fish-Speech（营收门槛）、Forvo（专有）；条款特殊需法务审查：FunASR/SenseVoice 模型许可、CosyVoice「学术用途」声明、Edge TTS（服务条款）、Parselmouth（GPL）。

### Cited Findings
- CC BY 4.0：rime-cantonese 主体、rime-cantonese-upstream、LSHK 字表、HKCanCor — [rime-cantonese](https://github.com/rime/rime-cantonese)；[rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream)；[jyutping-table](https://github.com/lshk-org/jyutping-table)；[TFDS HKCanCor](https://tensorflow.org/datasets/community_catalog/huggingface/hkcancor?authuser=1)
- ODbL 1.0：rime-cantonese 的 `jyut6ping3.maps`（数据库衍生需以 ODbL 共享）— [rime-cantonese](https://github.com/rime/rime-cantonese)
- CC BY-SA：CC-Canto（3.0）— [cantonese.org](https://cantonese.org/download.html)；UD_Cantonese-HK（4.0）— [GitHub UD](https://github.com/UniversalDependencies/UD_Cantonese-HK)；Lingua Libre（4.0）— [Wikipedia](https://www.wikipedia.com/wiki/Lingua_Libre)
- 非商业：words.hk（Non-Commercial Open Data License 1.0）— [words.hk 授權](https://words.hk/base/hoifong/)；botisan-ai 粤普平行（CC BY-NC-SA 4.0）— [HF](https://huggingface.co/datasets/botisan-ai/cantonese-mandarin-translations/discussions/7/files)；NLLB（CC-BY-NC 4.0）— [fairseq](https://github.com/facebookresearch/fairseq/blob/nllb/README.md)；MMS-TTS（CC-BY-NC-4.0）— [HF mms-tts](https://huggingface.co/facebook/mms-tts/tree/main)；MDCC（research only）— [GitHub HLTCHKUST](https://github.com/HLTCHKUST/cantonese-asr)
- CC0 + 平台条款：Common Voice yue/zh-HK（禁止再托管、禁止识别说话人）— [MDC CV 25.0](https://datacollective.mozillafoundation.org/datasets/cmn29rqn9016to107eniyak65)
- 自定义模型许可：FunASR（署名、保留模型名、禁诋毁、可单方修订）— [MODEL_LICENSE](https://github.com/modelscope/FunASR/blob/main/MODEL_LICENSE)；Fish Audio Community License（< 50 万美元年收入、禁对外 API、禁训练其他模型、须显示 Built with Fish Audio）— [LICENSE](https://github.com/fishaudio/fish-speech/blob/main/LICENSE)
- Apache-2.0：Qwen3-ASR — [GitHub](https://github.com/QwenLM/Qwen3-ASR)；OpenCC — [GitHub](https://github.com/BYVoid/OpenCC)；sherpa-onnx — [GitHub](https://github.com/k2-fsa/sherpa-onnx)；CosyVoice 代码（但 README 称学术用途）— [GitHub](https://github.com/FunAudioLLM/CosyVoice)
- MIT / BSD：PyCantonese — [GitHub](https://github.com/jacksonllee/pycantonese)；GPT-SoVITS — [GitHub](https://github.com/RVC-Boss/GPT-SoVITS)；CREPE — [GitHub](https://github.com/marl/crepe)；ToJyutping（BSD-2）— [LICENSE](https://github.com/CanCLID/ToJyutping/blob/main/LICENSE)
- GPL：Parselmouth（GPL-3.0+）— [GitHub](https://github.com/YannickJadoul/Parselmouth)；Cifu 频率词表（GPL-3.0）— [awesome-cantonese-nlp](https://github.com/CanCLID/awesome-cantonese-nlp)；edge-tts 库（GPL-3.0 文件）且服务条款风险 — [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/2088770/are-opensource-edge-tts-free-for-commercial-use)
- Forvo：2019 年底起为限制性自定义许可 — [Wikipedia: Forvo](https://en.wikipedia.org/wiki/Forvo)；Tatoeba 音频按录音者授权，许多为 CC BY-NC — [Tatoeba wiki](https://en.wiki.tatoeba.org/history/show-diff-between/132/2295/2313)

### Inferences
- **商用推荐栈（示意）**：
  - 字/词发音：LSHK 字表 + rime-cantonese-upstream（CC BY 4.0，App 内「关于/致谢」页署名）
  - 英文释义：CC-Canto/CC-CEDICT（BY-SA 3.0，将词典数据作为独立可下载数据并以 BY-SA 公开其改动）；若要粤语释义例句 → 与香港辞书有限公司（words.hk）洽谈商业授权
  - 分词/G2P：PyCantonese（MIT）、ToJyutping（BSD-2）、OpenCC（Apache-2.0）
  - ASR：Qwen3-ASR（Apache-2.0）服务器端；端侧 sherpa-onnx + SenseVoice（需评估 FunASR 许可）或 Qwen3-ASR-0.6B
  - TTS：Azure / Google / Polly 云服务（按量付费，条款清晰）；离线选 GPT-SoVITS（MIT，自训练音色时需取得配音员授权）
  - 声调评估：CREPE（MIT）+ 强制对齐 + 自建声调分类；Azure zh-HK 发音评估作补充
  - LLM：闭源 API 生成 + 人工审校；或自托管 CantoneseLLM v2（确认基座许可）
- 「训练数据授权」与「产品分发」需区分：用 NC 数据（MDCC、words.hk）训练的模型是否可商用在法律上存在争议，保守做法是商用模型只用 CC0/CC BY/自有数据训练。
- BY-SA 词典数据一旦与自有词典「融合」成单一数据库，整体可能需以 BY-SA 发布；建议在架构上分层存储并保留来源字段。

### Gaps
- words.hk 商业授权的价格/流程未知；分析页 public domain 与 NC 授权的适用范围差异需直接向 words.hk 确认。
- WenetSpeech-Yue、CosyVoice2-Yue、CantoneseLLM v2、Qwen3-ASR 权重卡的最终授权文本未能在 HF 上直接核实。
- 用 CC0 Common Voice 训练的模型在 MDC「禁止再分享」条款下的商业使用边界（条款原文未能打开）。
- 本文不构成法律意见；关键授权（BY-SA 传染范围、FunASR 许可、Fish 许可）建议由律师审阅。
