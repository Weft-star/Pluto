# CHANGELOG / 改动记录

按版本记录「做了什么、为什么、涉及哪里」。文件顶部注释块是给模型看的摘要，这里是给人看的历史。

---

## v7 — 2026-10-18 · P21 信号档案与知识框

**本轮核心**：「探索获得信息」。归档即解锁，信息本身就是奖励——不设门槛、不阻断探索、不新增操作。

### P21.1 知识框（通用组件，P20 后续复用）

- `showKnowledgeBox(title, text, quote)` → `#knowledgeBox`：fixed 中央偏上、宽 ≤340px、像素边框，三段（标签行 = 类型+编号 / 描述段 / 评论段 glow 单行）。
- 关闭：点击任意处（window pointerdown capture，**不阻断游戏输入**——那次点击照常发射声呐）、Esc、8s 自动淡出。不捕获键盘、不遮挡 `.ops`。
- reduced-motion 无淡入。

### P21.2 档案池

- 11 条（WRK 3 / RUN 2 / ORG 2 / ECH 2 / VNT 2），描述+评论文案按工单原文。
- 每个信号点由 seed（`sig.phase` + id 哈希）决定绑定条目；归档成功 → 若该条未解锁则解锁并弹框；同一信号不重复弹（`archiveShown`），不同信号可能绑同一条（先到先解锁）。
- 持久化 `'abyssal.sigarchives.v1'`。

### P21.3 图鉴存档

- 覆盖层新增第四标签「信号档案」（与图鉴/DS-06/设置并列）：按类型分组，已解锁三段全文（描述 + 引号评论），未解锁「◇ 尚未解锁」，顶部「已解锁 N / 11」，全解锁底部「深海里还有更多被藏起来的事。」

### P21.4 边界

- 不影响玩法数值、不参与 sectionArchived、与 DS-06/P20 独立；不新增色板、不新增 canvas 层、不改信号绘制。

---

## v6 — 2026-10-18 · P19.0 壳体应力 + P19 意外事件池 + P15 瞬时冲击最小版

**本轮核心**：「环境主动发生一件事」——不惩罚、不丢进度、不阻断探索。事件参数由当前状态推导。

### P19.0 壳体应力（氛围版）

- `state.hullStress` 0–100；目标 = `(depth-600)/800×100`，实际值 0.8/s 惯性趋近；事件开始时 +25 瞬时冲击，之后回落。
- 只驱动三件事：HUD 应力条颜色（<40 current / 40–75 glow / >75 alert）；应力 >60 时 fxLP 向 4000Hz 压低 + drone detune ±3 音分；应力 >70 时光锥边缘密度 +8%、每 3–5s 一条 0.1s 的 alert 应力线。
- HUD：telemetry 电量条下方同尺寸 `HULL` 条。
- 不强制上浮、不限制操作、不掉电量、不影响信号可见性。

### P19 调度器 + 八事件

- **不安度** = 深度×0.35 + 应力×0.25 + 静默×0.15 + 间隔×0.25；触发按 `不安度×0.0045/s` 累积掷骰（深水期望 ~4 分钟，浅水 ~15 分钟），硬冷却 300–480s。
- 事件按 zone 允许表 + 权重选择，全流程上限 6 次、同事件不重复、不连续同名（`'abyssal.encounters.v1'`）。
- 八事件全部「前兆→过程→稳定」，时长由物理量推导：
  1. **海水断崖**（depth 200–1395）：跃层距离按 zone 梯度（100–600m）×0.8–1.2，终速 15+应力×0.07，阻力趋近；深度真实推移，P11 三联动自动发生，eventLP 按速度压到 6000Hz，35Hz 轰鸣 + 0.8s 金属 tick；到平衡层自然停。
  2. **上升流**（depth>300）：对称反向，气泡噪声 highpass 800Hz + 上行 accent。
  3. **横向洋流**（depth>100）：信号 baseX 真实漂移（0.004–0.01/s）、背景横向 ±1px、光锥 0.05rad/s 推偏（玩家可覆盖）；depth 不变。
  4. **浊流**（ABYSSAL/HADAL depth>400）：turbidity 1→1.6–2.5 连续变化，lightExposureAt range 实时除以浓度，粒子 ×浓度（≤120）。
  5. **声学阴影**（非静默、非 PHOTIC、depth>200）：阴影带缓慢漂移，带内声呐 hold/glow ×(1-strength×0.6)，雷达对应方位出空洞；depth 变化 >150m 或到时消散。
  6. **生物迁徙**：zone 生态密度 48/38/28/10 个剪影沿梯度通过，纯演出不可点击。
  7. **热液喷发**（仅 ABYSSAL）：26 个 ember 点受重力回落，温度读数 +4–10℃ 实时变化，全场信号 hold ≥1.2。
  8. **远场震动**（ABYSSAL/HADAL depth>600）：canvas transform 波形抖动（非随机跳），雷达两个不可点击假回波；信号实际位置不变。
- 事件结束：冷却 300–480s、60s 异常静默、bed/LP 全部恢复。事件期间不触发 P15。

### P15 瞬时冲击（最小版，3 个）

- 「体感事件」开关默认开（`'abyssal.sensory.v1'`）；会话内 ≤5 次、间隔 ≥180s、同类不连续（工单未给持久化 key，计数为单次会话——见文件头注释）。
- **仪表冻结**：前兆 2s（SIGNAL 读数漂移 + 状态灯快闪）→ 冲击 1.2s（HUD 冻结、雷达停扫）→ 余波 4s（深度→温度→压力→信号→时钟逐项恢复，未恢复项显示乱码）→「传感器总线重置。自检正常。」
- **壳体呻吟**：前兆 3s（hum 压到 0.02）→ 冲击 1.4s（28+41Hz sawtooth 过 lowpass 60Hz，峰值 0.14；应力线抖 0.3s）→ 余波 5s → 日志记录压强。仅 depth>600，>1200 权重×2。
- **次声冲击**：前兆 2.5s（粒子静止）→ 冲击 0.8s（19+23Hz 过 lowpass 40Hz 峰值 0.12 + 35Hz sub；画面 0.08s 暗闪）→ 余波 3s →「记录到一次低频事件。来源不明。」仅静默或 depth>1000。
- 冲击 ≤1.5s；音量上限 0.14；节点全 disconnect；reduced-motion 跳过视觉抖动/暗闪，保留音频与日志。

### 实现说明

- `updateEncounters` 在 tick 中位于 `updateDepth` **之前**——事件先推深度，P11 速度/区域切换/深度锚点当帧即可读到。
- 音频链新增：`fx → fxLP`（应力闷化）、`master → eventLP → volGain`（事件闷化）、volGain → batGain → batLP → destination 不变。
- 设置页新增「环境行为」两个开关（默认开），关闭即完全短路（进行中的事件立即终止并清理）。

---

## v5 — 2026-10-18 · P16 音效系统 + P18 工具音身份 + P17 深度背景差异化

**本轮核心**：「感官可辨识」——听得出深度/类型/操作，看得出 zone。音效仍克制，不压过环境。

### P16 音效系统

- **总线**：master 下拆 `ambientBus 0.15` / `fxBus 0.45`（约 1:3），SFX 全走 fxBus；静默两路同比例压（0.045/0.15）；HADAL fxBus ×0.85（sonar 内部 ×1/0.85 补偿——工单声明的例外）；电量 <20% fxBus ×0.7。hum/bio 维持 v2/P3 的直连（增益已含目标电平，见文件头注释）。
- **sonar 参数化**：四 zone 各自的起始/终止频率、时长、feedback（920→220/1.1s/0.20 … 720→130/1.5s/0.32）。
- **回波空间化**：扫描命中时经 `StereoPannerNode` 按 `(sig.sx-probeX)/(W/2)` 定位；同帧多命中只发最近一个；发射音居中。
- **blip 类型音程**：organism 五度 1140 / vent 三度 950 / wreck 二度 860 / ruin 大七度 1350 / echo 760+810 不协和同响。
- **环境呼吸**：两枚 drone 各挂 0.035/0.055Hz 增益 LFO（±25%，第二只起始相位错开 0.4 周期）；depth>800 叠 25Hz 底噪呼吸（±15%，跨 800 触发）。
- **留白**：sonar/blip/thump 同名 3s 窗口 ×0.7→×0.4→静音，停 2s 重置；confirm 豁免。
- **静默单点声响**：20–40s 一次（0.08s 噪声 200Hz gain 0.03 或 45Hz 正弦 0.06s gain 0.025），不写日志不计数。
- **音量三档**：×0.7/×1.0/×1.3（volGain 独立节点，不超硬性上限），覆盖层「设置」标签页，`'abyssal.volume.v1'`。
- **无泄漏**：所有一次性节点 `onended` 后 disconnect（`track()` 助手）；sonar 的 delay+feedback 2s 后断开。

### P18 工具音效身份

- 共享 `fxToolBus 0.6 → toolLP`；同名 200ms 去重；静默只保留深度类；电量 <20% toolLP 500Hz。
- 深度 tick：上浮 480→500 / 下潜 380→360（0.04s gain 0.018），50m 一声（reduced-motion 100m）；0/1400m 极限双音 520+390。
- 静默切换：开 = 噪声 300Hz 0.12s + 60Hz 脉冲 / 关 = 反序、lowpass 400Hz。
- 音频开关：开 = 440/660 两短音 / 关 = 反序 / 失败 = 220Hz 0.15s（替代 v2 的 chime）。
- 记录成功：blip 后 0.15s 叠 180Hz 落定音（0.08s gain 0.015）。
- 解码成功：520→660→780 三上行（各 0.06s 隔 0.06s，末音 0.2s）——接线到 DS-06 记录解锁（档案页「已解密」语义）。**说明：应用内不存在「解码失败」路径，decodeFail 音形已定义但未接线。**
- 覆盖层：开 = 噪声 200→600Hz 扫频 / 关 = 反扫 / 标签 = 300Hz 0.03s。

### P17 深度背景差异化

全部绘制在现有 `drawMid` / `drawFarExtras` 执行路径内（`drawSeaLife` 由 drawMid 调用），无新 canvas 层，新增元素合计 ≤40 个 1px 点，不占粒子预算：

- PHOTIC：鱼群 5→2（每条 3–7 点水平漂移，受光变亮并逃离 1s；reduced-motion 只变色不加速）；海藻高度 ×(1-frac×0.3)。
- TWILIGHT：水母 3→5（上下浮动，触须 2–3px，受光收缩触须；鲸落骨架清晰度 ×(1-frac×0.4)）。
- ABYSSAL：管虫 6→3（喷口旁，顶端微光、受光增强）+ ember 色热液羽流 1px 点上浮（高度 ×(1-frac×0.3)）。
- HADAL：移除生命元素与喷口（按工单「只留岩壁和沉积物」——**改动：vents 不再画入 HADAL**），frac>0.7 偶现 2–3 条垂直线结构剪影（<5% 画面）。
- 密度只改 count/尺寸参数不重建对象；全部受 `lightExposureAt` 换色；光锥扫过无日志。

### 约束保持

不新增音效"类型"（仍 osc+gain+filter）、不提高音量上限、环境:音效 ≈ 1:3、短促不旋律化、一次性节点全清理。

---

## v4 — 2026-10-18 · P10 图鉴 + P11 深度视觉 + P12 DS-06 档案 + 电量氛围

**本轮核心**：「长期目标 + 探索反馈 + 世界厚度」。电量仅作氛围，永不阻断操作。

### P10 图鉴持久化

- `CODEX_KEY 'abyssal.codex.v1'`；`state.codex{types, maxDepth, silentFinds, total}`，init 读取失败用默认值。
- `activateSignal` 归档时累计（类型计数 / 总数 / 静默发现 / 最深发现深度）并写 localStorage。
- 入口：status panel led-row 下方「CODEX / 图鉴」按钮（不新增面板）+ `C` 键 + hints 提示。
- `#codexOverlay`：fixed 居中、Esc / 背板点击 / 关闭钮均可关；底部留白 84px 避开 `.ops`；标签页结构预留（本轮两个标签）。
- 无数据时显示「尚无记录」；每类未归档显示「—」。

### P11 深度视觉反馈

- `ZONES` 补 `nextRamp`（HADAL 自指）；`buildBackdrop` 按 `depthFrac` 用 bayer 对色板索引做逐像素混合——**不引入新颜色**，688→999m 背景连续变暗，不在 1000m 突变。
- `getBackdropKey` 粒度 /8→/4（配合 110ms 重建节流）。
- `state.depthVelocity`（m/s，clamp ±80 + 轻平滑）驱动三项：
  - snow 粒子纵向流速 `±0.008×|v|/80`（下潜加速下落，上浮反向）；
  - `state.bgScroll` 亚像素累积取整 `±1px`，背景随下潜轻微上移，停止 ~1s 内回零；
  - `lightExposureAt` range 最多收窄 5%。
- 粒子数随深度 50→30→22（每跨 100m 增删，不重建数组）；800m 以下新增 `sediment` 粒子（vy 0.001–0.002、shelf 色、占比随深度 6%→34%）。
- reduced-motion：跳过粒子加速与背景偏移，保留渐变混合。

### P12 DS-06 前置探勘记录

- `'abyssal.ds06.v1'`；13 条记录池：类型 5（首次归档对应 type）/ 深度 4（跨过 200/600/1000/1400，双向）/ 异常 1·3·6·9（echo 归档累计次数）。
- `unlockNarrative(id)` 统一入口 + 每帧最多解锁一条的队列；日志 `addLog('ARC', '◇ '+文本, 'quiet')`，复用 log-item 样式，未新增 CSS 类。
- 全解锁后延迟 3s 收束日志「如果你读到这里，说明我停下的地方，你还在。」——`final` 写入 localStorage，只出现一次；复位会丢计时器的边角情况在 `processNarrative` 里重新武装。
- 档案页 = 图鉴覆盖层第二个标签：三组分列，未解锁显示「◇ ▓▒░ 记录缺失」，顶部「已解密 N / 13」，final 单独置底。

### 电量氛围（不限制）

- `state.battery` 100 起：PHOTIC 0.2 / TWILIGHT 0.25 / ABYSSAL 0.3 / HADAL 0.4 每秒；静默 +0.8 回升；声呐期间 +1.5。
- <20%：光锥 ×0.7、主输出增益 ×0.7、lowpass 600Hz、电量条 alert 色；<5%：光锥 ×0.42、lowpass 320Hz。
- HUD：telemetry panel tape 下方**独立 PWR 行**（未与信号条共用——理由：信号条的六段刻度可读性值得保留，且 telemetry 纵向有空间）。
- 音频链改为 `master → batGain → batLP → destination`；音频懒加载后首次开启时同步当前电量档。
- 不强制静默、不强制上浮、不阻断操作、不扣任何东西；耗尽只是更暗更闷，静默时自己回来。

### 约束保持

单文件、无外部资源、不新增色板；tick() 仅增 `updateBattery` / `processNarrative` 两处调用；图鉴与 DS-06 合并同一个覆盖层；解锁日志 kind='quiet' 且每帧最多一条。

---

## v3 — 2026-10-18 · P6 首次引导 + P9 区段目标 + P8 静默收益

**本轮核心**：「入口清晰 + 有方向 + 有奖励」。不引入资源耗尽惩罚、强制位移、归档拒绝条件、失败结局；未达成时一切操作照常。

### P6 首次引导（TUT）

- `state.tutorial{active, step, forceSig, bypassRange, forceLabel}`；`TUT.steps` 四步 `move → scan → archive → depth`，各带 `onEnter/check/onExit`。
- 启动：`scheduleOpening` 3.2s `arms` 置位后查 `localStorage 'abyssal.tut.v1'`，未设则 `TUT.start()`；结束/跳过写标记，二次进入不再触发。
- 首屏焦点：`generateInitialSignals` 给第一个信号打 `tutorialPin`（`updateSignals` 中 target 0.22，若隐若现）；第三步强制 `reveal=1 / sonarGlow=1.6 / hold=9999` 并 `bypassRange + forceLabel`，完成后恢复常规衰减。
- 视觉：`body.tut-active` 压暗非目标面板（opacity .35，`.ops` 始终 1），目标加 `.tut-hl` outline glow；`#tutHint` fixed 底部居中单行、`pointer-events:none`、`aria-live` 播报。
- 跳过：任意步骤按 `H`，或非目标区域点击 3 次（`handleTap → TUT.noteTap`）。**说明：本应用没有独立帮助面板，「打开帮助即跳过」这条路径不存在，以 H 键与三次无效点击替代。**
- 无障碍：不锁焦点，Tab 仍可达全部按钮；reduced-motion 下无进入动画。

### P9 区段目标

- `state.sectionTarget=6 / sectionArchived / sectionDone`；`activateSignal` 归档计数。
- 达标三重反馈：雷达全环常亮 3s（复用 `completeUntil`）+ 已归档信号闪烁（`completeFlashUntil`）+ `Sound.confirm()` + 日志「本区段测绘完成。DS-07 进入自由探勘。」+ 屏幕中央提示「区段测绘完成 · 6/6」2.6s。
- `updateDepth` 跨 zone 重置计数并记「进入 [zone.cn]。新目标：6 个归档。」。
- taskline 挂在**现有雷达面板内**（`.task-line`，不新增独立面板）：未完成「本区段 x/6」，完成后「本区段 6/6 · 自由探勘」。
- 未达成不设门禁：换深度、探索、声呐全部照常。

### P8 静默收益

- `toggleSilent` 进入静默：`anomaly.nextAt = elapsed + 60~100`（异常打扰减半）、`silentBonusTimer = elapsed + 4~10`、屏幕提示「静默 · 稀有生物接近中」。
- `updateSignals` 到期强制显形一个 ORG 信号（`silentFind`，`reveal=1, hold=8`）——优先复用现有未归档 ORG，否则新建（允许超出 6 上限，保证「静默 5–10 秒内出现」的承诺）；日志 + 中央提示「静默发现 · 发光生物群」。
- 归档 `silentFind` 时额外日志「静默发现 · 该生物只在无源环境下出现。」，无其他门槛、无数值奖励。
- `silentFind` 豁免 P4「远距离需声呐确认」门（静默中无法声呐，不设归档拒绝条件）。
- 解除静默清零计时器；非静默绝不出现该生物。静默期间移动光标、调深度、查看已归档信号均不受影响。

### 约束保持

不新增色板、不新增外部资源；`tick()` 仅增调用不改结构；新增覆盖层 `#tutHint` / `#centerToast` 均 fixed、`pointer-events:none`、Esc 可关、不遮挡 `.ops`；文案保持短句仪器感。

---

## v2 — 2026-10-18 · 音频可达性（R1）

**起因**：v1 交付后用户反馈「没有声音」，且「没见到 AUDIO」。上一轮排查停在「按钮几何可见」，没有闭环。

**根因（三处，均已实测或按信号链算出）**

| # | 问题 | 证据 |
|---|---|---|
| R1.1 | 不可发现：AUDIO 是右下 59px 小按钮，默认 OFF，hints 里没有提及 | 1440×900 下 rect = `[1361,806,59,38]`，位于视口最角 |
| R1.2 | 不可确认：开启成功后只有一行日志，没有任何声音反馈 | `toggleAudio()` 原实现只有 `addLog` + `updateHud` |
| R1.3 | 不可听见：两枚 drone 在 30–55Hz，且经 lowpass 80Hz | `0.72 × 0.15 × 0.024 ≈ 0.0026`，约 **-52dBFS @ 36Hz**，笔记本扬声器无法重放 |

**改动**

- `hints` 增加「`A` 音频」；`onKeyDown` 新增 A 键切换（键位本身即用户手势，满足自动播放策略）。
- `scheduleOpening` 在 7.8s 检测仍静音时写 AUD 日志（同时进 `aria-live`），并给 `#audioBtn` 加 `.is-attn`：两次边框脉冲，7.2s 后移除。
- 新增 `Sound.chime()`：开启成功后播 659.25 / 987.77Hz 两音，使「点了有没有通电」一秒可判。
- 新增 mains-hum 层：triangle `220 / 196 / 175 / 147Hz` 随分区降调，增益 `0.0072 → 0.0044`，直连 `master`，静默时压到 45%。低频叙事不变，只补了扬声器能重放的那一段。
- `applyTone()` 负责 hum 的频率与增益；`setQuiet()` 经 `applyTone()` 联动。

**约束保持**：单文件、无外部资源、12 色板、Bayer 抖动、`tick()` 主循环未改动、音频默认关闭、`prefers-reduced-motion` 下 `.is-attn` 动画关闭。

---

## v1 — 2026-10-17 · 初版 + P0–P5

**初版**：按 `SPEC.md` §A 一次成型，单个 HTML，约 4300 行。交付时仅做静态检查（无色板外颜色、无外部资源、结构完整），**未在浏览器中运行**。

**验证**：随后用 headless Edge + CDP 探针跑通，12 张截图；发现下潜至 HADAL（1000m+）时 `drawMid` 抛 `ReferenceError`——早于运行交付，属盲写代价。

**P0–P5**（按 `SPEC.md` §B 实施）

- **P0 音频可用性**：`Sound.toggle()` 在 `resume()` 失败或 `ctx.state !== 'running'` 时返回 `null`；新增 `Sound.ensureRunning()` 处理 bfcache 返回后上下文挂起；失败时保持 OFF 并记录「音频总线不可用」。
- **P1 深度声景**：`Sound` 保存 lowpass / noiseGain / 两枚 drone 引用；`setZone()` 以 `setTargetAtTime` 切换四层截止频率、基频与噪声；`updateDepth` 与初始化各调用一次。
- **P2 异常归档**：`state.anomaly.logged`；声呐命中异常时写 alert 日志，雷达保留橙色固定标记（异常结束后不消失）。
- **P3 静默听感**：`updateSignals` 中 hold 衰减非静默 2.2/s、静默 1.0/s；`setQuiet()` 压低 ambient、lowpass 压到 45Hz、淡入静默生物声层。
- **P4 深度代价**：`depthPenalty` 乘到 reveal 速度；`lightExposureAt` 深水半径 ×0.9；声呐冷却 1.15s→1.2s；>160M 且无声呐确认的接触先拦截。
- **P5 全归档事件**：六件归档触发 3s 雷达全环、归档信号闪烁、低频确认和弦与叙事日志。
- **FIX**：`drawMid` 的 HADAL 装饰改用 `state.W/state.H`，修掉上述 `ReferenceError`。

**遗留**：音频可用性只在参数层验证，未做人工听感测试——v2 的 R1 即由此而来。
