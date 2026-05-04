---
name: algo-viz
description: "Generate algorithm and tech learning animation videos using Manim. Visualize sorting, data structures, graph algorithms, deep learning concepts (Transformer, Attention, Backprop), and math (gradient descent, matrix ops). Use when user asks to visualize, animate, or explain algorithms/concepts with video."
args:
  - name: topic
    description: "The topic to visualize — e.g. '快速排序', 'Self-Attention', 'BFS', '梯度下降', 'Transformer架构'"
    required: true
  - name: quality
    description: "Video quality: l=480p(fast), m=720p, h=1080p, k=4K. Default: m"
    required: false
  - name: data
    description: "Custom input data for the animation, e.g. '[5,3,8,1,9,2]' for sorting"
    required: false
user-invokable: true
---

# algo-viz: 算法与技术学习动画生成器

根据用户提供的主题，生成 **3Blue1Brown / 漫士沉思录 品质** 的教学动画视频。

工具位于 `D:/pythonPycharms/My_jupyerlab/tools/algo-viz/`。

## 工作流程

### 1. 判断是否有预制场景

```bash
python "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/generate.py" --list
```

### 2A. 匹配预制场景 → 直接渲染

```bash
python "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/generate.py" "快速排序" -q m
```

### 2B. 无匹配 → 编写新 Manim Scene

在 `scenes/` 下创建 .py 文件。**必须遵守下方的审美规范。**

渲染：
```bash
python -m manim render -qm --media_dir "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/output" "D:/pythonPycharms/My_jupyerlab/tools/algo-viz/scenes/<filename>.py" <ClassName>
```

## 审美规范 (3B1B Quality Rules) — 写新场景时必须遵守

### 基类与导入

```python
from manim import *
from scenes.style import *

class MyScene(StyledScene):  # 继承 StyledScene (MovingCameraScene)
    def construct(self):
        ...
```

StyledScene 提供: `show_title`, `show_step`, `clear_step`, `transition`, `zoom_in`, `zoom_out`, `auto_fit`, `subtle_pulse`, `flash_highlight`, `indicate`, `staggered_fadein`, `staggered_create`, `show_formula`

### 配色语义 (同一概念全片用同一颜色)

| 颜色 | 常量 | 语义 |
|------|------|------|
| 蓝 `#58C4DD` | `ACCENT_BLUE` | 主要元素/默认/"这是什么" |
| 黄 `#FFFF00` | `ACCENT_YELLOW` | 高亮/强调/"注意看这里" |
| 绿 `#83C167` | `ACCENT_GREEN` | 正确/完成/正面结论 |
| 红 `#FC6255` | `ACCENT_RED` | 关键/基准/需要注意 |
| 紫 `#9A72AC` | `ACCENT_PURPLE` | Value/辅助概念 |
| 青 `#5CD0B3` | `ACCENT_TEAL` | 次要分组/"小于" |
| 橙 `#FF8C00` | `ACCENT_ORANGE` | 警告/"大于" |

### 透明度三层制

| 层 | opacity | 用途 |
|----|---------|------|
| 主要 | `OP_PRIMARY=0.9` | 当前操作的元素 |
| 上下文 | `OP_CONTEXT=0.4` | 已完成/背景参考 |
| 结构 | `OP_STRUCTURE=0.15` | 网格线/辅助线/坐标轴 |

### 动画时间表

| 动画类型 | run_time | wait after |
|----------|----------|-----------|
| 小标注出现 | `T_LABEL=0.5s` | `W_BRIEF=0.5s` |
| 常规动画 | `T_NORMAL=1.0s` | `W_STEP=1.0s` |
| 关键变换 | `T_KEY=1.5s` | `W_THINK=1.5s` |
| 标题 Write | `T_TITLE=1.8s` | `W_STEP=1.0s` |
| 重要公式(aha) | `T_AHA=2.5s` | `W_AHA=2.5s` |

**铁律**: 每个 `self.play()` 后必须有 `self.wait()`。

### 安全边距

- 所有元素到屏幕边缘 `buff >= SAFE_MARGIN (0.5)`
- 文字工厂 `styled_*()` 自动限宽，超长自动缩小
- 组件 `BarChart/HeatMap/FlowDiagram` 超宽自动缩放

### 相机运动

```python
self.zoom_in(important_mob, scale=0.5)   # 聚焦
self.wait(W_THINK)
self.zoom_out()                           # 恢复
self.auto_fit(big_group)                  # 自动缩放适应
```

**原则**: 重要时刻 zoom in，总结时 zoom out。不要全程一个视角。

### rate_func 选择

| 场景 | rate_func |
|------|-----------|
| 大部分动画 | `smooth` (默认，不用写) |
| 匀速运动 | `linear` |
| 强调弹回 | `there_and_back` |
| 弹性出场 | `ease_out_back` |
| 元素交换 | 加 `path_arc=PI/3` |

### 布局规则 — 禁止手算坐标

```python
# ❌ 错误: 手算 x, y
bar.move_to([start_x + i * gap, -2 + h / 2, 0])

# ✅ 正确: arrange + next_to
group.arrange(RIGHT, buff=0.3)
label.next_to(bar, UP, buff=0.12)
section.to_edge(DOWN, buff=SAFE_MARGIN)
```

- 用 `VGroup.arrange()` 排列
- 用 `next_to` / `align_to` 相对定位
- 用 `to_edge` / `to_corner` 锚定屏幕
- 热力图用 `arrange_in_grid`
- 超宽时用 `self.auto_fit(group)` 缩放相机

### 视觉强调

```python
self.indicate(mob)                    # Indicate 脉冲
self.flash_highlight(mob)             # Circumscribe 画圈
self.subtle_pulse(mob, scale=1.05)    # 微妙呼吸感
```

**原则**: 重要结论出现时至少用一种强调动画。

### 叙事结构

每个场景按此结构组织 `construct()`:

```python
def construct(self):
    self.opening()        # 标题 + 核心问题
    self.intuition()      # 直觉/类比 (geometry before algebra)
    self.step_1()         # 技术细节 step by step
    self.step_2()
    self.conclusion()     # 总结/对比表
```

**"Geometry before algebra"** — 先出图形直觉，再出公式。

### LaTeX 规则

- 永远用 raw string: `MathTex(r"\frac{1}{2}")`
- 中文不放 `\text{}` 里 (LaTeX 不支持 CJK)，用单独 `Text()`
- 公式用单一字符串，不要跨参数拆分 `\frac{}`
- 多行公式用 `VGroup(MathTex(...), MathTex(...)).arrange(DOWN)`

### 可用组件

| 组件 | 用途 |
|------|------|
| `BarChart(data)` | 柱状图, 自动排版+baseline+弧形swap |
| `HeatMap(values, row_labels, col_labels)` | 热力图, arrange_in_grid |
| `FlowDiagram([(text, color), ...])` | 流程图, 自动箭头 |
| `glow_rect(w, h, color)` | 发光框 |
| `dim_rect(w, h)` | 暗色结构框 |
| `styled_title/subtitle/body/label/small/formula` | 文字工厂 |

## 输出

视频在 `D:/pythonPycharms/My_jupyerlab/tools/algo-viz/output/`。

默认 720p (`-qm`)。开发调试用 480p (`-ql`)。

## 注意事项

- Windows: `python`, 不是 `python3`
- 中文文字: `styled_*()` 自动处理宽度
- 渲染报错: 先检查 LaTeX 字符串 (是否有中文在 `\text{}` 里)
