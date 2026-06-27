# Skill: explanation-illustration

# Explanation Illustration

## Inputs to collect

- **文章正文**：用户粘贴的全文。如果用户只给了标题或要点，先问正文，否则无法识别结构。
- **结构类型提示**（可选）：用户有时会说"这是 X vs Y 的对比"或"这是 3 个阶段"，能省一轮猜测。多数情况下让模型自己读出来。

不要问的：风格选择（已固定）、画幅比例（由内容决定）、主色（已固定为暖白底 + 4–5 色克制配色）。

## Procedure

### 1. 读文章，判定结构类型

把文章压成一个结构判断，只用 4 个标签之一：

- **A 对照**（并列对比，如「A 三层 vs B 三层」）→ 横版 3:2 或 4:3，左右或上下分栏
- **B 递进**（线性阶段，如「第 1 步 → 第 2 步 → 第 3 步」）→ 横长条，优先 21:9，回退 16:9，从左到右流程
- **C 层级**（深度/嵌套，如「底层 → 中层 → 顶层」）→ 竖版 3:4 或 4:5，从下到上或金字塔
- **D 凝聚**（一个核心概念，多个属性围绕）→ 方形 1:1，中心 + 放射

判定后简短回报用户一行："我读出来是 B 递进，3 个阶段，横长条 3:1"。原因：让用户有机会纠错（"其实是对照不是递进"），比画完再返工便宜。

### 2. 构图与元素

按结构类型填分区：

- **分区数**：通常是 2–4 个。超过 4 个 → 砍掉次要的、并掉次要的；拥挤比信息少更糟
- **每分区**：1 个核心场景/人物 + 1 个核心物件 + 1 个 2–6 字中文标签
- **分区间**：箭头 / 台阶 / 虚线 / 流程线；选最贴合结构的，不要堆符号
- **空白**：分区之间留呼吸空间，整体不挤

如果读完文章发现结构不属于 A/B/C/D 任一（如叙事+反思+行动三段但每段独立），选最易读的那个标签并合并分区。

### 2.5 角色规格（全局通用）

所有分区的"主角"沿用同一规格，保证风格一致与系列感：

- **头部**：圆形，约为躯干宽度的 2 倍；**面部完全留白**（无表情、无点眼、无嘴）。这是默认规范 —— 留白 = 谁代入都行，且跨分区一致性最强。情绪表达靠道具 / 姿态 / 符号（✗/✓/思考云），不靠面部
- **跨分区面部一致性**：所有分区头部完全相同（圆形、留白、无变化），不画任何面部细节。表情变化被显式禁止
- **躯干**：单竖线（纯火柴人）或细矩形，长宽比约 2.5 : 1
- **四肢**：单线条，关节不弯曲
- **手**：圆圈或单线条末端，**无**手指细节
- **着装**：**默认不画**。纯线条火柴人最克制，颜色配额留给核心物件（✗/✓、思考云、清单等）。如某个分区需要视觉差异化，可临时加 `optional white shirt color block` 作为变体，不作为默认
- **大小**：单分区高度填满 60–70%，避免"小人在大场景里"那种失重感

**绝对约束 prompt 写法（实测有效）**：跨分区一致性是 image_synthesize 最容易跳脱的地方。面部完全留白是默认规范（无表情变量），但**不变的部分**（头型、身体比例、线条颜色）仍需用**肯定、绝对、重复**的措辞锁定。模板句：

```
Character: strictly identical stick figure across all panels —
same round blank head (no eyes, no mouth), same body proportions,
same line color. No facial details in any panel. Only posture and
props change between panels.
```

否定句（"no X", "don't vary"）单独使用不可靠，必须配合肯定句+重复+绝对词（strictly, absolutely, identical）。

**已知偷跑清单（image_synthesize 默认行为）**：以下场景会自作主张画面部细节，即使 prompt 明确写"完全留白"。**默认接受小瑕疵**（偷跑弧线微弱，不影响主题表达）；如需严格避免，用括注里的反向重申 prompt：

- **完成/落点/成功语境的角色**会**自动加微笑弧线**（B 递进的第 3 区高频出现）。示例：system-migration-engineering-v3.png 第 3 区"工程"角色、voice-to-article-bridge.png 第 3 区"自己搭桥"角色都观察到。严谨 prompt：`the character in panel [N] has the SAME completely blank round head as panels [M] — no smile, no arc, no implied expression of any kind, even though the context is positive`

**跨分区角色规则（按结构类型分）**：

- **B 递进 / C 层级 / D 凝聚**（同一主体在演变）→ **复用同一形象**（同色同形），只在**动作 / 道具 / 朝向**上变化。这条保证多分区画面的系列感
- **A 对照**（两个并列主体）→ **用 2 个不同形象**，靠**颜色 + 轮廓细节**区分（如左=墨绿圆头、右=砖红方头）。同一形象会抹掉"两个体系在对照"的语义

判断标准："读图的人能否一眼看出这是同一主体的演变还是两个并列主体" —— 能 → 复用；不能 → 区分。

**禁止**：写实肤色、发色（除头部轮廓的简单色块）、任何面部细节（眼/嘴/表情变化）、性别特征、服饰潮流元素。违规 → 风格跑偏 → 重写。

### 3. 写 image_synthesize 的 prompt

把上面的判断压成一个英文 prompt（image_synthesize 对英文 prompt 渲染更稳定），但**中文标签必须作为约束写进 prompt，要求画面里出现这几个汉字**。

prompt 模板骨架（按结构类型选一个）：

```
Flat cartoon explanation illustration, New Yorker / xkcd style,
warm off-white background, hand-drawn linework, no gradients, no 3D, no realism.
[结构类型描述，例如 "three side-by-side stages flowing left to right with
arrows between them" / "two columns comparing X vs Y" / "stacked layers
from bottom to top" / "one central concept surrounded by attribute icons"].
[每分区的具体场景描述：人物动作 + 核心物件 + 中文标签 in brackets]
Character: strictly identical stick figure across all panels — same
round blank head with completely blank face (no eyes, no mouth, no
facial details), same body proportions, same line color. No facial
details in any panel. Only posture and props change between panels.
[B 递进 / C 层级 / D 凝聚：复用同一形象，靠动作 / 道具 / 朝向变化]
[A 对照：写"two distinct figures, left panel uses [color/shape], right panel uses [color/shape]"]
Color palette: warm white background, deep ink lines, plus 1–2 accent colors
(terracotta orange, ink green, or deep navy). Maximum 4–5 colors total.
Style: minimalist, editorial, no watermark, no signature, no logo, no QR code.
[如适用：少量服务主题的英文术语，如口语习惯里的 "um/uh/like"；禁止英文版权/签名/Legal 类]
```

中文标签的处理：
- **首选**：把 2–6 字汉字直接写进 prompt，例如 `a small sign labeled "递进"` —— image_synthesize 在拉丁文 prompt 嵌入中文时偶有渲染失败
- **更稳的方案**：生成图后用 Node.js + `sharp` 或 Python + Pillow 把中文标签合成上去（需要多模态视觉校验）。默认走「写进 prompt」这一条；当且仅当校验发现字符乱码/伪中文时再走合成
- **字符密度经验**：当一个分区需要 3–4 个并列的中文标签（如「不要 / 要 / 自己 / 桥接」），image_synthesize 容易漏字或糊成一片。**改用通用视觉符号替代文字标签**（红 ✗ = 不要、绿 ✓ = 要），只在分区底部留 1 个 2–3 字的主标签。符号不会被渲染引擎吞掉，且视觉信息密度更高

### 4. 生成 + 校验

- **先精简 prompt**：实际经验是，超过约 200 词的 prompt 会触发上游过滤静默失败（没有任何错误信息）。压 prompt 时保留：风格句 + 结构类型 + 每分区一句场景描述 + 配色 + 排除项。砍掉任何重复修饰、举例性描述、解释性句子
- **比例兜底**：image_synthesize 对极端横长比（实测 3:1 反复静默拒绝、2.45:1 推测同样风险）支持不稳。B 递进想用横长条时，先试 21:9；失败回退 16:9。校验清单里加一条"比例生成是否成功"，失败 → 立刻换比例不要重写 prompt
- 调 `image_synthesize` 生成候选图（aspect_ratio 严格按 step 1 的判断）
- **跨分区一致性必须校验**：检查所有分区的角色眼睛位置、嘴弧度、头部大小是否完全一致。不一致 → 不是重写 prompt，是**加强约束语气**（用绝对句、重复句、肯定句），再来一轮
- 校验清单（任一失败就重写 prompt 再来一轮，最多 3 轮）：
  - 风格是否 New Yorker/xkcd 感（不是 3D 拟真 / 不是儿童读物 / 不是中国风滥用）
  - 比例是否匹配结构
  - 中文标签是否清晰无伪字（核心检查项；多模态能力能用就用）
  - **跨分区角色一致性**（头部大小、身体比例、线条颜色必须一致 —— 面部完全留白，不画眼/嘴/表情变化）
  - 是否含水印、签名、Logo、二维码、英文版权说明（命中即重写）。**例外**：当英文是主题本身的一部分（讨论英文写作、口语习惯、跨语言场景、引用术语等），允许画面里出现少量服务主题的英文；禁止英文签名/版权/Legal类文字、禁止与主题无关的装饰性英文
  - 配色是否克制（饱和霓虹、廉价渐变、3D 光泽 = 失败）
  - prompt 字数是否 ≤ 200 词（超过 → 重写精简版再来）

### 5. 交付

最终只输出：
- 一张图
- 一句话说明：用了什么结构、什么比例、几个分区

不要附"以下是详细步骤"、不要解释为什么要这么画。

## Output contract

- 文件：一张 `<workspace>/explanation-illustration-<timestamp>.png`
- 同时在对话里用 `<deliver-assets>` 块交付给用户
- 配套说明：单行，"结构：B 递进 | 比例：3:1 | 分区：3"

## Failure handling

- **结构判定犹豫**（A vs B / B vs C 都说得通）→ 选最容易画的那一个，分区少的那一个
- **文章太短**（< 200 字，没有可识别的结构）→ 直接告诉用户"文章太短，没有可视化的核心结构"，别硬画
- **3 轮 prompt 迭代后中文标签还是乱码** → 接受小瑕疵交付，附一句"中文标签由后期合成覆盖更稳；如需我合成请确认"
- **生成图风格跑偏**（3D / 儿童 / 中国风）→ 重写 prompt 强调"editorial illustration, NOT children's book, NOT 3D render"，再来一轮
- **用户对结构判定有异议** → 改完结构判断，从 step 2 重做，不要在原图上小修

## Examples

**Input**: 一篇 800 字讲「AI 三层（数据/算法/算力）vs 编程三层（语法/逻辑/系统）」对照的文章
**判定**: A 对照 → 横版 4:3 → 左右两栏各 3 个分区 → 箭头在中间做连线
**Output**: 一张横版 4:3，左栏"数据/算法/算力"三个简化人物+物件，右栏"语法/逻辑/系统"三个简化人物+物件，中间双向箭头，暖白底 + 墨绿 + 砖红
