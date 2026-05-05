---
name: algo-viz
description: "Generate 3Blue1Brown-style algorithm and tech learning animation videos using Manim. Supports preset scenes (sorting, graph, DL, math) with Chinese aliases, and custom multi-scene video projects with plan→code→render→stitch workflow. Use when user asks to visualize, animate, or explain algorithms/concepts with video."
args:
  - name: topic
    description: "The topic to visualize — e.g. '快速排序', 'Self-Attention', 'BFS', '梯度下降', 'Transformer架构', or a path to an .md file"
    required: true
  - name: quality
    description: "Video quality: l=480p(fast), m=720p, h=1080p. Default: m"
    required: false
  - name: data
    description: "Custom input data, e.g. '[5,3,8,1,9,2]' for sorting"
    required: false
user-invokable: true
---

# algo-viz: 算法与技术学习动画生成器

工具位于 `D:/pythonPycharms/My_jupyerlab/tools/algo-viz/`。

## 两种模式

### 模式 A: 预制场景（快速）

匹配预制场景，一条命令出视频：

```bash
python "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/generate.py" "快速排序" -q m
python "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/generate.py" --scene attention -q m
python "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/generate.py" --list  # 查看所有预制场景
```

### 模式 B: 自定义多场景项目（完整）

用户给一个主题或 MD 文件，走 **Plan → Code → Render → Stitch → Iterate** 流程。

---

## 模式 B 工作流详细

### Phase 1: Plan

在项目目录下写 `plan.md`：

```bash
mkdir -p "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/output/<project-name>"
```

```markdown
# [Video Title]

## Overview
- **Topic**: [Core concept]
- **Hook**: [Opening question]
- **Key Insight**: [The "aha moment"]
- **Resolution**: [480p(default) / 720p / 1080p]

## Narrative Arc
[2-3 sentences: from confusion to understanding]

---

## Scene 1: [Name]
**Duration**: ~Xs
**Purpose**: [What this accomplishes]
### Visual Elements
- [Mobjects, animations, camera movements]
### Content
[What happens]
### Subtitles
- "[text]" → syncs with [animation]

## Scene 2: [Name]
...

## Color Palette
- Primary: ACCENT_BLUE (#58C4DD) - main elements
- Highlight: ACCENT_YELLOW (#FFFF00) - emphasis
- Positive: ACCENT_GREEN (#83C167) - conclusions
- Alert: ACCENT_RED (#FC6255) - key terms
- Background: #1C1C1C
```

### Phase 2: Code

在同目录下写 `script.py`。**一个 class 一个场景**：

```python
from manim import *
import sys, os
sys.path.insert(0, r"D:\pythonPycharms\My_jupyerlab\tools\algo-viz")
from scenes.style import *  # 导入组件库

class Scene1_Introduction(StyledScene):
    def construct(self):
        self.caption("Introducing the topic", duration=2)
        mobs = self.show_title("Title", "Subtitle")
        self.wait(W_STEP)
        self.transition(*mobs)

class Scene2_CoreConcept(StyledScene):
    def construct(self):
        step = self.show_step("Step 1: The key idea")
        # ... animations ...
        self.clear_step(step)
```

### Phase 3: Render + Stitch

用 generate.py 的 `--stitch` 模式一键完成：

```bash
python "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/generate.py" --stitch "output/<project>/script.py" -q m
```

这会：
1. 自动发现 script.py 里所有 Scene 类
2. 分别渲染每个 Scene
3. 生成 concat.txt
4. ffmpeg 拼接为 final.mp4

或手动分步：

```bash
# 渲染
python -m manim render -qm script.py Scene1_Introduction Scene2_CoreConcept

# 拼接
ffmpeg -y -f concat -safe 0 -i concat.txt -c copy final.mp4
```

### Phase 4: Iterate

用户反馈后，只重渲出问题的 scene：

```bash
python -m manim render -qm script.py Scene2_CoreConcept  # 只渲染改过的
# 然后重新拼接
```

---

## 审美规范 (写新场景时必须遵守)

### 基类

```python
from scenes.style import *

class MyScene(StyledScene):  # MovingCameraScene 子类
    def construct(self):
        ...
```

StyledScene 提供:
- `show_title`, `show_step`, `clear_step`, `transition`
- `zoom_in(mob, scale)`, `zoom_out()`, `auto_fit(group)`
- `indicate(mob)`, `flash_highlight(mob)`, `subtle_pulse(mob)`
- `staggered_fadein`, `staggered_create`, `staggered_write`
- `show_formula(latex, explain=...)`
- `caption(text, duration)` — add_subcaption 包装

### 配色语义

| 颜色 | 常量 | 语义 |
|------|------|------|
| 蓝 `#58C4DD` | `ACCENT_BLUE` | 主要元素 |
| 黄 `#FFFF00` | `ACCENT_YELLOW` | 高亮/强调 |
| 绿 `#83C167` | `ACCENT_GREEN` | 正确/结论 |
| 红 `#FC6255` | `ACCENT_RED` | 关键/警告 |
| 紫 `#9A72AC` | `ACCENT_PURPLE` | Value/辅助 |
| 青 `#5CD0B3` | `ACCENT_TEAL` | 次要分组 |
| 橙 `#FF8C00` | `ACCENT_ORANGE` | 大于/对比 |

### 透明度三层

| 层 | 值 | 用途 |
|----|------|------|
| 主要 | `OP_PRIMARY=0.9` | 当前操作元素 |
| 上下文 | `OP_CONTEXT=0.4` | 已完成/背景 |
| 结构 | `OP_STRUCTURE=0.15` | 网格/坐标轴 |

### 动画时间表

| 类型 | run_time | wait after |
|------|----------|-----------|
| 标注 | `T_LABEL=0.5s` | `W_BRIEF=0.5s` |
| 常规 | `T_NORMAL=1.0s` | `W_STEP=1.0s` |
| 关键 | `T_KEY=1.5s` | `W_THINK=1.5s` |
| 标题 | `T_TITLE=1.8s` | `W_STEP` |
| 重要公式 | `T_AHA=2.5s` | `W_AHA=2.5s` |

**铁律**: 每个 `self.play()` 后必须有 `self.wait()`。

### 布局

- **禁止手算坐标**，用 `arrange()` + `next_to()` + `to_edge(buff=SAFE_MARGIN)`
- 文字工厂 `styled_*()` 自动限宽
- 组件 (`BarChart/HeatMap/FlowDiagram`) 超宽自动缩放
- 重要时刻 `zoom_in`，总结时 `zoom_out`

### LaTeX

- 永远 raw string: `MathTex(r"\frac{1}{2}")`
- 中文不放 `\text{}`，用 `Text()`
- 公式不跨参数拆分 `\frac{}`

### 叙事结构

```python
def construct(self):
    self.opening()      # 标题 + 核心问题
    self.intuition()    # 直觉/类比 (geometry before algebra)
    self.step_1()       # 技术细节
    self.conclusion()   # 总结
```

### 字幕

```python
self.caption("这是注意力机制的核心", duration=2)
# 或直接:
self.add_subcaption("This is the core of attention", duration=2)
```

渲染后自动生成 `.srt` 文件。

### 可用组件

| 组件 | 用途 |
|------|------|
| `BarChart(data)` | 柱状图 + baseline + 弧形 swap |
| `HeatMap(values, row_labels, col_labels)` | 热力图 |
| `FlowDiagram([(text, color), ...])` | 流程图 |
| `glow_rect(w, h, color)` | 发光框 |
| `dim_rect(w, h)` | 暗色结构框 |
| `styled_title/subtitle/body/label/small/formula` | 文字工厂 |

### 高级动画方法 (StyledScene)

| 方法 | 效果 | 适用 |
|------|------|------|
| `bounce_in(mob)` | ease_out_back 弹性入场 | 标题/重要元素出场 |
| `snap_in(mob)` | 0.3x→1x 缩放弹入 | 结论/结果弹出 |
| `arc_move(mob, pos)` | 弧形移动 (path_arc) | 交换/转移 |
| `morph(src, tgt)` | ReplacementTransform+弧线 | 公式变形 |
| `wave_highlight(group)` | 逐个 Indicate 波浪 | 强调一组元素 |
| `typewriter(text)` | 打字机效果 | 代码/终端文字 |
| `focus_then_restore(mob)` | zoom in→停留→zoom out | 聚焦关键细节 |

### 高级缓动预设

| 预设 | 效果 | 用法 |
|------|------|------|
| `EASE_SPRING` | 弹簧感(超调回弹) | `rate_func=EASE_SPRING` |
| `EASE_SNAP` | 快速卡入 | `rate_func=EASE_SNAP` |
| `EASE_GENTLE` | 极柔和 | `rate_func=EASE_GENTLE` |
| `ease_out_back` | Manim 内置弹性 | `rate_func=ease_out_back` |
| `ease_out_bounce` | 弹跳落地 | `rate_func=ease_out_bounce` |
| `there_and_back` | 去了又回来 | 临时强调 |

### 创建动画选用指南

| 元素 | 动画 | 不要用 |
|------|------|--------|
| 文字/公式 | `Write(text)` | `Create` (太慢) |
| 几何形状 | `Create(shape)` 或 `DrawBorderThenFill(shape)` | `FadeIn` (太平) |
| 快速引入 | `FadeIn(mob, shift=UP)` | 无方向的 `FadeIn` |
| 弹性出场 | `GrowFromCenter(mob)` 或 `bounce_in` | |
| 退出 | 和入场匹配: Create↔Uncreate, FadeIn↔FadeOut | |

### 视觉增强技巧 (来自 manimce-best-practices)

**连接线:**
```python
CurvedArrow(start, end, angle=PI/2)   # 弧形箭头 (比直线更有流动感)
DashedLine(start, end, dash_length=0.2) # 虚线 (辅助/参考线)
Brace(mob, DOWN); brace.get_text("Width") # 大括号标注尺寸
always_redraw(lambda: Line(a.get_center(), b.get_center())) # 动态连线
```

**文字强调:**
```python
Circumscribe(mob, color=YELLOW)         # 画圈强调
Indicate(mob, scale_factor=1.2)          # 脉冲强调
AddTextLetterByLetter(text, time_per_char=0.05) # 打字机
BackgroundRectangle(text, fill_opacity=0.8, buff=0.1) # 文字背景框
```

**函数图/数据可视化:**
```python
graph = axes.plot(lambda x: x**2, color=BLUE)
axes.get_graph_label(graph, MathTex("y=x^2"), x_val=2, direction=UR)
axes.get_area(graph, x_range=[0,2], color=BLUE, opacity=0.3) # 曲线下面积
x_tracker = ValueTracker(0)
dot = always_redraw(lambda: Dot(axes.i2gp(x_tracker.get_value(), graph)))
self.play(x_tracker.animate.set_value(5), run_time=3) # 沿曲线跟踪点
```

**公式变形:**
```python
TransformMatchingTex(eq1, eq2)           # 公式过渡 (匹配相同部分)
TransformMatchingShapes(source, target)  # 形状匹配变形
```

### 渲染后自检

渲染完成后自动抽取 6 帧 PNG 到 `_frames/` 目录。写场景后应:
1. 渲染 (`-ql` 快速预览)
2. 用 Read 工具查看 `_frames/*.png` 检查布局
3. 发现问题 → 改对应 scene → 只重渲该 scene
4. 确认无误 → 渲染 `-qm` 正式版

## 已知陷阱 (踩坑经验)

### API 兼容

- `ease_out_back`, `ease_out_bounce` 等 CSS 风格缓动在 ManimCE 0.20 **不存在**。用 `smooth`、`there_and_back`、`rush_from`，或自定义 `make_bezier_rate_func()`
- `AddTextLetterByLetter` 只能用于 `Text` 对象，**不能用于 `MathTex`**
- 中文不能放在 LaTeX 的 `\text{}` 里（LaTeX 不支持 CJK），用单独的 `Text()` 放在旁边

### 布局定位

- **公式不要用 `to_edge(UP)`** — 会和标题重叠。用 `next_to(title, DOWN, buff=0.5)` 相对定位
- **图例先定位再出现** — 不要先 FadeIn 到错误位置再 `animate.to_corner`，应该创建时直接 `to_corner(UR)` 再 FadeIn
- **图例加 BackgroundRectangle** — 否则和曲线/网格重叠时看不清
- **角度/距离标签不要堆在原点** — 用 `next_to(arc 中点, 外向方向)` 分散放置，或沿半径方向偏移

### 相机运动

- **zoom in 前先 FadeOut step 标签** — 否则标签被裁掉
- **zoom out 后再显示全局信息** — 如右侧面板、底部总结
- **zoom 用 `save_state()` + `Restore()`** — 不要手动 `.scale(1/0.7)` 回去

### 场景过渡

- **不要用全黑空帧过渡** — 内容清完后立即开始下一段，用 `FadeOut(VGroup(...))` 一次性淡出
- **退出动画匹配入场** — `Create` 配 `Uncreate`，`FadeIn` 配 `FadeOut`，`GrowFromCenter` 配 `ShrinkToCenter`

### 帧自检流程 (铁律)

每次渲染完**必须**抽帧检查：
1. `--stitch` 自动抽 8 帧到 `_frames/`
2. 用 Read 逐帧检查：有没有重叠？溢出？空帧？颜色对不对？
3. 发现问题 → 只改对应 Scene 的代码
4. 只重渲该 Scene → 重新 ffmpeg 拼接
5. 再次抽帧确认修复

```bash
# 只重渲 Scene3
python -m manim render -qm script.py Scene3_RoPERotation
# 重新拼接
ffmpeg -y -f concat -safe 0 -i concat.txt -c copy final.mp4
```

## 注意事项

- Windows: `python`, 不是 `python3`
- 中文: `styled_*()` 自动限宽
- 详细 Manim API: 参考 `manimce-best-practices` skill 的 21 个规则文件
