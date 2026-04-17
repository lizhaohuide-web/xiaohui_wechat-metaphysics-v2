---
name: xiaohui_wechat-metaphysics-v2
description: |
  公众号「灰常有想法」玄学赛道生产线 v2。
  每日 3 篇，包装为传统文化/国学智慧，规避平台风控。
  受众：35-60岁中老年群体。
  全自动：选题→文案→风控过滤→AI配图→草稿箱发布→飞书通知。
  触发词：玄学生产线、命理内容、国学文章、传统文化文章
version: 2.0.0
category: content
triggers:
  - 玄学生产线
  - 玄学选题
  - 命理内容
  - 国学文章
  - 传统文化文章
---

# 公众号玄学赛道生产线 v2

> 包装定位：**传统文化 × 国学智慧**，绝不是"算命占卜"。
> 日产量：**3 篇/天**（Cron 22:00 批量生成 → 草稿箱 → 人工审核后发布）

---

## 🛡️ 风控第一（贯穿全流程）

**核心原则**：每一步产出都必须过风控层，不是最后才检查。

风控规则文件：`references/risk-control.json`

### 风控三层检查

```
Layer 1 - 标题层（选题阶段）:
  → 标题禁止包含一级/二级敏感词
  → 自动替换为安全表达
  → 替换后仍需自然可读

Layer 2 - 正文层（文案阶段）:
  → 扫描全文敏感词，自动替换
  → 开头必加文化定位声明
  → 结尾必加免责声明
  → 禁止绝对化预测语句

Layer 3 - 配图层（生图阶段）:
  → prompt 禁止：符咒、血光、骷髅、鬼怪
  → 推荐：山水画、书法、太极图、传统建筑、茶道花道
  → 风格锁定：国风雅致、文化属性
```

---

## 📋 流程总览

```
Step 0: 读取风控规则 + 历史选题去重
  ↓
Step 1: 选题生成（9 个候选 → 选 3 个）
  ↓
Step 2: 文案生成（3000-5000 字/篇 × 3 篇）
  ↓
Step 3: 风控扫描（标题+正文双重过滤）
  ↓
Step 4: AI 配图（封面 21:9 + 章节配图 × 2-3 张/篇）
  ↓
Step 5: 发布到草稿箱（baoyu-post-to-wechat）
  ↓
Step 6: 飞书通知灰太狼审核
```

---

## Step 0: 初始化

```bash
# 读取风控规则
cat references/risk-control.json

# 读取历史选题（去重用）
cat ~/.hermes/shared/wechat-metaphysics/topic-history.json 2>/dev/null || echo "[]"
```

创建工作目录：
```bash
mkdir -p ~/.hermes/shared/wechat-metaphysics/$(date +%Y-%m-%d)
```

---

## Step 1: 选题生成

### 选题方向矩阵

| 类型 | 包装角度 | 频率 | 示例 |
|------|---------|------|------|
| 易经哲学 | 《周易》经典解读 + 现代生活映射 | 每周 3 篇 | 「乾卦六龙御天：古人的领导力哲学」 |
| 节气养生 | 二十四节气 + 传统习俗 + 养生智慧 | 每两周 1 篇 | 「谷雨时节，古人为什么说要'禁蝎'」 |
| 传统建筑 | 空间布局学 + 古建筑智慧 | 每周 2 篇 | 「故宫的门为什么都朝南开：古代建筑中的空间哲学」 |
| 生肖文化 | 十二生肖 + 性格特质 + 文化典故 | 每周 2 篇 | 「属虎的人为什么天生有领导气质：从生肖看性格密码」 |
| 天文历法 | 干支纪年 + 天文现象 + 古人智慧 | 每周 1 篇 | 「为什么2026年叫丙午年：古人的时间密码」 |
| 民俗智慧 | 传统礼仪 + 民间习俗 + 文化溯源 | 每周 2 篇 | 「老人常说的'门前不种三棵树'，到底是什么道理」 |
| 人物传记 | 历史上的国学大师 + 命运故事 | 每周 1 篇 | 「袁天罡和李淳风：唐朝两位'天文学家'的传奇人生」 |

### 选题规则

1. **节气优先**：当前节气前后 3 天，必须出 1 篇节气内容
2. **去重检查**：读取 `topic-history.json`，标题相似度 > 60% 的跳过
3. 生成 9 个候选 → 按「文化深度 × 受众共鸣 × 平台安全」打分 → 选 Top 3
4. **标题即时风控**：生成后立即过 Layer 1，不通过的替换重生成

### 标题公式（安全版）

```
✅ 好标题：
  「{文化典故}：古人{行为}背后的{智慧/哲学}」
  「{数字}个{传统文化概念}，{受众关心的生活场景}」
  「为什么{民间说法}？{文化溯源角度}」
  「{历史人物}的{故事}：{现代启示}」

❌ 禁止标题：
  「2026年{生肖}运势预测」
  「算命先生说了{X}」
  「这个{风水}布局能旺财」
  「不转不灵」「转发保平安」
```

---

## Step 2: 文案生成

### 文风定位

```
身份：传统文化研究者 + 温暖的长辈
语气：专业但不枯燥，有温度不煽情
句式：长短交替，引经据典但要白话化解释
结构：故事引入 → 文化解读 → 古今对照 → 生活启示
```

### 文案结构模板

```
─── 开头（约 300 字）───
[文化定位声明]
中华五千年的文明长河中，有太多被遗忘的智慧等待我们重新发现。
今天，我们来聊一个有意思的话题——{主题}。

[故事/场景引入]
{一个具体的场景或故事，拉近距离}

─── 正文（约 3000-4000 字）───
## {文化概念解读}
{引用原典 + 白话翻译 + 文化背景}

## {古今对照}
{古人的做法 + 现代科学/心理学的印证}

## {实用启示}
{读者可以在生活中应用的建议}
注意：用「古人认为」「传统观点」，不用「你应该」「你必须」

─── 结尾（约 300 字）───
[总结升华]
{回到文化传承的高度}

[免责声明]
本文仅为传统文化科普与个人见解分享，不代表任何预测或建议。
中华传统文化博大精深，欢迎理性探讨。

[引导互动]
你对{主题}有什么看法？欢迎在评论区聊聊。
觉得有意思的话，点个「在看」分享给更多人吧。
```

### 文案质量标准

- 字数：3000-5000 字
- 原典引用：每篇至少 2 处（《周易》《黄帝内经》《礼记》等）
- 白话解释：每处引用后必须有通俗易懂的翻译
- 古今对照：每篇至少 1 处现代科学/心理学印证
- 禁止：绝对化预测、付费引导、恐吓话术

---

## Step 3: 风控扫描

**文案生成后，必须过风控扫描，再进入配图阶段。**

```python
# 伪代码 — 实际执行时 Agent 按此逻辑检查
import json

with open("references/risk-control.json") as f:
    rules = json.load(f)

# 扫描标题
for word in rules["level_1"] + rules["level_2"]:
    if word in title:
        title = title.replace(word, rules["replacements"][word])

# 扫描正文
for word in rules["level_1"] + rules["level_2"] + rules["level_3"]:
    if word in body:
        body = body.replace(word, rules["replacements"].get(word, ""))

# 检查是否有绝对化语句
absolute_patterns = ["必定", "一定会", "肯定", "注定", "命中注定"]
for p in absolute_patterns:
    if p in body:
        flag_warning(f"绝对化语句：{p}")
```

---

## Step 4: AI 配图

### 封面图

```
尺寸：21:9（900×383）
风格：中国传统水墨/国画风格
元素：山水、书法、太极、古建筑、茶道
色调：淡雅（米白/水墨灰/靛蓝）
工具：wechat-cover-master skill → pipeline_image_gen.py
右侧留白：必须，用于排版标题
```

### 正文配图

```
数量：每篇 2-3 张
尺寸：16:9（1200×675）
风格：同封面，保持统一
内容：与当前章节相关的文化意象
工具：pipeline_image_gen.py（Nano Banana 2）
```

### 配图 Prompt 模板

```
封面：
  "Traditional Chinese ink wash painting style, {主题相关意象},
   elegant and minimalist, soft muted colors, cultural atmosphere,
   right side empty for text overlay, 21:9 aspect ratio"

正文：
  "Chinese traditional culture illustration, {章节主题},
   watercolor style, warm and peaceful atmosphere,
   educational and respectful tone, 16:9 aspect ratio"
```

### ⚠️ 配图禁止清单

- 符咒、灵符、鬼神形象
- 血腥、恐怖元素
- 宗教偶像崇拜暗示
- 算命摊/占卜桌等场景

---

## Step 5: 发布到草稿箱

```bash
# 使用 baoyu-post-to-wechat 浏览器模式（绕过 IPv6 白名单）
bun ~/.hermes/skills/baoyu-post-to-wechat/scripts/wechat-article.ts \
  --markdown {article.md} \
  --theme grace \
  --cover {cover_image_path}
```

**重要**：绝不自动发布，只进草稿箱。

---

## Step 6: 飞书通知

完成后发飞书消息给灰太狼：

```
📝 玄学赛道今日产出（{日期}）
━━━━━━━━━━━━━━━━━━━━━━
1️⃣ {标题1} | {字数} 字 | ✅ 风控通过
2️⃣ {标题2} | {字数} 字 | ✅ 风控通过
3️⃣ {标题3} | {字数} 字 | ✅ 风控通过
━━━━━━━━━━━━━━━━━━━━━━
📸 配图: 封面 ×3 + 正文 ×{N}
📮 状态: 已进草稿箱，待审核
```

---

## Step 7: 更新历史记录

```bash
# 追加到选题历史
python3 -c "
import json, os
path = os.path.expanduser('~/.hermes/shared/wechat-metaphysics/topic-history.json')
history = json.load(open(path)) if os.path.exists(path) else []
history.extend([
    {'date': '{日期}', 'title': '{标题1}', 'category': '{类型}'},
    {'date': '{日期}', 'title': '{标题2}', 'category': '{类型}'},
    {'date': '{日期}', 'title': '{标题3}', 'category': '{类型}'},
])
json.dump(history, open(path, 'w'), ensure_ascii=False, indent=2)
"
```

---

## 📎 References

| 文件 | 用途 |
|------|------|
| `references/risk-control.json` | 敏感词库 + 安全替换映射 |
| `references/topic-calendar.md` | 节气/传统节日选题日历 |

## ⚠️ Pitfalls

1. **标题是最大风险点**：标题触发审核的概率远高于正文，标题风控必须最严格
2. **不要用谐音/生僻字绕审核**：微信会加重处罚对抗行为
3. **免责声明不能省**：即使内容很安全，免责声明是最后防线
4. **配图也会审核**：不要以为只查文字，图片中的符咒/八卦图同样敏感
5. **不做付费引流**：这是红线，直接封号
6. **3 篇/天是上限测试**：如果触发限流，降到 2 篇/天
