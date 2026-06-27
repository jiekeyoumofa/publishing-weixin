# Skill: plus-seven-cover

# 加七 Plus Seven · 公众号封面生成

为加七（JackyPan）公众号生成统一 VI 风格的封面图。所有封面遵循「晨光加七」视觉系统。

## 触发场景

用户输入包含：
- 文章标题或全文（粘贴文本）
- "做封面"、"出个封面"、"加七封面"、"公众号封面"等关键词
- 可选：副标题、日期、期号、关键数字、引文

## VI 核心规则（每次生成都必须遵守）

### 配色

| 角色 | HEX | 用途 |
|---|---|---|
| 主色 · 晨光橙 | `#E89B4C` | 强调、关键数字、副关键词、分隔线 |
| 深琥珀 | `#B67428` | 主色深档、英文斜体 |
| 湖蓝 | `#4A90B8` | 克制对比（备用，不强制） |
| 草绿 | `#6FAF6F` | 极少量点缀（备用） |
| 蜜黄 | `#F2C94C` | 5% 以内点缀 |
| 米白 | `#FAF6EE` | **主背景，强制** |
| 墨蓝 | `#1E2A3A` | **标题/正文字色，强制** |
| 纸白 | `#FFFFFF` | 卡片底（按需） |

**禁止**：纯黑 `#000000`、粉色、荧光色、霓虹色。

### 字体（描述用，模型自动选择相近字体）

- 中文标题：思源宋体（Noto Serif SC，衬线，雅致）
- 中文正文：苹方（Noto Sans SC，无衬线）
- 英文标题：Playfair Display（衬线）
- 英文副标题/说明：Outfit 或 Inter

### 布局铁律

1. **比例**：21:9（与公众号 2.35:1 视觉等效，工具不支持 2.35:1）
2. **米白背景**（`#FAF6EE`）—— 不接受纯白
3. **右下角 "+7" 落款**（墨蓝或主色，字号小）—— 每张必有
4. **右上角日期戳**（胶囊形描边，主色或墨蓝）—— 格式 `YYYY.MM`
5. **留白优先**：信息不超过 3 层（标题 / 副标 / 落款）
6. **永远不出现 emoji 装饰**

## 4 套模板（按内容自动选型）

| 模板 | 触发条件 | 构图 |
|---|---|---|
| **主题型** | 标题含多个并列关键词（"X｜Y｜Z"或"/"分隔）| 主标题居中 + 副关键词用 `｜` 分隔 + 英文小副标 |
| **数字强调型** | 标题含具体数字（X 个 / 第 N 篇 / 完成 N 件事）| 关键数字用主色 `#E89B4C`，其余墨蓝 |
| **引文型** | 文章有 1 句金句，或用户明确说"用这句" | 居中金句 + 左上引号（主色）+ 下方署名（"文章名"或"作者"） |
| **期号型** | 标题是连载/合集类（"VOL."/"第 X 期"），或用户指定 | 大数字 + "ISSUE/DAY/VOL" 小标签 + 主题说明 |

**选型规则**：4 个都不明显匹配时，默认用 **主题型**。如果用户明确说"用引文"、"用数字"，尊重用户选择。

## 执行步骤

### Step 1 · 解析输入

从用户消息中提取：
- `title` — 文章标题（必填）
- `subtitle` — 副标题/英文名（可选，从标题派生）
- `date` — YYYY.MM，默认当前月份
- `quote` — 引文型时使用
- `highlight_number` — 数字型时使用
- `keywords` — 主题型时使用
- `issue_label` — 期号型时使用（如 ISSUE / DAY / VOL）

如果 `title` 缺失，**停下来问用户**，不要瞎编。

### Step 2 · 选模板

按上表触发条件匹配。如果都不明显，问用户："你希望用 主题型 / 数字型 / 引文型 / 期号型 哪个？"

### Step 3 · 生成 Prompt

按模板生成结构化 prompt。**每个 prompt 必须包含**：
- 完整的英文风格描述（minimalist、warm、bright、editorial）
- 精确的 hex 颜色值
- 明确的字体提示（"elegant Chinese Songti serif"、"Playfair Display"）
- 布局指令（位置、留白、辅助元素）
- 比例：21:9
- **"No people, no photo, no emoji"** 收尾

**Prompt 模板参考**（存于脑内，每次按需组合）：

主题型 prompt 骨架：
```
A minimalist WeChat public account cover, ultra-wide 21:9 aspect ratio.
Warm cream off-white background #FAF6EE with a very subtle warm peach gradient at the top.
Centered large dark navy #1E2A3A serif Chinese title '[TITLE]' in elegant Chinese Songti serif font with generous letter spacing.
Below it, three smaller characters separated by vertical bar dividers: '[K1] | [K2] | [K3]' in warm amber #E89B4C.
Small English subtitle '[SUBTITLE]' in light gray.
Tiny rounded date stamp '[DATE]' in top right corner with thin border.
Small '+7' monogram in light amber at bottom right corner.
Composition is calm, bright, warm, sophisticated, premium editorial.
No people, no photo, no emoji.
```

数字型 prompt 骨架：在主标题中 `[NUMBER]` 用主色，其余墨蓝。

引文型 prompt 骨架：左上主色引号 + 居中 3 行引文 + 下方署名 + 日期戳 + "+7"。

期号型 prompt 骨架：顶部小标签 + 巨大主数字 + 下方主题 + 英文小副标 + 日期戳 + "+7"。

### Step 4 · 调用 image_synthesize

- 工具：`image_synthesize`
- `aspect_ratio`: `"21:9"`
- 不要传 `resolution` 参数（用默认值即可，2K 偏大）
- 文件路径：`/workspace/.skills/plus-seven-vi/covers/{YYYY-MM-DD-slug}.png`
  - `slug` 从标题生成（去掉标点、空格转 `-`、截前 30 字符）

### Step 5 · 交付

用 `<media src="绝对路径" />` 标签交付给用户。
附 1 句话说明用了哪个模板，方便用户复盘。

## 输出契约

每次成功执行必须交付：
1. 一张 21:9 的 PNG 封面图
2. 一句话说明（"主题型 · 2026 关键词那张风格"）
3. 文件存到 `/workspace/.skills/plus-seven-vi/covers/` 目录

## 失败处理

| 情况 | 应对 |
|---|---|
| 用户没给标题 | 问"标题是什么？其他信息我可以自动补" |
| 4 套模板都不明显 | 问"哪种风格你更喜欢？主题 / 数字 / 引文 / 期号" |
| 用户说"和上次那张一样" | 调取 `references/cover-samples.md`（见下）让模型对齐风格 |
| 工具返回失败 | 简化 prompt 重试；如仍失败，告诉用户稍后再试 |
| 用户要改局部（换颜色、去掉 +7）| 在 VI 规则允许范围内调整；超出 VI 规则的改动，跟用户确认是否"破例" |
| 用户要其他比例（小红书 3:4、朋友圈 1:1）| 提醒"这套 VI 主要是公众号 21:9，其他比例需要单独适配"，问是否要继续 |

## 风格参考

4 个 baseline 样图存于：
- `/workspace/.skills/plus-seven-vi/brand-book/cover-1.png` — 主题型
- `/workspace/.skills/plus-seven-vi/brand-book/cover-2.png` — 数字型
- `/workspace/.skills/plus-seven-vi/brand-book/cover-3.png` — 引文型
- `/workspace/.skills/plus-seven-vi/brand-book/cover-4.png` — 期号型

完整 VI 手册：`/workspace/.skills/plus-seven-vi/brand-book/index.html`（包含配色规范、字体规范、Logo 变体、使用 Do/Don't）

如果用户对生成结果说"再调一下"，先比对 baseline 找差异，再针对性调整 prompt。

## 验证清单

每次生成后自检：
- [ ] 背景是米白，不是纯白
- [ ] 标题用衬线，不是黑体
- [ ] 右下角有 "+7" 落款
- [ ] 右上角有日期戳
- [ ] 没用纯黑、粉色、荧光
- [ ] 信息层数 ≤ 3
- [ ] 没有 emoji
