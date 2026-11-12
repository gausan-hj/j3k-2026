# 项目接手文档 · 感谢老师（2026 届毕业留念）

这是一个**双形态交付**的毕业感谢作品，同一个主题做了两个版本：

1. `index.html` —— **互动网页版**：点击按钮 → 信封开启 → 信纸升起、展开 → 逐句浮现感谢信。
2. `video/` —— **44 秒成片**：`感谢老师·三年.mp4`，由 HTML 场景逐帧渲染而成，配自制钢琴曲。

两版共用同一套视觉语言（暖橘落日渐变 + 金色 + 衬线中文），改动其一时应同步考虑另一版。

---

## 目录结构

```
index.html                 网页版（单文件，无构建）：取件码 → 开箱 → 奖状 / 感谢信，内置 4 语言
code.html                  老师姓名 × 取件码对照表（密码门 0102，可搜索/复制/跳转；由 _verify/gen_code_page.py 生成，
                           密码只以 djb2 哈希存在页面里，数据是 base64，明文不进源码）
index.before-i18n.html     加多语言之前的备份（确认无误后可删）
AGENTS.md                  本文件
_verify/                   验证脚本与截图（可删）
voice/                     四语 AI 配音（作者提供）
  zh voice.mp3             中文     21:22  ✅ 完整
  bm voice.mp3             马来文    8:49  ⚠️ 仅约 1/3
  en voice.mp3             英文      6:14  ⚠️ 仅约 1/5
  hl voice.mp3             泰米尔文  8:27  ⚠️ 仅约 1/4（文件名沿用作者原样）
video/
  scene.html               视频分镜（1920×1080 定尺寸，动画全部暂停，靠 seek() 定位）
  music.py                 numpy 合成 44s 钢琴伴奏 → music.wav（无外部素材依赖）
  music.wav                配音轨（44s / 44100Hz / 立体声 / 7.8MB）
  render.py                Playwright 逐帧截图 → pipe → ffmpeg 编码成片
  感谢老师·三年.mp4          成片：1920×1080 / 30fps / 44.00s / H.264 CRF18 + AAC 192k
```

---

## 取件码 → 开箱 → 奖状 / 感谢信

进入页面先看到一只箱子（`body.is-boxed` 时隐藏原本的 `.stage`）。流程：

1. **输入取件码**（箱子上的标签也会显示输入的码）→ 校验失败会抖动并提示，成功则：
2. **箱盖掀起 + 镜头向上、俯视箱内**：`.box-scene` 做 `translateY(11vh) scale(1.05)`（画面下移＝镜头往上），
   箱盖 `.box-lid` 以箱口为轴 `translateY(-100%) rotateX(30deg)` 掀到箱口上方后仰；
   箱口浮出金边 `.box-rim`，箱内是暗紫内衬 `.box-inside`（**下窄上宽的梯形 clip-path** + 顶部内阴影 = 纵深），
   **三样东西躺在箱底**（底部留一条箱底地面，才读得出"躺在里面"而不是"贴在外面"）。
3. **奖状**、**荣誉证书**（每位老师各一张，都由取件码决定）与**感谢信**（即原本那封信，动画与内容完全没改）。
   点感谢信 → 箱子退场 → 走原来的信封/信纸流程；点另外两张 → 弹出对应的证书。

- **荣誉证书**与奖状共用 `.cert-*` 结构，外层加 `.is-honour` 换成红调（红标题 + 标题下红色双线 + 红框），
  独立一份 `#honourBackdrop` / `#honourName` / `#honourSubject`，数据同样取自 `TEACHERS`（不带 `note`）。
  文案键：`itemHonour` / `honourTitle` / `honourBody` / `honourAward` / `honourClose`。
- **信纸也盖章**：`.letter-seal`（`<use href="#j3kStamp">`）放在落款之后、右对齐，
  宽 `clamp(76px,17vw,96px)`、旋转 -8° —— 就像真的在信末盖了个章。
- 箱内三件物品：`.box-items` 在 `left/right:10%`、`top:12% / bottom:42%`，`align-items:flex-end`
  让三张卡**底边对齐**、标签成一行；物品宽 `clamp(50px,11.5vw,66px)`，带 `drop-shadow` 落地阴影。
  标签用奶油色（内衬是深色）。`.box-body` 设了 `z-index:1` 自成堆叠上下文，
  否则丝带/标签（z3）会盖到内衬上。`_verify/three_items_check.py` 逐件核对是否在箱内、
  且不与取件码标签 / J3K 印章重叠。

- **名册在 `index.html` 的 `TEACHERS` 常量**（28 位老师，取件码是**随机乱数**，与名单顺序无关）：
  ```js
  '5824': { zh:'Amelia',                             // 证书上显示的名字
            latin:'Latin Name',                      // 可选，缺省时四语都用 zh
            subject:{zh:'英文 · J3C（J3 Success）', ms:'…', en:'…', ta:'…'},
            cert:{zh:'奖状正文', ms:'…', en:'…', ta:'…'},     // 每位老师各不相同
            rose:{hex:'#E91E63', zh:'玫红玫瑰', ms:'…', en:'…', ta:'…'}, // 箱内可拾取的玫瑰
            honour:{zh:'荣誉证书正文', ms:'…', en:'…', ta:'…'} } // 每位老师各不相同
  ```
  `cert` / `honour` 是**逐位老师**的四语文案（取自感谢信里真实写到的回忆），缺省时回落到
  `I18N[lang].certBody` / `honourBody` 通用文案。旧的 `NOTES` 分组表已废弃（改为逐人文案）。
  取件码任意字符串，查询时忽略大小写与空格/连字符。`subject` 缺省时奖状自动省略那一行。
- **个人链接**：`index.html?code=1362` 会预填并**自动开箱**（方便发给指定老师）；纯手工输入也行。
  最近一次成功的码存 `localStorage.letterCode`，下次打开自动预填（不自动开箱）。
- **奖状按钮**：箱子打开后右下角出现奖状按钮（`#certBtn`），信件关闭后依然能再次打开奖状。
- 奖状支持**打印 / 保存**（`window.print()`），`@media print` 只输出奖状本身。
- **奖状是横版**（`width:min(920px,96vw)`，正文 `max-width:42em` 居中以免行长过宽），
  右下角盖 **J3K 印章**：SVG 双圈 + 顶部弧形年份 `2024 — 2026` + 中心 `J3K` + 五角星。
- **关掉信封后会回到箱子**：`closeBtn` 流程的最后一步执行
  `body.classList.add('is-boxed')` + 移除 `.box-stage.is-gone`，箱子保持**打开**状态，
  奖状与感谢信都能再点开（焦点落在奖状按钮上）。`_verify/seal_flow_check.py` 已验证二次开启。
- **J3K 印章三处共用一套 SVG 图形**（`<body>` 开头的 `.seal-defs` 里定义两个 `<symbol>`）：
  - `#j3kStamp` 红色印泥章（朱砂红 `#bf3a2b`）：**扭索纹玫瑰**（24 + 12 片椭圆旋转叠加，`opacity .1 / .09`）
    + 外粗圈 + 细圈 + 极细点圈（`stroke-dasharray=".5 3.1"`）+ 内细圈 + 弧形年份 `2024 — 2026`
    （`<textPath>` 沿 `#sealArc`，r=35.5）+ 中心 `J3K` + 细线菱形装饰
    + 中心柔和留白（`#sealBlot` 模糊白圆，让字从纹样里跳出来）+ 几团模糊淡红做"墨迹不匀"；
    圈线另叠一层 `feTurbulence` 位移滤镜做毛边。**滤镜只加在装饰层与留白上**，
    万一某浏览器不支持，基础图形仍然完整显示。
  - `#j3kWax` 金色火漆印（信封用）：`feTurbulence` 位移做出不规则蜡边 + 高光 + 内圈 + 奶油色 `J3K`。
  - 三处引用：奖状 `.cert-seal`、箱子 `.box-seal`、信封 `.seal`（`<use href="#…">`）。
  - 图形内部一律用 SVG 表现属性而非 CSS 类，避免 `<use>` 影子树里的样式匹配问题；
    文字用 `dy="0.35em"` 垂直居中（比 `dominant-baseline` 兼容性更好）。
  - 印章与语言无关，`certSeal` 这个 i18n 键已不再使用（保留在字典里以便还原）。
- 箱子开箱后，标签抬头由「取件码」变为「已签收」/`boxStamp`（`openBox()` 与 `applyLang()` 同步）。
- **打印**：`@page{ size:A4 landscape }`，`@media print` 只输出奖状。已用 `page.pdf(prefer_css_page_size=True)`
  验证 MediaBox = 841.9 × 595（A4 横向）。
  选择器用 `.cert-backdrop.is-open`，因为页面上现在有**两个**证书 overlay（奖状 + 荣誉证书），
  只打印当前打开的那一张。
- 称呼格式按语言：中文 `陈丽华 老师`；马来文 `Cikgu Tan Lee Hua`；英文 `Teacher …`；泰米尔文 `ஆசிரியர் …`。
- **落款统一带班级名 J3K**（`certIssuer`，奖状与荣誉证书共用）：中文 `2026 届 J3K全体学生 敬上`、
  马来文 `Daripada semua anak murid J3K, Kelas 2026`、英文 `From all the students of J3K, Class of 2026`、
  泰米尔文 `J3K அனைத்து மாணவர்களும், 2026 தொகுதி`。
- 验证脚本：`python _verify/box_check.py`（空码/错码 → 正确码开箱 → 奖状 → 换语言 → 打印 → 转信封 → 个人链接）。



---

## 多语言（index.html）

左上角胶囊按钮可在 **中文 / Bahasa Melayu / English / தமிழ்（泰米尔文）** 间即时切换。
切换会同步更新：所有带 `data-i18n` 的节点、`<html lang>`、浏览器标签标题、音符按钮的无障碍标签，
并把**已展开**的信纸按新文案重新量高居中。选择存 `localStorage.letterLang`，下次打开沿用。

- 文案表：`index.html` 脚本里的 `I18N` 对象，四个键 `zh` / `ms` / `en` / `ta`。
- **正文是数组**：`body: ['第一段', '第二段', …]`，段数可随语言不同；`applyLang()` 每次切换都重建
  `#letterBody` 里的 `<p>`。**四语全部完成，各 110 段**（作者原文，逐字保留，含各自原文的引号与破折号风格）。
- 实测信纸内滚动高度：中文 15,127px ｜ 英文 16,002px ｜ 马来文 19,023px ｜ 泰米尔文 25,333px。
  四语**横向溢出均为 0**（无文字被裁切）；纵向滚动是长信的正常表现，泰米尔文最长（需滚 24,541px）。
- 单段结构：`title`（标签页标题）、`headline`（信封上方献词）、`letterTitle`（信纸标题）、
  `letterSub`（标题下的金色副题）、`sign`（落款）、`cta` / `close`（按钮）、`scrollHint`（滚动提示）、
  `musicPlay` / `musicPause` / `musicNone`（音符按钮无障碍文案）。
- 加第 5 种语言：往 `I18N` 加一份字典 → 往 `.lang-switch` 加一个
  `<button class="lang-btn" data-lang="xx">` → 若字形不是中文/拉丁，再往 Google Fonts 链接补字体。
  （例：换成印地文 हिन्दी 只需把 `ta` 换成 `hi` 文案 + 补 `Noto+Sans+Devanagari`。）
- 版式适配：`html[lang="ms"|"en"|"ta"]` 下单独调了正文字号与行高；`finalRect()` 给非中文放宽信纸与高度上限。
- **长信阅读**：信纸高度上限为视口 82%（中文）/ 88%（其他语言），超出部分在信纸内滚动
  （`.letter.is-revealed{overflow-y:auto}`）。`.scroll-hint` 提示条由 `placeHint()`/`syncHint()` 定位，
  只在"确实还有内容没读完"时出现，用户一开始滚动就淡出。
- 验证脚本：`_verify/lang_check.py`（四语截图 + 溢出像素，正常为 0）、
  `_verify/letter_check.py`（长信段落数 / 滚动 / 提示 / 关闭回程）。
- 视频版 `scene.html` **仍是纯中文**，尚未多语言。

---

## 朗读 / 语音（voice/）

右下角「朗读这封信」按钮播放当前语言的 AI 配音：播放中按钮下缘显示金色进度条，
再点一次暂停，第三次继续；切换语言会停止播放并重新指向该语言的音频。

- **按钮只在开箱后才出现**：`#readBtn` 初始带 `hidden`（CSS 另有 `.read-btn[hidden]{display:none}`，
  因为 `.read-btn` 是 flex），`openBox()` 里与奖状按钮一起解除隐藏。
- **背景音乐按钮已按作者要求删除**：`#musicToggle`、`<audio id="bgm">`、
  `hasMusic()` / `pauseMusicForReading()` 及相关 JS 全部移除；
  `musicPlay` / `musicPause` / `musicNone` 三个 i18n 键保留但已无人引用。
  右下角现在只有两个控件：朗读（胶囊，`right: 18px + 76px`）与奖状（圆钮，`right: 18px`）。

- **先弹温馨提醒**：首次点朗读会弹窗「本音频由人工智能（AI）生成……请以原始文字内容为准」，
  点「我明白了，开始播放」才开始。勾选「不再显示此提醒」会写入 `localStorage.letterVoiceNotice`，
  之后直接播放。弹窗支持 Esc / 点背景关闭。
- 音频**按需加载**（`VOICE` 映射表 + `new Audio()`），打开页面不会预下载几十 MB。
- 文案键（四语齐备）：`readLabel` / `readPause` / `readNoAudio` / `noticeTitle` / `noticeBody` /
  `noticeSkip` / `noticeCancel` / `noticeOk`。
- 验证脚本：`python _verify/read_check.py`（弹窗 → 播放 → 换语言 → 互斥 → 不再提示）。

### 音频文件现状（⚠️ 重要）

| 文件 | 对应语言 | 时长 | 与正文的匹配 |
|---|---|---|---|
| `voice/zh voice.mp3` | 中文 | 21:22 | ✅ **完整**（正文 6,931 字 ≈ 5.4 字/秒，147 处长停顿与 110 段相符） |
| `voice/bm voice.mp3` | 马来文 | 8:49 | ⚠️ 仅约 **1/3**（正文 4,104 词；8:49 读完需 ~465 词/分钟，不可能） |
| `voice/en voice.mp3` | 英文 | 6:14 | ⚠️ 仅约 **1/5**（正文 4,355 词；6:14 读完需 ~698 词/分钟，不可能） |
| `voice/hl voice.mp3` | 泰米尔文（作者确认） | 8:27 | ⚠️ 约 **1/4**（泰米尔文正文 29,966 字，完整读完估计需 35–40 分钟） |

推断：TTS 工具（ElevenLabs）可能存在单次字数上限（约 8,000–10,000 字）——中文 6,931 字刚好没超，所以完整；
其余三种被截断。**需作者确认后重新生成（建议分段生成再拼接）**。

文件名 `hl` 与泰米尔文对不上，作者已确认内容就是泰米尔文，故 `VOICE.ta = 'voice/hl%20voice.mp3'`。
文件的 ID3 里只带 C2PA 凭证（author: Eleven Labs Inc.），**没有语言/音色信息**，无法从文件本身核验语种；
若日后发现它其实是印地语，改 `VOICE` 一行即可。

---

## 重新生成

```powershell
cd video

# 1. 重新合成背景音乐（可选，会覆盖 music.wav）
python music.py

# 2. 抽帧预览（改 scene.html 后先看这个，秒级）
python render.py preview 3 10 29 38     # → preview_3.jpg 等

# 3. 渲染全片（约 1320 帧，需要先有 music.wav）
python render.py                        # → 感谢老师·三年.mp4
```

网页版不需要构建：直接双击 `index.html`，或本地起个静态服务打开。
注意页面字体走 Google Fonts（Noto Serif SC / Noto Sans SC / Noto Serif Tamil / Noto Sans Tamil / Ma Shan Zheng），**离线会掉字形**。

---

## 视频时间轴（改片时的对齐依据）

| 时间 | 内容 |
|---|---|
| 1.2s / 2.1s / 3.2s | `2024 — 2026` / `三年`（金色"年"字弹入）/ `谢谢您，一路相伴` |
| 4.5s | 第一架纸飞机飞过 |
| 7.8s–26.8s | 底部时间轴（2024 → 2025 → 2026 三节点，16s 走完） |
| 8.6s / 14.6s / 20.6s | 第一年「初遇」/ 第二年「成长」/ 第三年「启程」（图标描边 + 逐字浮现） |
| 26.0s | 教学楼窗户亮灯 |
| 27.4s | `三年，<计数> 个日子`，27.9–30.1s 缓动数到 **1095** |
| 29.3s / 29.8s / 30.5s | `谢谢您，老师` / 金色下划线 / `把最好的年华，留给了我们的成长` |
| 36.0–36.8s | 十顶学士帽抛起 |
| 37.0s | 光芒放射；37.2s 起 70 片彩纸爆开 |
| 37.4s / 39.0s / 39.9s | `老师，您辛苦了` / `—— 您永远的学生们` / `2026 届 · 毕业留念` |
| 42.2s–44.0s | 白幕淡出收尾 |

渲染确定性：`scene.html` 里所有动画被 `animation-play-state:paused` 冻结，
由 `window.seek(t)` 统一设置 `currentTime`，因此同一 `t` 每次截图完全一致（粒子/飘落用固定种子 `seed=7`）。

---

## 环境

- Python 3.12（`C:\Users\sansan\AppData\Local\Programs\Python\Python312\python.exe`）
- 依赖：`playwright 1.61.0`、`numpy 2.4.6`、`imageio-ffmpeg 0.6.0`（ffmpeg 二进制由该包自带，无需另装）
- **限制**：在 DSH `workspace-write` 沙箱下 Playwright 无法启动浏览器子进程（命名管道被拦，`WinError 5`）。
  跑 `render.py` 需要放宽到完全访问权限，或在沙箱外执行。

---

## 已知问题 / 待办

1. ~~网页版没有背景音乐~~ —— **音乐按钮已按作者要求删除**（见「朗读 / 语音」一节），
   若要重新加回背景音乐，需同时恢复按钮、`<audio id="bgm">` 与互斥逻辑。
2. **`1095` 是写死的**：`scene.html` 里 `1095 * (1-(1-k)^3)` 与 `2024—2026` 相配；
   若改年份，记得同步这个数字和封面日期。
3. **两版文案需要人工保持一致**：网页版信件正文与视频字幕目前是各自独立的文本，
   改文案要两边都改（无共享数据源）。
4. `render.py` 无断点续渲：中途中断要从头来，改片时优先用 `preview` 定帧。
5. **长信在手机上要滑很多屏**（泰米尔文纵向约 24,500px）。若想提升阅读仪式感，可考虑按
   「初一 / 初二 / 初三 / 致各科老师 / 结尾」分节翻页，一屏一节，读完一节点一下继续。
6. 视频版 `scene.html` 仍是纯中文；若要四语版本，字幕/文案需另做，成片需重渲（`python render.py`）。
7. **朗读四语已全部接通**：`VOICE.ta` 指向 `voice/hl voice.mp3`（作者确认该文件内容是泰米尔文，仅文件名写作 `hl`）。
   若日后发现它其实是印地语，只需改 `VOICE` 一行。
8. **只有中文音频是完整的**：马来文（约 1/3）、英文（约 1/5）、泰米尔文（约 1/4）都只读了正文开头一段。
   要让老师听完整封信，需要用同样的文本分段生成后拼接（我可以帮拼）。
9. **名册是真实名单（28 位老师，取件码为随机乱数）**，来自 `.txt`（作者提供的老师名单）+ 后续补充
   （Teacher Subatra / 张雪玲 / Anita / 任志康 / Amelia），清单见 `取件码一览.md`。
   **每位老师的奖状 / 荣誉证书文案都不一样**（`teacher.cert` / `teacher.honour`，各自四语），
   内容取自感谢信里真实写到的回忆（“小天使”变“魔丸”、联合国会议厅、冷笑话开源地、
   数学课那场争执与眼泪、初二历史课、班导那句“我们是一个班的”……）。
   此外每位老师还有一朵**玫瑰**（`rose:{hex,zh,ms,en,ta}`，箱内可拾取，见 `.rose` / `#roseView`）。
   标了 `bear:true` 的老师（詹晋沣 / 林淑娟 / 林素铭 / Nabilah / Rizan，即教满三年的）箱里还有一只**泰迪熊**
   （`#bearItem` / `.box-bear`，点开进 `#bearView`），颜色同样取该老师的玫瑰色。
   **小熊是真的 3D**：用 `_verify/bear_cpu_render.py`（numpy SDF raymarch）把模型渲染成
   12 个角度（−30°…+30°）的序列帧，拼成 `assets/bear_sprite.webp`（2880×282，约 55 KB），
   页面里用 `background-position` + `steps(11)` 逐帧播放＝它在真的转身（`@keyframes bearSpin`）。
   - 3D 数学（球/椭球/胶囊 + `smin` 平滑融合 + 兰伯特/高光/边缘光）在 `bear_cpu_render.py` 的 `scene()`
     与 `render()` 里，**改造型就改那里再重渲**：`python _verify/bear_sprite.py "#2196F3" bear_blue`。
   - 序列帧只烘焙"蓝系基准色"，其它颜色靠 `.b-sprite` 的 `filter: hue-rotate()` 换算
     （`fillBear()` 里用 `hueOf()` 算出色相差，基准 hue≈207°）。
   - **金颈圈是独立元素 `.b-collar`**，不能用 `.b-sprite::after`：伪元素会跟着 `hue-rotate` 一起染色，
     红色老师的颈圈会变成蓝色。颈圈还要 `translateZ(.05em)`，否则被 sprite 的 `translateZ` 挡在后面。
   - 图加载不到时自动回退到 SVG 版小熊（`buildBearSVG()`，`.b-svg`）—— 所以单独拷 `index.html` 也不会坏。
   - `#bearView` 里**不要**再给 `.rose-spin` 套 3D 旋转（序列帧自己会转，而平面容器转起来就是纸片）。
   `bearFromBox()` 用 `.b-sprite` 的包围盒算起飞起点，改尺寸要同步改那里的 `2 * em`。
   带班级的老师：Nabilah J3B、Rizan J3A、Shula J3C、Anita J3C、任志康 J3K、
   Vatsala / Lim Wen Xin / Shanthini J3B（J3 WISDOM）、Pavithra / Subatra / Amelia J3C（J3 Success）。
   ⚠️ **名册目前以 `index.html` 为准、直接手改** —— 因为 `_verify/gen_roster.py` 还不认识 `rose` 字段，
   一跑就会把玫瑰整片抹掉（脚本顶部已加警告）。要恢复用脚本生成，需先给它补上 rose。
   **中文老师的罗马拼音尚未提供** —— 目前马来文/英文/泰米尔文证书显示中文名（如 `Cikgu 林淑娟`）；
   补 `latin` 字段即可。`python _verify/content_check.py` 逐个码核对各老师内容是否两两不同。

---

## 视觉规范（改稿直接复用）

```
落日渐变  #fff3e4 → #fbd2a8 → #e99a9e → #a06382 → #5c3f5c
金色      #cf9a4c      浅金 #f1d9a6      纸色 #fbf4e6 / #f2e6cd
墨色      #5b4636      次级文字 #8a7461    剪影 #42324a
字体      正文 Noto Serif SC ｜ 手写签名 Ma Shan Zheng ｜ 说明 Noto Sans SC
          泰米尔文 Noto Serif Tamil / Noto Sans Tamil（已加入字体栈，按字形自动回退）
```
