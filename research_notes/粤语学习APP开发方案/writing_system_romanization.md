# 粤语书写系统（粤语白话文）与罗马化方案：面向“会读简体中文、会说普通话＋赣语”学习者的 APP 设计调研笔记

> 调研时间：2026-10-24。调研环境限制：本会话的出站代理拦截了 Wikipedia、words.hk、lshk.org、CUHK、jyutping.net 等大部分站点，WebFetch 也无法解析域名，只有 GitHub（raw 文件和 git clone）能直接访问，而且网络搜索额度在调研中途用完了。所以本笔记的一手材料主要来自 GitHub 上的开源项目源文件：jyutping.org 网站源码、rime-cantonese、LSHK 字表、CanCLID 工具、Cifu 词频库。另有约 30 次网络搜索的结果摘要，以及作者对 Cifu 语料和 OpenCC 的实测统计（脚本可复现，见各条说明）。引用 Wikipedia 的条目来自搜索摘要，没有读过全文，引用时已注明。

## 1. 什么是粤语白话文？它与标准书面语（SWC）是什么关系，实际在哪些场合使用？

### Takeaway
香港、澳门、广州一带是典型的双层语言（diglossia）：正式书写用以普通话为基础的标准书面语（SWC，即“书面语”），粤语白话文主要出现在社交媒体、论坛（LIHKG）、小报娱乐版、网络小说、广告文案、部分电影和儿童片字幕等非正式场合。用户在两者之间常常不加标注地混用，形成“官粤混杂”的连续体。主流电视剧和流媒体字幕仍以 SWC 为主。

### Cited Findings
- 粤语白话文是作为非正式交流手段发展起来的；大多数正式书面交流里，粤语使用者仍须用标准书面语甚至文言，因为其他方言的使用者可能看不懂粤语白话文。— [Wikipedia: Written Cantonese（搜索摘要）](https://en.wikipedia.org/wiki/Written_Cantonese)
- Lau、Lau、To（LREC-COLING 2024 EURALI 研讨会）指出：粤语白话文的社会语言地位较低，用户在粤文和 SWC 之间“互换使用而缺乏语言标注”，两者常被一并归入“Traditional Chinese”标签。论文因此提出用关键字串和引号标记做规则分类，分出粤文、SWC 和中间变体。— [Lau et al. 2024, ACL Anthology](https://aclanthology.org/2024.eurali-1.4)；[canto-filter README 中的论文摘要](https://github.com/CanCLID/canto-filter)
- CanCLID 的分类器把文本分成四类：`cantonese`（纯粤文，如“你喺邊度”）、`mandarin`（纯官话文，如“你在哪裏”）、`mixed`（官粤混杂，如“是咁的”）、`neutral`（无特征，如“去學校讀書”）。可见大量日常句子本身并无粤/官区别标记。— [CanCLID/canto-filter README](https://github.com/CanCLID/canto-filter)
- Lau 在 BYU 的主题演讲摘要中说，粤语在香港非正式书面交流中越来越流行，但缺乏标准化书写系统；虽有逐渐趋同，变异仍然显著，编纂词典时必须考虑。— [WICL7 keynote abstracts](https://wicl7.byu.edu/abstracts-of-keynote-talks)
- Snow（2008，JAPC）认为，香港的粤语白话文已在相当程度上获得标准语的许多属性（社会功能扩大、威望上升），但并非全部。（2008 年旧资料）— [Snow 2008, Journal of Asian Pacific Communication](https://www.jbe-platform.com/content/journals/10.1075/japc.18.2.05sno)
- Snow 的专著《Cantonese as Written Language》（HKU Press 2004）把粤语书写追溯到晚明的歌本文字，再到当代香港出版物。他认为书写粤语的增多是长期历史过程的自然结果，不是“语文水平下降”。— [Duke Scholars: Cantonese as Written Language](https://scholars.duke.edu/publication/1447677)；[HKU CLEAR Don Snow seminar flyer](https://www.fe.hku.hk/clear/doc/Don%20Snow%20seminar%20flyer.pdf)
- 二手资料：一篇书评称 Snow 书末附录分析了 14 篇粤文文本，显性粤语成分占比在约 3%–36% 之间，平均约 17%。（博客转述，未核对原书）— [Language Log: Is this authentic Cantonese?](https://languagelog.ldc.upenn.edu/nll/?p=62681)
- CantoBlog（2020）称：多数近年 DVD、电视剧和流媒体默认用 SWC 字幕；带粤语字幕的主要是 80–90 年代港片的某些版本，今天不少儿童片也用粤文字幕。香港的口语粤文通常限于社交媒体帖子、小报报道、网络小说等特定平台和体裁。（2020 年资料）— [CantoBlog](https://blog.cantoblog.com/2020/01/how-to-start-learning-cantonese-after.html)
- Wikipedia 的 LIHKG 条目把该论坛的语言标为“Written Cantonese”。— [Wikipedia: LIHKG（搜索摘要）](https://en.wikipedia.org/wiki/LIHKG)
- 2016 年 TVB 一个普通话新闻节目用了简体字幕，引发香港观众不满；TVB 回应说该频道全天大部分时间播粤语节目，仍用繁体字幕。这件事涉及字体，不涉及字幕的语体（SWC 还是粤文）。（2016 年资料）— [HKFP 2016-02-23](https://hongkongfp.com/2016/02/23/viewers-outraged-as-simplified-chinese-subtitles-appear-on-tvb-news-programme/)
- 苹果日报 1995–2021 年在香港出版（已停刊）。本次检索没有找到它使用粤语白话文的具体资料。— [Wikipedia: Apple Daily（搜索摘要）](https://en.wikipedia.org/wiki/Apple_Daily)
- 广告用例：一篇营销文章分析了香港的士上的一则粤语广告，它利用“對手”的双关，作者认为保留“嘅”字能让乘客几秒内读懂双关。文章还提到广州杜蕾斯在公交车尾投放过粤语谐音广告；该文标题写“系”、正文写“係”，可见网络粤语用字并不统一。— [数英网 digitaling](https://www.digitaling.com/articles/1388232.html)
- 广州：Jing Yan 调查了 116 名广州双语者，发现认同“广州人”身份的受访者对粤语白话文（VWC）态度更正面，这种身份认同支持 VWC 在词汇、量词、句法上与 SWC 拉开距离。— [Yan, WICL 论文 PDF](https://www.ling.ohio-state.edu/wicl/doc/2B_03_yan.pdf)
- Lau 有专章讨论“意识形态驱动的粤语白话书写分化”（Routledge《The Politics of Language in Hong Kong》2024，页 19–42）。canto-filter 的分类准则即以该章描述的粤文书写体为基础（本次未能读到全文）。— [CanCLID/canto-filter README 的引用信息](https://github.com/CanCLID/canto-filter)；[EdUHK Dr LAU Chaak Ming](https://www.eduhk.hk/fhm/research-and-knowledge-transfer/fhm-researchers/fhm-researchers-dr-lau-chaak-ming)

### Inferences
- 对 APP 的含义：学习者要同时掌握两种读法。一是“读 SWC 文本时用粤音念”，即字幕、新闻、正式文本的文读；二是“读写粤语白话文”，即聊天、论坛、广告。课程可以分两条线：SWC 粤读（学习者已认识全部汉字，只需学音）和粤文白话（要新学几十个方言字和语法词）。
- 因为真实粤文处在“官粤混杂”的连续体上，阅读材料应按 canto-filter 的四类给出难度标签。也可以直接用 `cantonesedetect` 或 `canto-filter` 给 UGC 和语料自动打标。
- 视频类素材（TVB 剧集、流媒体）字幕多为 SWC，与实际对白不一致，正好可做“听粤语、看书面语”的听写或对照练习。粤文字幕素材要从老港片和儿童片里找。

### Gaps
- 苹果日报、东方日报、漫画（如《老夫子》、港漫）、报章专栏使用粤文的具体比例和体例：搜索没有找到可引用的资料，Wikipedia 和学术全文也无法访问。
- 广州当下使用粤文的场景（微博、微信公众号、地铁或商业广告、本地节目字幕）缺少系统资料；只找到零散的广告例子。
- Snow 书中“平均 17% 显性粤语成分”只是二手转述。

## 2. 学习者必须掌握的核心粤语字：大约多少个高频方言字能覆盖大部分日常粤文？有哪些频率表？

### Takeaway
核心粤语专用字非常集中。按作者对 Cifu 词频库“Written”语料的统计，只要约 13 个方言专用字就占全部方言专用字出现次数的 90%，约 16 个占 95%，约 23 个占 99%。把“唔”和常用句末语气词也算进来，约 18 个占 90%，约 31 个占 99%。在书面语料里，这类字合计约占全部汉字的 7%–10%；在口语转写里约占 13%–18%。所以 APP 的“方言字”教学清单控制在 50–60 个字内就够了，词汇层面再加 300–500 个粤语特有词。

### Cited Findings
- CanCLID 的粤文分类器把以下字定为“粤语独有字”（`CANTO_UNIQUE_CHAR`），可直接作为核心字清单：嘅嗰啲咗佢喺咁噉冇哋畀嚟諗惗乜嘢瞓睇餸𨋢摷嚿嚡嘥嗮啱喐逳噏攞𥄫攰癐冚孻冧𡃁嚫跣𨃩瀡氹凼嬲孭黐唞㪗埞忟𢛴踎脷厾；手部动词 𢳂揾搵揦揈捹撳㩒掟揼抌揸㩧𢫏擳；粗口字 撚閪𨳍𨳊𨶙𨳒；语气词 㗎𠺢喎噃啩𠿪啫嗱。另有“粤语独有词”正则：唔係/唔得/唔會…、點樣/點解、琴日/聽日、而家/依家、真係/即係、邊度/邊個、仲有、返工、傾偈、屋企、隔籬、求其等。— [CanCLID/canto-filter judge.py](https://github.com/CanCLID/canto-filter/blob/main/cantofilter/judge.py)
- 同一文件中“官话独有”的标记是 這哪您們唄咱啥甭她、還是、～兒；“官话特征字”是 那是的他它看吧沒麼么些了卻説說吃弄也、而已（“在不把”因已大量融入粤语而不算）。这份清单本身就说明了普通话母语者写粤语最容易“露馅”的字。— [canto-filter judge.py](https://github.com/CanCLID/canto-filter/blob/main/cantofilter/judge.py)
- Cifu 是香港粤语词频库（Lai & Winterstein, LREC 2020），每个词条给出汉字、粤拼，以及在 Written、Spoken adult、Spoken child、Child-directed 四种语体中的出现次数和每百万频率，另附笔画数和邻域密度，GPL-3.0 许可。— [gwinterstein/Cifu](https://github.com/gwinterstein/Cifu)；[LREC 2020 论文页](https://aclanthology.org/2020.lrec-1.375)
- 作者用 Cifu v1 数据自行统计（把词频展开到字频；脚本思路：词频×词内每个字累加）：
  - Written 语料：约 337 万字次，4,542 个不同字。canto-filter 的“严格”粤语独有字共 41 个出现，合计占全部字次的 6.67%。按频次排序，前 3 个（佢、嘅、咁）占这些字出现次数的 50%，前 9 个占 80%，前 13 个占 90%，前 16 个占 95%，前 23 个占 99%。前 20 个依次为：佢 1.51%、嘅 1.13%、咁 1.07%、咗 0.37%、啲 0.31%、嚟 0.27%、冇 0.27%、睇 0.25%、諗 0.20%、喺 0.19%、哋 0.19%、嗰 0.15%、乜 0.13%、搵 0.12%、嘢 0.11%、喎 0.08%、啱 0.07%、撚 0.06%、瞓 0.03%、啫 0.03%（百分比为占全部字次）。— 数据来源 [Cifu v1](https://github.com/gwinterstein/Cifu)
  - 扩展集（再加 唔、咩、咋、囉、咯、嘞、喇、吖、嘛、啦、呀、俾 等）：共占 Written 字次的 10.16%；前 12 个占该集合出现次数的 80%，前 18 个占 90%，前 22 个占 95%，前 31 个占 99%。“唔”是最常见的单字（1.61%）。— 数据来源 [Cifu v1](https://github.com/gwinterstein/Cifu)
  - SpokenAdult（成人口语转写，约 40.6 万字次）：严格集占 13.23%，扩展集占 18.32%。前 14 个占严格集出现次数的 90%；排序为 嘅、咁、佢、嗰、啲、喺、噉、冇、咗、哋、嘢、喎、㗎、睇……— 数据来源 [Cifu v1](https://github.com/gwinterstein/Cifu)
  - 总字量：Written 语料中，最高频的 392 个字覆盖 80% 字次，766 个覆盖 90%，1,176 个覆盖 95%，1,735 个覆盖 98%（口语分别为 231/472/752/1,176）。— 数据来源 [Cifu v1](https://github.com/gwinterstein/Cifu)
- 网文整理的“315 个粤语字”，附读音、粤拼、释义和例句（科普性质，非官方）；该文主张“给”字应写“畀”不写“俾”。— [搜狐文章](https://www.sohu.com/a/285823237_355825)
- 其他可用的频率或语料资源：香港粤语语料库 HKCanCor、马来西亚粤语语料库、早期粤语口语文献与标注语料库（科大）、香港二十世纪中期语料库（教大）、LIHKG 爬虫、粤文维基百科 dump、PyCantonese 等。— [CanCLID/awesome-cantonese-nlp](https://github.com/CanCLID/awesome-cantonese-nlp)

### Inferences
- 建议 APP 的“方言字 Tier 1”约 25 个：佢 嘅 咁/噉 咗 啲 嚟 冇 睇 諗 喺 哋 嗰 乜 搵 嘢 啱 瞓 畀 唔 咩 㗎 喎 啦 喇 嘞。学会这些即可覆盖粤文中九成以上的方言字出现次数。Tier 2 是剩下约 30 个（攞 嬲 攰 揸 撳 黐 嘥 冚 氹 踎 冧 孭 脷 餸 𨋢 𡃁 啩 噃 啫 嗱 咋 囉 咯 吖……）。粗口字（撚等）在 Written 语料中频率不低（0.06%），APP 应决定是“识别不教写”还是完全过滤。
- 难点不在方言字本身，而在约 300–500 个粤语特有词和语法词，例如 而家、琴日、點解、邊度、仲、返工、屋企，以及“动词＋咗/緊/過/晒/埋/返/親/住”等体标记。词汇层建议以 canto-filter 的 `CANTO_UNIQUE_WORD` 和 Cifu 高频词为种子。
- 学习者已认识约 3,000 个 SWC 常用字，Cifu Written 语料中约 1,200 个字就覆盖 95%。所以对学习者来说，“认字”几乎不是问题，问题在于每个已知字的粤音（尤其是文白异读、变调）和少数方言字。

### Gaps
- Cifu 论文说明“Written”语料来源于何处（论坛、报刊或其他），但全文无法访问。从其中大量出现“既/左/野/黎”等网络写法看，很可能是网络或非正式文本（推断，未证实）。
- Cifu 去掉了拉丁字母（全库只剩“T恤”一个含字母的词），因此无法用它统计“D”“OK”等字母词的频率。
- 没有找到 words.hk 官方字频表，也没有找到“HKCanCor 字频”的现成发表数据。

## 3. 本字之争、网络“错别字”和异写、口字旁惯例、Unicode/HKSCS 缺字与字体渲染、英语夹杂与字母词

### Takeaway
粤文没有官方正字法。社区（CanCLID/jyutping.org、rime-cantonese、words.hk）事实上采用“一字一语素”的推荐用字：嘅不写既、咗不写左、嘢不写野、嚟不写黎、係/喺分开、噉/咁分开、畀不写俾/比。但真实网络文本里“错别字”比例很高（作者统计：既约占 ge3 的 37%，左约占 zo2 的 56%，野约占 je5 的 45%，俾约占“给”的 80%，gam2 几乎一律写成“咁”）。核心粤字大多在 Unicode 基本区（BMP），但一批字在 Ext-A/Ext-B 甚至更后的扩展区，GB2312 和 Big5 都缺；字体缺字会显示为方块（“豆腐”）。

### Cited Findings
- **推荐用字表（jyutping.org《粵文常見錯別字》）**，按粤拼列出“常见混用写法→规范”：
  - bei2：比/畀/俾。比＝比较，畀＝给予、被动，俾＝罕用，用于译名（俾斯麥）。
  - dei6：地/哋。哋只用于复数人称（我哋、佢哋），地另有 dei2 用于“神神地”。
  - di1/dit1：Dd/滴/尐/啲 → 一律写“啲”。
  - fan3：瞓（睡）/訓（训练）。
  - faan1：翻（越过、攀爬）/返（返回、前往、回复体，返工、整返好佢）/番（次数、西方的）。
  - gam2/gam3：噉（gam2，代词，“噉樣”）/咁（gam3，副词，“咁犀利”）。
  - gau6：舊（古老）/嚿（量词“块”，亦作䊆）。
  - ge3：嘅（属格，我嘅書）/既（gei3，既然）/噶（got3，拟声）。
  - go2：嗰（远指，嗰個人）/個（go3）/果（gwo2，水果）。
  - haa5：吓（语气词 haa2）/下（haa5 次数，“睇下”“試下”）。
  - hai6：係（谓语）/系（系列）/喺（hai2，位于，我喺度）/繫（联繫）。
  - je5：嘢（名词，东西）/野（形容词，野外）。
  - lai4：嚟（過嚟）/來（loi4，將來）/黎（姓、黎明）。
  - mou5：冇（mou5）/無（mou4）。
  - nam2：諗（想；惗是异体，不推荐）/冧（lam3，倒塌）。
  - sai2：使（使錢、唔使）/駛（駕駛）/洗（清洁；“使錢”≠“洗錢”）。
  - saai3：晒（助词“全部”，食晒飯）/曬（曬太陽）/嗮（不推荐）/哂（can2，哂笑）。
  - zo2：咗（完成体）/左（方向）/唨（不推荐）。
  - zyu6/zoek3：住（袋住先）/着（着衫）/著（著作）。
  
  该页还说明：粤文一般用传统汉字，因此没有简化字合并问题。— [jyutping.org《粵文常見錯別字》（源文件）](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/typo.mdx)；[网站页面](https://jyutping.org/blog/typo/)
- **语气词推荐用字（jyutping.org《粵語語氣詞》）**：原则是“从分不从合”、一字一语素。同音节的第 1、3、4 声视为独立语素；第 2、5、6 声视为变调，推荐用标点表示，例如 gaa2 写“㗎？”而不用“𠿪”。aa/a 开音节语气词不区分（laa3＝la3）。“了”只读 liu5，不推荐作句末语气词；粤语没有“吧”。例：吖 aa1、啊 aa3、呀 aa4、𡅅 aak3、噃 bo3、𠺢/咖 gaa1、㗎 gaa3、嘎 gaa4……— [jyutping.org《粵語語氣詞》（源文件）](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/particles.mdx)
- rime-cantonese 的词条用字原则：粤语没有官方指导用字标准，项目自定一套方案，由当下大众习惯修订而来。核心是“一个汉字对应一个语素”。例如助词 saai3 统一用“晒”，码表里只有“食晒飯”“啱晒你”，不收“食曬飯/食嗮飯”。— [rime-cantonese Wiki：詞條用字原則](https://github.com/rime/rime-cantonese/wiki/%E8%A9%9E%E6%A2%9D%E7%94%A8%E5%AD%97%E5%8E%9F%E5%89%87)
- CanCLID 的粤文错别字修正器把错别字分成两类。“规则错别字”可直接正则替换，例如 `on9` 永远替换成“戇鳩”。“上下文错别字”要用 PyCantonese 分词和词性标注后判断。分类器默认输入采用推荐用字：“畀本書我”判为 cantonese，“比本書我”却判为 neutral。— [CanCLID/typo-corrector](https://github.com/CanCLID/typo-corrector)；[canto-filter README](https://github.com/CanCLID/canto-filter)
- **真实网络文本中的异写比例**（作者对 Cifu v1 词级频次的统计，Written / SpokenAdult）：
  - 嘅 37,996 / 7,809，独立词“既”22,453 / 1，“既然”另计 396 / 12。“既”约占 ge3 书写的 37%。
  - 咗 12,388 / 2,220，独立词“左”15,733 / 357（含“左边”义，但“左手”另计 254）。粗算“左”约占 zo2 的 56%。
  - 嘢 3,783 / 2,016，“野”3,056 / 2，约 45%。
  - 嚟 7,248 / 714，“黎”2,385 / 3，约 25%；“過黎”539 次。
  - 畀 930 / 407，俾 3,708 / 418，“俾”约占 80%。
  - 噉 1 / 2,599，咁 33,862 / 5,478；“噉樣”0 / 681，“咁樣”2,006 / 703。书面语料中 gam2 几乎全写成“咁”。
  - 係 55,120 / 15,318，系 66 / 3。香港书写几乎不用“系”代“係”。
  - 嗰 4,928 / 5,413，果 903 / 14，“果個”366 / 0。
  
  口语转写语料基本遵循规范用字，书面（网络）语料则大量使用同音替代字。— 数据来源 [Cifu v1](https://github.com/gwinterstein/Cifu)
- **字母、外来词与书写混合**：Bauer 在 CUHK 的论文指出，粤语吸收英语词主要靠音译，写法有全汉字、全英文字母和混合三种，原因是粤语有书面形式但从未正式标准化。— [Bauer, Journal of Chinese Linguistics monograph 24](https://www.cuhk.edu.hk/journal/jcl/jcl/mono_ser/24/24_18.html)。2000 年《Written Language & Literacy》一文讨论港人用“语音借用”（phonetic loan）和形声造字来弥合口语与书面标准的差距，借音来源可以是标准汉语或英语。— [Benjamins: WLL 3.2](https://benjamins.com/catalog/wll.3.2.02li)。Li & Wong（2024，Multilingua）研究了非正式粤文中的日、韩借词：韩语词多用汉字音译、舍弃字义；日语汉字词一般直接沿用汉字、忽略日语读音。作者认为书写混合是粤文保持活力的关键之一。— [Li & Wong 2024, Multilingua](https://doi.org/10.1515/multi-2023-0094)；[PolyU 研究成果页](https://www.polyu.edu.hk/fh/research/research-output/journal-articles/phonetic-loan-graphic-borrowing-and-script-mixing/)
- “Dd”被 jyutping.org 列为“啲”的非规范写法（说明字母 D/d 代“啲”很常见）。— [jyutping.org typo 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/typo.mdx)
- **Unicode 位置与编码覆盖**（作者用 Python `unicodedata` 和标准编解码器实测；`big5hkscs` 为 Python 内置的 Big5-HKSCS 编解码器）：
  - 位于 BMP 基本区（URO）：佢嘅咗冇喺嘢啲睇嚟唔乜嗰咁噉畀俾攰氹冚嬲揾搵哋諗瞓攞啱嘥踎冧喎噃啩嚿撳揸黐脷餸瀡孭唞嘞咩囉咯吖嗱。
  - 位于 Ext-A：㗎 U+35CE、㩒 U+3A52。
  - 位于 Ext-B：𠮶 U+20BB6、𨋢 U+282E2（升降机）、𡃁 U+210C1、𠺢 U+20EA2、𠿪 U+20FEA。
  - GB2312 不含 嘅 咗 喺 嘢 啲 嚟 哋 啱 冚 攰 氹 瞓 等；GBK 含全部 URO 字，但不含 Ext-A/Ext-B 字；GB18030 全覆盖。
  - 原版 Big5 不含 嘅 咗 喺 嘢 啲 嚟 哋 瞓 攞 啱 攰 氹 冚 嘞 吖 揸 等，这些都在 Big5-HKSCS 中；𠮶 不在 Big5-HKSCS 中。— 实测依据：码位与区块对照 [Unicode Unihan / CJK 区块图表](https://www.unicode.org/charts/unihan.html)；编码覆盖用 [Python 标准编解码器 gb2312/gbk/gb18030/big5/big5hkscs](https://docs.python.org/3/library/codecs.html#standard-encodings) 实测；码位与 [LSHK 電腦用漢字粵語拼音表 list.tsv（含 UCODE 栏）](https://github.com/lshk-org/jyutping-table) 交叉验证
- HKSCS-2008 是最后一个全部字符都有 Big5 编码的版本，共 5,009 字（含 4,568 个汉字），是 Big5 之外的增补。HKSCS-2016 于 2017 年 5 月公布，比 2008 版多 23 个汉字和 1 个符号。香港正从 Big5-HKSCS 迁移到 Unicode 子集（当时称“Hong Kong Character Set”）；澳门 MSCS 沿用了 HKSCS-2008 并加入几乎全部 HKSCS-2016 新字。— [Wikipedia: HKSCS（搜索摘要）](https://en.wikipedia.org/wiki/Hong_Kong_Supplementary_Character_Set)；[IRG N2247 HKSAR Activity Report](https://unicode.org/irg/docs/n2247-HKSARActivityReport.pdf)；[IRGN2430 feedback](https://appsrv.cse.cuhk.edu.hk/~irg/irg/irg53/IRGN2430EisoFeedback.pdf)
- 仍有粤语字在申请编码或横向扩展：2021 年 HKSAR 申请为 U+2D25D、U+2BB37 两个字做横向扩展，它们见于一本香港粤语词典，用作拟声词（经码位计算，分别在 Ext-F 和 Ext-E）。2024 年有个人提案向 UAX #45 提交 3 个粤语教育用字。— [L2/21-159 (IRGN2507)](https://www.unicode.org/L2/L2021/21159-irgn2507-hksar-ext.pdf)；[L2/24-056](https://www.unicode.org/L2/L2024/24056-cantonese-ed-uax45.pdf)
- 字体渲染问题：Ext-B 字在不少字体中缺字，会显示为空方框；如果字体声称支持补充平面但缺个别字，系统仍会选用该字体。（2008 年资料）— [Michael Kaplan 博客存档](https://archives.miloush.net/michkap/archive/2008/11/19/9116067.html)。Ubuntu 的 bug 报告称，HKSCS 所需的约 1,700 个 Ext-B 字中，文泉驿字体只有 1 个字形，AR PL UMing HK 是唯一接近完整支持 HKSCS 的字体（旧资料）。— [Launchpad bug 1270647](https://launchpad.net/bugs/1270647)；[Fedora 邮件列表 2011](https://lists.fedoraproject.org/pipermail/users/2011-June/399016.html)
- jyutping.org 网站自带“昭源黑體 HK 粵拼”字体（Chiron Hei HK，带粤拼注音字形），并用 glyphhanger + pyftsubset 按页面实际用字做子集化，因为完整字体太大。— [CanCLID/jyutping.org README](https://github.com/CanCLID/jyutping.org)

### Inferences
- APP 的“标准答案”应采用 jyutping.org/rime-cantonese/words.hk 一系的推荐用字。判分时应宽容常见异写（既/左/野/黎/俾/D/咁代噉/系代係），标为“可接受的网络写法”并提示规范写法。读真实 UGC 时，可用 typo-corrector 的规则先做规范化，再交给 NLP 或词典查询。
- 为避免“豆腐块”，APP 应内置覆盖 HKSCS 和 Ext-B 的 Web 字体，或按课程实际用字子集化的字体，做法同 jyutping.org。不要依赖系统字体，特别是中国大陆安卓机型和老旧系统。服务端、数据库一律用 UTF-8/utf8mb4，因为 Ext-B 字是 4 字节。
- 对 APP 而言，码位和编码问题主要出现在“简体模式”（见下一节：OpenCC 会把“嗰”转成 Ext-B 的 𠮶）和少数语气词、名词（㗎、𠺢、𨋢、𡃁）上。

### Gaps
- words.hk 的官方编辑和用字指引（如 base/hoifong 页）无法访问，未能直接引用。
- 没有拿到 iOS、Android、各大陆厂商系统字体对 Ext-B 粤字的实测覆盖数据，需要在真机上测试。
- “check下”“OK唔OK”这类英语夹杂的频率，没有找到可引用的统计；Cifu 去掉了字母，也无法测量。

## 4. 繁体与简体：简体粤文的表现，以及对面向简体读者的 APP 的含义

### Takeaway
多数粤语专用字没有简化形式，简繁同形（嘅 咗 啲 喺 嘢 嚟 佢 冇 噉 咁 畀 睇 瞓 啱 攞 嬲 等）。但 OpenCC 默认的繁→简转换会把 係→系、嗰→𠮶（Ext-B）、喎→㖞（Ext-A）、餸→𩠌（Ext-B）、𨋢→䢂、搵→揾、撳→揿、諗→谂、囉→啰。其中“系”无法逆转回“係”。另外，简化字的同音合并和声旁替换是按普通话读音设计的，会给粤音学习制造歧义和误读。建议：以繁体（OpenCC 标准字形）为内部规范存储，界面提供“简体显示”给简体读者过渡，同时逐步引导他们认读繁体粤字。

### Cited Findings
- jyutping.org《點解簡化字唔適合用嚟寫粵語？》列出两类问题：
  - (1) 按普通话同音合并、但粤语不同音的字：卜(蔔 baak6/卜 buk1)、斗(鬥 dau3/斗 dau2)、复(複 fuk1/復 fuk6)、里(里 lei5/裏 leoi5)、表(表 biu2/錶 biu1)、干(干 gon1/乾 gon1 kin4/幹 gon3)、仆(仆 puk1/僕 buk6)、松(鬆 sung1/松 cung4)、坛(罈 taam4/壇 taan4)、系(係繫系 hai6/喺 hai2)、吓(嚇 haak3/吓 haa2)、须(須 seoi1/鬚 sou1)、涌(涌 cung1/湧 jung2)、郁(鬱 wat1/郁 juk1)、脏(髒 zong1/臟 zong6)、折(折 zit3/摺 zip3)、症(癥 zing1/症 zing3)、只(只 zi2/隻 zek3)。
  - (2) 按普通话读音换声旁、导致与粤音冲突的字（部分）：坝(壩 baa3)、补(補 bou2)、纤歼忏(纖殲 cim1/懺 caam3)、担胆(擔 daam1/膽 daam2，韵尾 m→n)、灯(燈 dang1)、柜(櫃 gwai6)、吓(嚇 haak3)、舰(艦 laam6，易误读为“見”)、邻(鄰 leon4)、扑朴(撲樸 pok3)、烛触独(燭觸 zuk1/獨 duk6)。
  
  该站提供简化字粤文版，但“仅作初学者过渡适应用”，主张学习者以传统汉字书写粤文为最终目标，理由是传统汉字更容易习得粤语发音规律。— [jyutping.org simplified 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/simplified.mdx)；[页面](https://jyutping.org/blog/simplified/)
- OpenCC 实测（作者使用 opencc-python-reimplemented 包的 `t2s` 和 `s2hk` 配置，它是 OpenCC 词典的 Python 移植，版本可能落后于官方 OpenCC）：
  - `t2s`：“佢哋係咪喺度食緊飯？”→“佢哋系咪喺度食紧饭？”；“嗰個人好鬆弛，搵咗份工”→“𠮶个人好松弛，揾咗份工”；“噉樣做唔啱㗎”→“噉样做唔啱㗎”；“你畀啲嘢我睇吓”保持不变；“喎”→“㖞”(U+359E)，“餸”→“𩠌”(U+2980C)，“𨋢”→“䢂”(U+4882)，“撳”→“揿”，“諗”→“谂”，“囉”→“啰”。
  - `s2hk`：“佢系咪喺度食紧饭？”→“佢系咪喺度食緊飯？”（系没有恢复成係）；“𠮶个人揾咗份工”→“嗰個人揾咗份工”（𠮶 恢复成 嗰，但 揾 未转为 搵）。
  
  可见简→繁是有损的。— 工具：[OpenCC](https://github.com/BYVoid/OpenCC)；[opencc-python-reimplemented (PyPI)](https://pypi.org/project/opencc-python-reimplemented/)
- rime-cantonese 的单字收录全部传统字及其异体，不收大陆简化字形。词条只收 OpenCC 传统字形（如收“因爲”不收“因為”），因为 OpenCC 的传统字形“从分不从合”，比港台字形保存了更多信息，可以自动转换到台湾、香港、大陆、日本字形。方案仍支持切换为“香港传统汉字、台湾传统汉字、大陆简化汉字”输出。— [rime-cantonese Wiki：本方案字形選用原則](https://github.com/rime/rime-cantonese/wiki/%E6%9C%AC%E6%96%B9%E6%A1%88%E5%AD%97%E5%BD%A2%E9%81%B8%E7%94%A8%E5%8E%9F%E5%89%87)；[rime-cantonese README](https://github.com/rime/rime-cantonese)
- rime-cantonese-upstream 的说明：字表一般不应有简化字，但部分简体中文用到的字属于俗字或古字（如“国”“网”“碍”），这类可标为“罕见”异体；所有词条都标准化为 OpenCC 字形。— [CanCLID/rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream)
- canto-filter 默认所有输入为传统汉字，简体文本要先用 OpenCC 转成繁体再分类。— [canto-filter README](https://github.com/CanCLID/canto-filter)
- 广州网络粤文中“系/係”混用，同一篇文章标题写“系”、正文写“係”。— [数英网](https://www.digitaling.com/articles/1388232.html)

### Inferences
- **存储**：以 OpenCC 繁体字形为唯一权威文本，每个音节带粤拼。简体显示是派生视图，不要反过来以简体为源，因为“系”会同时对应 係/系/繫/喺，无法自动恢复。
- **简体视图的自定义规则**（覆盖 OpenCC 默认）：
  - “嗰”保持“嗰”，不转 Ext-B 的“𠮶”；“餸”保持“餸”；“喎”保持“喎”，避免 Ext-A/Ext-B 渲染风险和陌生字形。
  - “係”可设为保留“係”，或转“系”并在注音层显示 hai6，以区别“喺”hai2。前者更利于学习，后者更贴近广州简体网文；可做成设置项。
  - 学习者已熟悉简体，建议默认“简体字形＋繁体粤字保留”的混合方案。课程中段开始引入繁体（香港字形）阅读，最终能读 LIHKG/香港社媒原文。
- **混淆点提示**：遇到“表/錶”“松/鬆”“只/隻”“里/裏”“系/係/喺”“干/乾/幹”等合并字时，在简体视图里给出繁体和不同读音提示。“舰 laam6”“担 daam1”等声旁误导字要列为发音易错卡片，正好对应普通话母语者“-m 韵尾丢失”的问题。

### Gaps
- 没有找到广州或大陆“简体粤文”的系统性用字调查（例如简体用户写“系”与“係”、“咁”与“噉”、“𠮶”与“嗰”的比例）。Lau 的“法域间正字法比较”研究只查到题目，未读到正文。— [EdUHK repository](https://repository.eduhk.hk/en/publications/the-influence-of-orthographic-repertoire-on-written-cantonese-acr/)
- 官方 OpenCC（C++，最新词典）对“嗰”“喎”“餸”的处理是否与 Python 移植版一致，未验证。

## 5. 规范参考资料：政府和教育局资源、LSHK 字表、词典、words.hk、CUHK 审音配词字库、何文汇正音运动，以及 APP 该教哪种“标准音”

### Takeaway
粤语没有官方的书写和拼音标准。事实上的读音参考是：LSHK《電腦用漢字粵語拼音表》（2.9 万多字，CC BY 4.0，GitHub 维护）、CUHK《粵語審音配詞字庫》、《廣州話正音字典》、《常用字廣州話讀音表》，以及 words.hk 粤典（词条和例句）。何文汇 1980 年代起推动“正音”，主张依据《广韵》等反切，学界分为“从切”与“从众”两派。APP 宜采用“从众”的当代香港和广州通行音（LSHK/words.hk 一系，区分 n/l、ng-/∅-、gw/g），并在少数字上注明“正音派读法”。

### Cited Findings
- LSHK《電腦用漢字粵語拼音表》为 29,000 多个汉字标注粤拼，包括 ISO/IEC 10646-1:2000 的 27,484 个字和 HKSCS-2001 的 4,384 个字。原表只在 iso10646hk.net 提供 PDF；后来 LSHK 粤拼小组征得原作者同意接手维护，在 GitHub 以 TSV/JSON 发布，许可为 CC BY 4.0。鸣谢陆勤教授、张群显博士（理大）。— [lshk-org/jyutping-table](https://github.com/lshk-org/jyutping-table)
- 该表 list.tsv 当前约 38,305 行（一字多音则多行），字段为 CH/UCODE/JP/INIT/FINL/TONE/DESC。示例：嘅 ge3/ge2/koi3；啲 di1/dit1/di4/ti4；係 hai6/hai2；佢 keoi5/heoi5；咁 gam2/gam3；畀 bei2/bei3。— [lshk-org/jyutping-table list.tsv](https://github.com/lshk-org/jyutping-table)（作者下载计数）
- rime-cantonese 的读音资料来源包括：LSHK 字音表、粤音小镜、《廣州話正音字典》（2004）、《常用字廣州話讀音表》（1992 版数据）、CUHK《粵語審音配詞字庫》、LSHK《電腦用漢字粵語拼音表》、HKSCS；词组来源包括 LSHK 粤拼词表（教大 corpus.eduhk.hk/JPwordlist）、粤典 words.hk、开放词典、CC-Canto、《A Dictionary of Cantonese Slang》。— [rime-cantonese Wiki：資料來源](https://github.com/rime/rime-cantonese/wiki/%E8%B3%87%E6%96%99%E4%BE%86%E6%BA%90)
- rime-cantonese-upstream 的单字表以 LSHK 粤拼表为主，参考 Unihan kCantonese、粤语审音配词字库、《廣州話正音字典》；词条以粤典、冚唪唥粤文、《實用廣州話分類詞典》、《廣州話詞典》等为主。每个读音标注常见度（预设、常用、罕见、弃用）、本调或变调、文读或白读，例如“-aak/-ak 互换、ek/ik 归白读；浊上归去的阳去读音归文读，阳上读音归白读；高升变调归变调”。— [CanCLID/rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream)
- 收录原则：区分 ng-/∅- 声母，只收原读音，不收混淆后的读音；不收懒音（如 n-/l- 混淆）；词语标音记实际发音（变调后的音）。— [rime-cantonese-upstream README](https://github.com/CanCLID/rime-cantonese-upstream)
- 正音运动 1980 年代始于香港，由何文汇主导，其规范性引发不少争议。— [Wikipedia: Proper Cantonese pronunciation（搜索摘要）](https://en.wikipedia.org/wiki/Proper_Cantonese_pronunciation)。何文汇 2021 年回应“矯枉過正”的批评：课堂需要标准，查字典找正音很重要，教诗词不能读错；日常闲谈能沟通即可。他也承认语音会变，50 年前的粤语与今天已有差别。他认为粤语的“四声九调”与中古音相近，有助于演绎诗词平仄。— [恒生大学新闻稿 2021-12](https://www.hsu.edu.hk/wp-content/uploads/2021/12/%E6%96%B0%E8%81%9E%E7%A8%BF-%E6%8E%A8%E5%8B%95%E7%B2%B5%E8%AA%9E%E6%AD%A3%E9%9F%B3%E9%9D%9E%E7%9F%AF%E6%9E%89%E9%81%8E%E6%AD%A3%E3%80%80%E5%8A%A9%E6%89%BF%E5%82%B3%E4%B8%AD%E5%9C%8B%E5%8F%A4%E5%85%B8%E8%A9%A9%E8%A9%9E%E6%96%87%E5%8C%96%E3%80%80%E4%BD%95%E6%96%87%E5%8C%AF%EF%BC%9A%E7%B6%AD%E6%8C%81%E8%AA%9E%E6%96%87%E6%B0%B4%E5%B9%B3%E3%80%80%E6%94%BF%E5%BA%9C%E8%B2%AC%E7%84%A1%E6%97%81%E8%B2%B8.pdf)；[何文汇 CUHK 中國語文通訊文章](https://www.cuhk.edu.hk/ics/clrc/crcl_51/ho.pdf)
- 港大一篇论文摘要指出，学界分歧的核心是“從切”（依传统反切）还是“從眾”（依社会通行读音）。— [HKU Scholars Hub 10722/228896](https://hub.hku.hk/handle/10722/228896)
- 中大语言中心（ILC）有“Proper Cantonese Pronunciation”教材和粤语正音资源（PDF，2018），面向课程教学。— [CUHK ILC ProperCanPronunciation_20180503.pdf](https://ilc.cuhk.edu.hk/workshop/Chinese/Cantonese/Romanization/inc/files/ProperCanPronunciation_20180503.pdf)；[CUHK ILC Lang_CAN_Pronunciation](https://ilc.cuhk.edu.hk/EN/CHResources/Lang_CAN_Pronunciation.aspx)
- 懒音：年轻港人倾向把 n- 并入 l-、丢失 ng-，例如“你”nei5 读成 lei5；广州也有类似现象但程度较轻。（学生论文级来源，可靠性一般）— [bild-lida.ca 文章](https://bild-lida.ca/educationalsociolinguistics/uncategorized/hong-kong-cantonese-vs-guangzhou-cantonese/)。jyutping.org 也指出：很多广州人已分不清 ng- 与零声母，例如把“我”读成 o5，规范为 ngo5。— [jyutping.org《一文掌握粵拼》源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/learn/yue/index.mdx)
- 《香港小學學習字詞表》由前教统局 2003 年委托香港理工大学中文及双语学系研究后重新编订（据第三方 App 介绍）。有论文批评其字形与日常所见和台湾标准不同。— [App Store：香港小學學習字詞表](https://apps.apple.com/us/app/id1213923119)；[EdUHK 论文：字詞表所收字形研究](https://repository.eduhk.hk/en/publications/%E9%A6%99%E6%B8%AF%E5%B0%8F%E5%AD%B8%E5%AD%B8%E7%BF%92%E5%AD%97%E8%A9%9E%E8%A1%A8%E6%89%80%E6%94%B6%E5%AD%97%E5%BD%A2%E7%A0%94%E7%A9%B6%E5%85%BC%E8%AB%96%E9%A6%99%E6%B8%AF%E5%AF%AB%E5%AD%97%E6%95%99%E5%AD%B8%E4%B9%8B%E5%9B%B0%E5%A2%83/)

### Inferences
- **APP 的“标准音”建议**：
  - (a) 发音示范用区分 n/l、ng-/∅-、gw-/g-（如 廣 gwong2）、kw-/k- 的“规范音”，这也是 LSHK 字表、rime-upstream、words.hk 录音的默认做法。
  - (b) 字音取“从众”的常用音（pron_rank＝预设或常用），对“正音派”读法（何文汇式依反切的读法）只作注释，不作为考核标准。
  - (c) 听力材料有意加入懒音样本（l-代 n-、零声母代 ng-、g-代 gw-），让学习者能听懂真实口语。
  - (d) 判分时对学习者的口语输出可以宽容懒音（如 ASR 评测开“模糊音”），但书写和拼音答题按规范拼写判。
- 数据栈建议：单字音用 LSHK jyutping-table（CC BY 4.0）＋rime-cantonese-upstream char.csv（CC BY 4.0，带文白和变调标注）；词条和例句视许可选用 words.hk、CC-Canto；自动注音可用 ToJyutping 或 PyCantonese。— 许可信息见 [lshk-org/jyutping-table](https://github.com/lshk-org/jyutping-table)、[rime-cantonese README](https://github.com/rime/rime-cantonese)、[awesome-cantonese-nlp](https://github.com/CanCLID/awesome-cantonese-nlp)

### Gaps
- words.hk 的收词量、编辑指引和许可条款，以及 CUHK 审音配词字库所用的拼音系统和收字数，都因站点被拦截未能核实。
- 《香港小學學習字詞表》是否附粤拼（还是其他注音），未能从官方来源确认。
- 饶秉才《广州话方言词典》《广州话正音字典》所用拼音（应为广州话拼音方案，但未核实）。
- 何文汇正音的具体争议字例（如“時間”的“間”gaan1/gaan3 等）未找到可引用的来源，不能列具体字例。

## 6. 罗马化方案比较（粤拼、耶鲁、教院式、刘锡祥、广州话拼音方案、IPA）及推荐

### Takeaway
推荐 LSHK 粤拼（Jyutping）作为 APP 的唯一主拼音。理由：它是 1993 年起的事实标准，纯 ASCII，可直接打字；一个符号对应一个音位；主流开源输入法、字表和词典都用它；与汉语拼音的差异少而规律。对普通话使用者，要专门讲清 j=[j]、z/c/s=[ts tsʰ s]、aa/a、eo/oe、yu=ü、gw/kw、-m/-p/-t/-k 韵尾和 6 个调号。耶鲁可作为“读英文旧教材”的转换选项；IPA 作专家视图；广州话拼音方案因附加符号多，不适合打字。

### Cited Findings
- 五种常见方案（jyutping.org 对照表）：
  - 粤拼：LSHK 1993 年制定。
  - 广州话拼音方案（广拼）：广东省教育部门 1960 年制定，饶秉才 1980 年代修订。
  - 教院式：香港教育署语文教育学院中文系 1990 年《常用字廣州話讀音表》，数字标调；后被设计更完善的粤拼取代。
  - 耶鲁：黄伯飞与 Gerald P. Kok 制定，常见于教外国人的教材。
  - 刘锡祥（Sidney Lau）：1972 年制定，常见于外国人教材。
  
  黄锡凌、Meyer–Wempe 等历史方案当代已基本不用。— [jyutping.org《粵拼與其他粵語拼音方案對照表》源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/table.mdx)；[页面](https://jyutping.org/blog/table/)
- 声母对照（IPA：粤拼/广拼/教院/耶鲁/刘锡祥）：[ts]：z / z,j / dz / j / j；[tsʰ]：c / c,q / ts / ch / ch；[s]：s / s,x / s / s / s；[kʷ]：gw / gu / gw / gw / gw；[kʷʰ]：kw / ku / kw / kw / kw；[j]：j / y / j / y / y。其余 b p m f d t n l g k ng h w 各方案相同。— [jyutping.org table 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/table.mdx)
- 韵母关键差异：
  - [œː]：oe / ê / oe / eu / euh。
  - [ɵy]（銳）：eoi / êu / oey / eui / ui。
  - [ɵn]（閏）：eon / ên / oen / eun / un。
  - [yː]（於）：yu / ü / y / yu / ue。
  - [ɐ] 类韵（鶯、噏、兀、扼）：粤拼 ang/ap/at/ak，广拼 eng/eb/ed/eg。
  - 广拼入声韵尾写 -b/-d/-g。
  - 口语韵 eu/em/en/ep/et 在其他方案原表中无定义。
  - oet（𠰲，呕吐拟声）在广拼、教院、耶鲁中无法表示。
  
  — [jyutping.org table 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/table.mdx)
- 声调对照（调类、调值、例字：粤拼/广拼/教院/耶鲁/刘锡祥）：
  - 阴平 53/55 分：1/1/1/à ā 或 1/1, 1°
  - 阴上 35 粉：2
  - 阴去 33 训：3
  - 阳平 21 坟：4，耶鲁 àh
  - 阳上 13 愤：5，耶鲁 áh
  - 阳去 22 份：6，耶鲁 ah
  - 上阴入 5 忽：粤拼 1，教院 7 或 1
  - 下阴入 3 发：粤拼 3，教院 8 或 3
  - 阳入 2 佛：粤拼 6，教院 9 或 6
  
  耶鲁一般用调号并在阳调元音后加 h，不便时用数字。阴平的 53 与 55 在现代粤语中一般是自由变体，粤拼、广拼、教院都记作 1，耶鲁和刘锡祥则加以区分。— [jyutping.org table 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/table.mdx)
- 粤拼只用 1–6，入声不另编号；教院式通常用 1–9，也允许入声用 1/3/6 代替 7/8/9。例如“好開心認識你”，粤拼 hou2 hoi1 sam1 jing6 sik1 nei5，教院式“識”作 sik7。— [Wikipedia: Jyutping（搜索摘要）](https://en.wikipedia.org/wiki/Jyutping)；[Wikipedia: ILE romanization（搜索摘要）](https://en.wikipedia.org/wiki/ILE_romanization_of_Cantonese)
- jyutping.org 评价拼音方案的四原则：一符号对应一音位；避免附加符号和非 ASCII 字符；符号符合大众习惯；准确反映音位系统。据此指出：
  - 耶鲁：a 既表 /a/ 又表 /ɐ/（fa“花” vs faan“翻” vs fan“分”），无法表示开音节 /ɐ/（如“嘞”）；不分 /ɵ/ 和 /œ/（统一写 eu），无法表示 /ɛːu/（掉 deu6）和 oet；用调号加 h 标调（如“蛇”sèh），而 h 又是声母。
  - 广拼：用 e ê é ü 等附加符号，难以用于输入法；z/c/s 与 j/q/x 两套重复表示同一组音位（为兼容普通话拼音）；不能表示 oet、eu。
  - 教院式：最接近粤拼，但 dz/ts 冗长，且影响输入法简拼（打 ts 是“此”的声母，还是“聽寫”两字的声母？）。
  - 港府拼音和邮政式不是严格系统，如“蔡”有 Choi/Choy/Tsoi，chun 可表“俊”也可表“珍”。
  
  — [jyutping.org《點解要用粵拼》源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/scheme.mdx)；[页面](https://jyutping.org/blog/scheme/)
- 粤拼常被批评的两点：j 表 [j]，因为 y 留给了 yu；yu 用两个字母表示单元音 [y]。jyutping.org 承认 yu 是冗余设计，并说明市面粤拼输入法基本都支持用 y 代替 yu。— [jyutping.org scheme 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/scheme.mdx)
- 粤拼方案 2018 年新增韵母 a（开音节 /ɐ/）和 oet；声调标在音节后，如 fu1 夫、fu2 虎、fu3 副、fu4 扶、fu5 妇、fu6 父。— [jyutping.org 粵拼方案页源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/jyutping/yue/index.mdx)
- **面向汉语拼音使用者的粤拼速成要点**（jyutping.org《一文掌握粵拼》，副标题“識普通話拼音，一文掌握粵拼”）：
  - 写法区别：普拼 y→粤拼 j（一 jat1、以 ji5、夜 je6）；ü→yu（豬 zyu1、村 cyun1、雨 jyu5）；gu-/ku-→gw-/kw-（廣 gwong2、狂 kwong4、昆 kwan1，但“古”仍是 gu2）；普拼 ong＝粤拼 ung（鍾 zung1），粤拼 ong 是普通话没有的 [ɔŋ]（莊 zong1）；ao→aau（考 haau2）。
  - 音系区别：aa[a] 与 a[ɐ]（三 saam1/心 sam1）；eo[ɵ] 与 oe[œ]（率 seot1/削 soek3）；e 永远是 [ɛ]（寫 se2）；i 在 ik/ing 中为 [ɪ]；ng- 与零声母对立（岸 ngon6/按 on3）；保留 -m/-n/-ng 三种鼻音韵尾和 -p/-t/-k 入声；6 个声调。
  - 阴平分化为 55（詩）和 53（私），粤拼不区分。
  
  — [jyutping.org learn 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/learn/yue/index.mdx)
- 粤拼在开源生态中是主流：rime-cantonese 只支持 LSHK 粤拼，IPA 显示版以 Bauer & Benedict《Modern Cantonese Phonology》（1997）为准，其他拼音方案以补丁形式提供（CanCLID/rime-cantonese-schemes）；LSHK 字表、Cifu、rime-upstream 全部用粤拼标音。— [rime-cantonese README](https://github.com/rime/rime-cantonese)；[Cifu](https://github.com/gwinterstein/Cifu)
- Jyut Dictionary（开源离线粤语词典）汇集 CC-CANTO、ABC Cantonese-English Dictionary、CantoDict、《廣州話正音字典》等来源，支持繁体、简体、粤拼、普拼、英文输入，可在设置里隐藏拼音或改变字的颜色。— [aaronhktan/jyut-dict README](https://github.com/aaronhktan/jyut-dict)
- 一款 Spanish 学习网站称，没有任何政府统一规定官方粤语罗马化，粤拼因便于打字已成主流；面向英语学生的教材主要是耶鲁与粤拼之争。（二手、可靠性一般）— [studycli.org](https://studycli.org/es/learn-chinese/mandarin-vs-cantonese/)

### Inferences
- **推荐粤拼的理由**（针对本学习者）：
  - (1) 它与汉语拼音共享 b/p/d/t/g/k（不送气/送气）的逻辑，z/c/s 与普拼 z/c/s 发音相近，学习者能直接迁移。
  - (2) 纯 ASCII，学习者可以用任何键盘输入答案，也能直接接入 Rime、Gboard、iOS 的粤拼键盘。
  - (3) 数据资源（LSHK 字表、rime-upstream、Cifu、words.hk）全部是粤拼，零转换成本。
  - (4) 耶鲁的调号和 h、广拼的 ê é ü，都会让打字和判分变复杂。
- **对普通话使用者的重点陷阱卡片**：
  - j＝普拼 y：jat1“一”不是“家”的 j。
  - 粤拼 z/c 覆盖了普拼 zh/ch/z/c/j/q 的字群，例如“知”zi1、“張”zoeng1、“車”ce1。
  - 粤拼 c 是送气 [tsʰ]，不是英文 /k/ 或 /s/。
  - yu 才是 ü；粤拼单写 u 是 [u]。
  - eo 与 oe 要区分。
  - aa 与 a 的长短对立，如“三 saam1 / 心 sam1”。
  - 韵尾 -m 和入声 -p/-t/-k，普通话都没有。
  - 6 个调号不对应普通话的 4 个调（普通话第一声≈粤拼 1，其余没有直接对应）。
- 若学习者的赣语保留入声和/或 -m 等韵尾，可能有助于掌握粤语入声。这是待验证的假设，见 Gaps。
- 方案转换：APP 内部存粤拼，用规则表（上面的对照表）实时转换为耶鲁、教院式或 IPA，供参考旧教材的用户使用。但练习和输入只接受粤拼（可容忍 y→yu 缩写和不带调号）。

### Gaps
- 无法访问 CantoDict、words.hk、CUHK Lexi-can、Pleco 等站点确认各自默认用哪种拼音（一般认为 words.hk 用粤拼，CantoDict 同时显示粤拼和耶鲁，但本次未核实）。
- 《广州话方言词典》等大陆出版物使用广州话拼音方案的程度，以及大陆粤语教材当前的拼音选择，没有一手资料。
- 赣语（南昌话等）入声和韵尾的保留情况，以及对粤语学习的正负迁移，没有检索到可引用的研究。

## 7. 界面中的声调表示：数字、调号、调型标记、颜色编码；现有应用怎么做

### Takeaway
主流做法是粤拼加数字调号（1–6）：ASCII、可输入、可检索。调号（耶鲁式）只存在于旧教材。新兴的学习工具在数字之上叠加颜色（声母、韵母、声调分色，或按调着色）和调型示意（五度标记 55/35/33/21/13/22）。APP 建议：默认“汉字＋上方 ruby 粤拼（带数字）”，可选开启按调着色和调型小图标；声调教学用五度音高图和“詩史試時市事”六调对比。

### Cited Findings
- 粤拼声调标在音节后（fu1–fu6）；对照表给出调值：阴平 53/55、阴上 35、阴去 33、阳平 21、阳上 13、阳去 22，入声 5/3/2（IPA 调符 ˥ ˧˥ ˧ ˨˩ ˩˧ ˨）。— [jyutping.org table 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/table.mdx)；[粵拼方案页](https://github.com/CanCLID/jyutping.org/blob/master/src/content/jyutping/yue/index.mdx)
- jyutping.org 的教学页用 HTML `<ruby>` 在汉字上方注粤拼（如 蘿<ruby>蔔<rt>baak6</rt></ruby>），并为每个例字配真人录音（录音数据来自粤典）。声调练习用 si 音节 6 调两两组合（詩史試時市事），还建议用粤语歌练声调。— [jyutping.org simplified 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/simplified.mdx)；[learn 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/learn/yue/index.mdx)
- 2023 年有港人设计“彩色粤语字体”，给粤拼各部分着色，并有标示声调位置的视觉记号，用于促进学习。— [HKFP 2023-05-16](https://hongkongfp.com/2023/05/16/hongkonger-creates-colourful-cantonese-font-to-foster-language-learning/)
- iOS 应用“Cantonese Jyutping Annotator”宣称可把声母、韵母、声调数字用三种颜色显示（商店文案，未独立验证）。— [mwm.ai 应用页](https://mwm.ai/apps/cantonese-jyutping-annotator/6757694004)
- Chrome 扩展“Mandarin + Cantonese Dictionary”（Zhongwen 分支）在汉字旁显示粤拼和普拼，但只给普拼按声调着色；有用户评论要求粤拼也按调着色，说明该功能缺失。— [Chrome Web Store](https://chrome.google.com/webstore/detail/mandarin-+-cantonese-dict/aoofnmlljjgifabglpelnbmipdfnfflk)
- Jyut Dictionary 允许用户“改变字的颜色或完全关闭”，即提供按声调着色并可配置。— [aaronhktan/jyut-dict README](https://github.com/aaronhktan/jyut-dict)
- JyutDict（Android，岭南粤音）使用“扩展粤拼”（J++）并提供声调图。— [jyutjam.org JyutDict](https://jyutjam.org/en/jyutdict-android/)
- rime-cantonese 有“聲調顯示版”和“IPA 顯示版”两个方案。— [rime-cantonese README](https://github.com/rime/rime-cantonese)
- jyutping.org 使用带粤拼注音的“昭源黑體 HK 粵拼”字体。— [CanCLID/jyutping.org README](https://github.com/CanCLID/jyutping.org)

### Inferences
- **默认显示**：汉字上方 ruby 粤拼，数字调号紧跟音节（nei5）。可切换为“只显示声调数字”“隐藏拼音（测试模式）”“IPA”。不推荐耶鲁调号作为默认，因为上去两调无调号、阳调靠 h 标记，对新手不直观，也无法打字。
- **颜色**：6 色按调类着色，入声沿用 1/3/6 的颜色，并加下划线或短横表示“短促”。颜色必须配合数字（色盲友好），并提供“只按高、中、低三层着色”的简化选项。
- **调型图标**：在每个音节旁用微型五度线（55↗35→33↘21↗13→22）展示调型；对普通话使用者，单独对比“普通话一声 55 ≈ 粤拼 1”“普通话三声 214 ≠ 粤拼 5（13）”等。
- **连读变调**：变调（如 高升变调 → 2 调）在词条上标实际读音，同时显示本调（例如“女 neoi5→neoi2”），做法同 rime-upstream“标实际发音”和“本调/变调”字段。

### Gaps
- Pleco、CantoDict、words.hk、Google 翻译粤语模式是否按调着色，以及采用哪种配色，本次无法核实。
- 没有找到比较不同声调显示方式学习效果的实证研究（例如颜色对比数字）。

## 8. 粤语输入法（Rime 粤拼、Gboard、iOS、大陆输入法）及其对 APP 内输入答案的影响

### Takeaway
粤拼输入在各平台都有成熟方案：Rime/rime-cantonese（PC 和移动端，支持不打声调、模糊音、简繁切换、普通话拼音反查），iOS 自带“粤语拼音”（Jyutping）键盘，Android 的 Gboard“粤语（香港）粤拼”，以及开源的“粤拼输入法”（iOS/macOS/Android）和 TypeDuck（Rime 编译成 WebAssembly 的网页版）。APP 内输入答案应做到：(1) 接受粤拼（可无调号，y 可代 yu，可选模糊音）；(2) 也可让学习者用系统粤拼键盘直接输入汉字；(3) 若要做内嵌键盘，可参考 TypeDuck，用 Rime 的 WASM 在端侧运行。

### Cited Findings
- rime-cantonese（CanCLID 维护，主体 CC BY 4.0，`jyut6ping3.maps` 为 ODbL）：
  - 只支持 LSHK 粤拼。
  - 输入时可省略声调；如要打声调，用 v、x、q 键：v＝阴平或上阴入（siv 诗、sikv 色），x＝阴上（six 史），q＝阴去或下阴入（siq 试、sekq 锡），vv＝阳平，xx＝阳上，qq＝阳去或阳入。
  - 默认不支持模糊音和懒音（区分 n-/l-、∅-/ng-），用户可在 schema 中取消注释，例如 `derive/^n(?!g)/l/`，让“lei hou”出“你好”。
  - 输出用字可切换：OpenCC 传统汉字（默认）、香港传统汉字、台湾传统汉字、大陆简化汉字。
  - 支持普通话（按 \`）、粤语两分、笔画、仓颉五代反查。
  - 一键安装包见 jyutping.net。
  
  — [rime/rime-cantonese README](https://github.com/rime/rime-cantonese)；[rime-cantonese Wiki Home](https://github.com/rime/rime-cantonese/wiki)
- iPhone 可以在“通用 → 键盘 → 添加新键盘 → 中文（繁体）”中启用苹果自带的“粤语拼音”（Cantonese Phonetic, Jyutping）布局；Android 的 Gboard 可在“语言 → 添加键盘 → 粤语（香港）”中选择粤拼。本次未确认 iOS 版 Gboard 是否有粤拼。— [cantonese.hk keyboard 指南](https://cantonese.hk/keyboard)
- 开源免费的“粵拼輸入法 Jyutping – Cantonese Keyboard”完全支持 LSHK 粤拼，离线运行，iOS 与 macOS 版本功能可能不同；Android 版在 F-Droid 发布，0.60.0 版新增九宫格（T9）粤拼布局。— [App Store id1509367629](https://apps.apple.com/app/id1509367629)；[F-Droid org.jyutping.jyutping](https://f-droid.org/packages/org.jyutping.jyutping/)
- TypeDuck Web 是基于 RIME 的网页粤语输入法，用 Emscripten 编译为 WebAssembly，在浏览器本地运行，数据不发往服务器。— [TypeDuck-HK/TypeDuck-Web](https://github.com/TypeDuck-HK/TypeDuck-Web)
- 粤拼输入法普遍支持用 y 代替 yu。— [jyutping.org scheme 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/scheme.mdx)
- 教院式的 dz/ts 会增加简拼重码（ts＝“此”还是“聽寫”）。这说明拼音方案的选择直接影响输入体验。— [jyutping.org scheme 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/scheme.mdx)

### Inferences
- **答题输入的三种模式**：
  - (a) 拼音作答：学习者输入粤拼字符串，判分时规范化（小写、去空格、y→yu、可选去调号）。用 LSHK/rime 字表比对，按“全对 / 调错 / 声韵错”分级反馈，“调错”应单独统计，作为声调薄弱诊断。
  - (b) 汉字作答：学习者用系统粤拼键盘或普通话拼音键盘打字。注意学习者用普通话拼音键盘打不出“嘅/咗/喺”等字（多数大陆输入法词库不收或排序很后）。所以要么内置粤字候选条（点选 Tier-1 方言字），要么引导安装 Gboard、iOS 粤拼或 Rime。
  - (c) 内嵌粤拼键盘：参考 TypeDuck 的 Rime-WASM 方案，或直接用 rime-cantonese 词库（CC BY 4.0，注意 maps 文件为 ODbL）。
- 模糊音设置应与教学目标分开：输入可以宽容 n/l、ng/∅，发音评测和拼写题则按规范判。
- 对普通话使用者，rime-cantonese 的“普通话反查”很有用：用普通话拼音查一个字的粤音，APP 可以复制这种“我知道这个字、不知道粤音”的查询路径。

### Gaps
- 搜狗、讯飞、百度等大陆主流输入法对粤拼或粤语字的支持现状（2026 年）未核实；只知道讯飞有粤语语音输入，但没有查到可引用资料。
- iOS 自带粤语键盘（速成、仓颉、笔画、粤拼）与 Gboard 各自的确切功能（是否支持声调输入、简体输出）未核实。

## 9. 香港粤语与广州粤语的差异（词汇、语音、外来词）及目标变体选择

### Takeaway
港穗粤语可以互通，差异主要在口音、个别读音和词汇：香港英语借词多、语码混用多，“懒音”更明显；广州受普通话影响更多，书写上用简体、常见“系”代“係”。考虑到学习者读简体，可能更多接触广州或大陆的粤语社群；但书面资源、字表、词典和粤文规范几乎都以香港为基准。建议 APP 以“香港和广州共同核心＋香港书写规范”为主，在词条上标注【港】【穗】差异，并让用户选择侧重。

### Cited Findings
- 香港、澳门粤语与广州粤语可以互通，在口音、发音和词汇上有些小差异。这些差异源于 1841–1997 年的英国统治，以及 1949 年后的边境封闭。— [Wikipedia: Hong Kong Cantonese（搜索摘要）](https://en.wikipedia.org/wiki/Hong_Kong_Cantonese)
- 词汇示例：香港说“巴士”，广州说“公共汽車”；广州老一辈仍说“食晏”，香港多用英语借词说法（如“lunch”）。— [cantonese.hk C1 Lesson 4: Dialect variations](https://cantonese.hk/courses/c1-mastery/dialect-variations)
- 香港粤语吸收了大量外语词汇，并形成大批本地特有词；语码混用（夹英文）是辨别港穗口音的重要特征，广州人用英文明显较少。（学生论文，可靠性一般）— [bild-lida.ca](https://bild-lida.ca/educationalsociolinguistics/uncategorized/hong-kong-cantonese-vs-guangzhou-cantonese/)
- 懒音：香港年轻一代 n→l、丢失 ng- 明显，广州也有但程度较轻（同上来源）。广州很多人分不清 ng- 和零声母（如“我”读 o5）。— [bild-lida.ca](https://bild-lida.ca/educationalsociolinguistics/uncategorized/hong-kong-cantonese-vs-guangzhou-cantonese/)；[jyutping.org learn 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/learn/yue/index.mdx)
- 阴平调在广州话中分化为 55 和 53 两个调值（如“詩”55、“私”53，“私人”与“詩人”听感不同），现代一般视为自由变体，粤拼不区分。— [jyutping.org learn 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/learn/yue/index.mdx)；[jyutping.org table 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/table.mdx)
- rime-cantonese-upstream 的词条来源同时包括香港（粤典、《A Dictionary of Cantonese Slang》）和广州（《實用廣州話分類詞典》《廣州話詞典》《地道廣州話用語》）资料。说明开源词库是“港穗合一”的。— [CanCLID/rime-cantonese-upstream](https://github.com/CanCLID/rime-cantonese-upstream)
- jyutping.org 推荐用字表中“部分字有多于一个建议用字，主要考虑到粤语区不同地域用字习惯”。— [jyutping.org typo 源文件](https://github.com/CanCLID/jyutping.org/blob/master/src/content/blog/yue/typo.mdx)
- 广州的地域认同与对粤语白话文的正面态度相关（116 人调查）。— [Yan, WICL 论文](https://www.ling.ohio-state.edu/wicl/doc/2B_03_yan.pdf)

### Inferences
- **目标变体建议**：发音以 LSHK、words.hk 记录的规范音为准（港穗共同的“标准广州话”音系）；词汇以港穗共有核心词为主，并标注变体差异（如 巴士/公共汽車，以及香港常用英语借词如 的士、士多、波）；书写以香港推荐用字为“规范答案”，简体模式下允许广州习惯写法（如 系）。
- 如果学习者主要想和广东亲友或同事交流，可在设置中选“广州侧重”：例句少用英语夹杂，增加大陆语境词汇。想看港剧、读 LIHKG，则选“香港侧重”。
- 港式英语夹杂（如 check、book、OT、ok）宜作为“香港侧重”的专门词汇单元，用“英文原形＋粤语读法＋语法位置”教学，例如“check 下”中的“下”为动量补语。具体频率缺少数据，需要自建语料统计。

### Gaps
- 缺乏有学术来源的港穗词汇差异系统清单和语音差异量化研究；可用来源多为教学网站和学生论文。
- 广州年轻人粤语能力下降和普通话化对词汇的影响，本次没有找到可引用资料。
- “香港 6 调、广州 7 调”的说法只见于学生论文，没有核实，未采用。
