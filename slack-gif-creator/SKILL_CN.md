---
name: slack-gif-creator
description: 用于创建针对Slack优化的动画GIF工具包，包含尺寸约束验证器和可组合的动画基元。当用户请求为Slack创建动画GIF或表情符号动画时应用此技能。
license: 完整条款见LICENSE.txt
---

# Slack GIF创建器 - 灵活工具包

用于创建针对Slack优化的动画GIF的工具包。提供Slack约束的验证器、可组合的动画基元和可选的辅助工具。**根据需要应用这些工具来实现创意愿景。**

## Slack的要求

Slack根据GIF的用途有特定要求：

**消息GIF：**
- 最大尺寸：约2MB
- 最佳尺寸：480x480
- 典型帧率：15-20 FPS
- 颜色限制：128-256色
- 时长：2-5秒

**表情符号GIF：**
- 最大尺寸：64KB（严格限制）
- 最佳尺寸：128x128
- 典型帧率：10-12 FPS
- 颜色限制：32-48色
- 时长：1-2秒

**表情符号GIF具有挑战性** - 64KB的限制非常严格。有帮助的策略：
- 限制为10-15帧总数
- 最多使用32-48种颜色
- 保持设计简单
- 避免渐变
- 频繁验证文件大小

## 工具包结构

此技能提供三种类型的工具：

1. **验证器** - 检查GIF是否符合Slack要求
2. **动画基元** - 用于运动效果的可组合构建块（抖动、弹跳、移动、万花筒）
3. **辅助工具** - 常见需求的可选函数（文本、颜色、特效）

**在如何应用这些工具方面具有完全的创作自由。**

## 核心验证器

为确保GIF符合Slack约束，使用这些验证器：

```python
from core.gif_builder import GIFBuilder

# 创建GIF后，检查是否符合要求
builder = GIFBuilder(width=128, height=128, fps=10)
# ... 以任何方式添加帧 ...

# 保存并检查大小
info = builder.save('emoji.gif', num_colors=48, optimize_for_emoji=True)

# save方法在文件超出限制时会自动警告
# info字典包含：size_kb, size_mb, frame_count, duration_seconds
```

**文件大小验证器**：
```python
from core.validators import check_slack_size

# 检查GIF是否符合大小限制
passes, info = check_slack_size('emoji.gif', is_emoji=True)
# 返回：(True/False, 包含大小详情的字典)
```

**尺寸验证器**：
```python
from core.validators import validate_dimensions

# 检查尺寸
passes, info = validate_dimensions(128, 128, is_emoji=True)
# 返回：(True/False, 包含尺寸详情的字典)
```

**完整验证**：
```python
from core.validators import validate_gif, is_slack_ready

# 运行所有验证
all_pass, results = validate_gif('emoji.gif', is_emoji=True)

# 或快速检查
if is_slack_ready('emoji.gif', is_emoji=True):
    print("准备上传！")
```

## 动画基元

这些是用于运动效果的可组合构建块。以任何组合方式应用于任何对象：

### 抖动
```python
from templates.shake import create_shake_animation

# 抖动表情符号
frames = create_shake_animation(
    object_type='emoji',
    object_data={'emoji': '😱', 'size': 80},
    num_frames=20,
    shake_intensity=15,
    direction='both'  # 或 'horizontal', 'vertical'
)
```

### 弹跳
```python
from templates.bounce import create_bounce_animation

# 弹跳圆圈
frames = create_bounce_animation(
    object_type='circle',
    object_data={'radius': 40, 'color': (255, 100, 100)},
    num_frames=30,
    bounce_height=150
)
```

### 旋转
```python
from templates.spin import create_spin_animation, create_loading_spinner

# 顺时针旋转
frames = create_spin_animation(
    object_type='emoji',
    object_data={'emoji': '🔄', 'size': 100},
    rotation_type='clockwise',
    full_rotations=2
)

# 摇摆旋转
frames = create_spin_animation(rotation_type='wobble', full_rotations=3)

# 加载旋转器
frames = create_loading_spinner(spinner_type='dots')
```

### 脉动/心跳
```python
from templates.pulse import create_pulse_animation, create_attention_pulse

# 平滑脉动
frames = create_pulse_animation(
    object_data={'emoji': '❤️', 'size': 100},
    pulse_type='smooth',
    scale_range=(0.8, 1.2)
)

# 心跳（双泵）
frames = create_pulse_animation(pulse_type='heartbeat')

# 表情符号GIF的注意脉动
frames = create_attention_pulse(emoji='⚠️', num_frames=20)
```

### 淡入淡出
```python
from templates.fade import create_fade_animation, create_crossfade

# 淡入
frames = create_fade_animation(fade_type='in')

# 淡出
frames = create_fade_animation(fade_type='out')

# 两个表情符号之间的交叉淡入淡出
frames = create_crossfade(
    object1_data={'emoji': '😊', 'size': 100},
    object2_data={'emoji': '😂', 'size': 100}
)
```

### 缩放
```python
from templates.zoom import create_zoom_animation, create_explosion_zoom

# 戏剧性放大
frames = create_zoom_animation(
    zoom_type='in',
    scale_range=(0.1, 2.0),
    add_motion_blur=True
)

# 缩小
frames = create_zoom_animation(zoom_type='out')

# 爆炸缩放
frames = create_explosion_zoom(emoji='💥')
```

### 爆炸/粉碎
```python
from templates.explode import create_explode_animation, create_particle_burst

# 爆发爆炸
frames = create_explode_animation(
    explode_type='burst',
    num_pieces=25
)

# 粉碎效果
frames = create_explode_animation(explode_type='shatter')

# 溶解成粒子
frames = create_explode_animation(explode_type='dissolve')

# 粒子爆发
frames = create_particle_burst(particle_count=30)
```

### 摆动/抖动
```python
from templates.wiggle import create_wiggle_animation, create_excited_wiggle

# 果冻摆动
frames = create_wiggle_animation(
    wiggle_type='jello',
    intensity=1.0,
    cycles=2
)

# 波浪运动
frames = create_wiggle_animation(wiggle_type='wave')

# 表情符号GIF的兴奋摆动
frames = create_excited_wiggle(emoji='🎉')
```

### 滑动
```python
from templates.slide import create_slide_animation, create_multi_slide

# 从左侧滑入并过冲
frames = create_slide_animation(
    direction='left',
    slide_type='in',
    overshoot=True
)

# 横向滑动
frames = create_slide_animation(direction='left', slide_type='across')

# 多个对象顺序滑入
objects = [
    {'data': {'emoji': '🎯', 'size': 60}, 'direction': 'left', 'final_pos': (120, 240)},
    {'data': {'emoji': '🎪', 'size': 60}, 'direction': 'right', 'final_pos': (240, 240)}
]
frames = create_multi_slide(objects, stagger_delay=5)
```

### 翻转
```python
from templates.flip import create_flip_animation, create_quick_flip

# 两个表情符号之间的水平翻转
frames = create_flip_animation(
    object1_data={'emoji': '😊', 'size': 120},
    object2_data={'emoji': '😂', 'size': 120},
    flip_axis='horizontal'
)

# 垂直翻转
frames = create_flip_animation(flip_axis='vertical')

# 表情符号GIF的快速翻转
frames = create_quick_flip('👍', '👎')
```

### 变形/变换
```python
from templates.morph import create_morph_animation, create_reaction_morph

# 交叉淡入淡出变形
frames = create_morph_animation(
    object1_data={'emoji': '😊', 'size': 100},
    object2_data={'emoji': '😂', 'size': 100},
    morph_type='crossfade'
)

# 缩放变形（一个缩小同时另一个放大）
frames = create_morph_animation(morph_type='scale')

# 旋转变形（类似3D翻转）
frames = create_morph_animation(morph_type='spin_morph')
```

### 移动效果
```python
from templates.move import create_move_animation

# 线性移动
frames = create_move_animation(
    object_type='emoji',
    object_data={'emoji': '🚀', 'size': 60},
    start_pos=(50, 240),
    end_pos=(430, 240),
    motion_type='linear',
    easing='ease_out'
)

# 弧线移动（抛物线轨迹）
frames = create_move_animation(
    object_type='emoji',
    object_data={'emoji': '⚽', 'size': 60},
    start_pos=(50, 350),
    end_pos=(430, 350),
    motion_type='arc',
    motion_params={'arc_height': 150}
)

# 圆形移动
frames = create_move_animation(
    object_type='emoji',
    object_data={'emoji': '🌍', 'size': 50},
    motion_type='circle',
    motion_params={
        'center': (240, 240),
        'radius': 120,
        'angle_range': 360  # 完整圆圈
    }
)

# 波浪移动
frames = create_move_animation(
    motion_type='wave',
    motion_params={
        'wave_amplitude': 50,
        'wave_frequency': 2
    }
)

# 或使用低级缓动函数
from core.easing import interpolate, calculate_arc_motion

for i in range(num_frames):
    t = i / (num_frames - 1)
    x = interpolate(start_x, end_x, t, easing='ease_out')
    # 或：x, y = calculate_arc_motion(start, end, height, t)
```

### 万花筒效果
```python
from templates.kaleidoscope import apply_kaleidoscope, create_kaleidoscope_animation

# 应用于单帧
kaleido_frame = apply_kaleidoscope(frame, segments=8)

# 或创建动画万花筒
frames = create_kaleidoscope_animation(
    base_frame=my_frame,  # 或None用于演示模式
    num_frames=30,
    segments=8,
    rotation_speed=1.0
)

# 简单镜像效果（更快）
from templates.kaleidoscope import apply_simple_mirror

mirrored = apply_simple_mirror(frame, mode='quad')  # 4向镜像
# 模式：'horizontal', 'vertical', 'quad', 'radial'
```

**要自由组合基元，请遵循以下模式：**
```python
# 示例：弹跳+抖动实现冲击效果
for i in range(num_frames):
    frame = create_blank_frame(480, 480, bg_color)

    # 弹跳运动
    t_bounce = i / (num_frames - 1)
    y = interpolate(start_y, ground_y, t_bounce, 'bounce_out')

    # 在冲击时添加抖动（当y到达地面时）
    if y >= ground_y - 5:
        shake_x = math.sin(i * 2) * 10
        x = center_x + shake_x
    else:
        x = center_x

    draw_emoji(frame, '⚽', (x, y), size=60)
    builder.add_frame(frame)
```

## 辅助工具

这些是常见需求的可选辅助工具。**根据需要使用、修改或用自定义实现替换。**

### GIF构建器（组装与优化）

```python
from core.gif_builder import GIFBuilder

# 使用您选择的设置创建构建器
builder = GIFBuilder(width=480, height=480, fps=20)

# 添加帧（无论您如何创建它们）
for frame in my_frames:
    builder.add_frame(frame)

# 保存并优化
builder.save('output.gif',
             num_colors=128,
             optimize_for_emoji=False)
```

关键特性：
- 自动颜色量化
- 重复帧移除
- Slack限制的大小警告
- 表情符号模式（激进优化）

### 文本渲染

对于像表情符号这样的小GIF，文本可读性具有挑战性。常见解决方案涉及添加轮廓：

```python
from core.typography import draw_text_with_outline, TYPOGRAPHY_SCALE

# 带轮廓的文本（有助于可读性）
draw_text_with_outline(
    frame, "BONK!",
    position=(240, 100),
    font_size=TYPOGRAPHY_SCALE['h1'],  # 60px
    text_color=(255, 68, 68),
    outline_color=(0, 0, 0),
    outline_width=4,
    centered=True
)
```

要实现自定义文本渲染，使用PIL的`ImageDraw.text()`，对于较大的GIF效果很好。

### 颜色管理

专业外观的GIF通常使用协调的调色板：

```python
from core.color_palettes import get_palette

# 获取预定义调色板
palette = get_palette('vibrant')  # 或 'pastel', 'dark', 'neon', 'professional'

bg_color = palette['background']
text_color = palette['primary']
accent_color = palette['accent']
```

要直接处理颜色，使用RGB元组 - 任何适用于用例的方法都可以。

### 视觉效果

用于冲击时刻的可选效果：

```python
from core.visual_effects import ParticleSystem, create_impact_flash, create_shockwave_rings

# 粒子系统
particles = ParticleSystem()
particles.emit_sparkles(x=240, y=200, count=15)
particles.emit_confetti(x=240, y=200, count=20)

# 每帧更新和渲染
particles.update()
particles.render(frame)

# 闪光效果
frame = create_impact_flash(frame, position=(240, 200), radius=100)

# 冲击波环
frame = create_shockwave_rings(frame, position=(240, 200), radii=[30, 60, 90])
```

### 缓动函数

平滑运动使用缓动而不是线性插值：

```python
from core.easing import interpolate

# 物体下落（加速）
y = interpolate(start=0, end=400, t=progress, easing='ease_in')

# 物体着陆（减速）
y = interpolate(start=0, end=400, t=progress, easing='ease_out')

# 弹跳
y = interpolate(start=0, end=400, t=progress, easing='bounce_out')

# 过冲（弹性）
scale = interpolate(start=0.5, end=1.0, t=progress, easing='elastic_out')
```

可用的缓动：`linear`, `ease_in`, `ease_out`, `ease_in_out`, `bounce_out`, `elastic_out`, `back_out`（过冲），以及`core/easing.py`中的更多选项。

### 帧合成

如果需要的基本绘图工具：

```python
from core.frame_composer import (
    create_gradient_background,  # 渐变背景
    draw_emoji_enhanced,         # 带可选阴影的表情符号
    draw_circle_with_shadow,     # 带深度的形状
    draw_star                    # 五角星
)

# 渐变背景
frame = create_gradient_background(480, 480, top_color, bottom_color)

# 带阴影的表情符号
draw_emoji_enhanced(frame, '🎉', position=(200, 200), size=80, shadow=True)
```

## 优化策略

当GIF太大时：

**对于消息GIF（>2MB）：**
1. 减少帧数（降低FPS或缩短时长）
2. 减少颜色（128 → 64色）
3. 减小尺寸（480x480 → 320x320）
4. 启用重复帧移除

**对于表情符号GIF（>64KB） - 要激进：**
1. 限制为10-12帧总数
2. 最多使用32-40种颜色
3. 避免渐变（纯色压缩效果更好）
4. 简化设计（更少元素）
5. 在save方法中使用`optimize_for_emoji=True`

## 示例组合模式

### 简单反应（脉动）
```python
builder = GIFBuilder(128, 128, 10)

for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))

    # 脉动缩放
    scale = 1.0 + math.sin(i * 0.5) * 0.15
    size = int(60 * scale)

    draw_emoji_enhanced(frame, '😱', position=(64-size//2, 64-size//2),
                       size=size, shadow=False)
    builder.add_frame(frame)

builder.save('reaction.gif', num_colors=40, optimize_for_emoji=True)

# 验证
from core.validators import check_slack_size
check_slack_size('reaction.gif', is_emoji=True)
```

### 带冲击的动作（弹跳+闪光）
```python
builder = GIFBuilder(480, 480, 20)

# 阶段1：物体下落
for i in range(15):
    frame = create_gradient_background(480, 480, (240, 248, 255), (200, 230, 255))
    t = i / 14
    y = interpolate(0, 350, t, 'ease_in')
    draw_emoji_enhanced(frame, '⚽', position=(220, int(y)), size=80)
    builder.add_frame(frame)

# 阶段2：冲击+闪光
for i in range(8):
    frame = create_gradient_background(480, 480, (240, 248, 255), (200, 230, 255))

    # 在前几帧上闪光
    if i < 3:
        frame = create_impact_flash(frame, (240, 350), radius=120, intensity=0.6)

    draw_emoji_enhanced(frame, '⚽', position=(220, 350), size=80)

    # 文本出现
    if i > 2:
        draw_text_with_outline(frame, "进球！", position=(240, 150),
                              font_size=60, text_color=(255, 68, 68),
                              outline_color=(0, 0, 0), outline_width=4, centered=True)

    builder.add_frame(frame)

builder.save('goal.gif', num_colors=128)
```

### 组合基元（移动+抖动）
```python
from templates.shake import create_shake_animation

# 创建抖动动画
shake_frames = create_shake_animation(
    object_type='emoji',
    object_data={'emoji': '😰', 'size': 70},
    num_frames=20,
    shake_intensity=12
)

# 创建触发抖动的移动元素
builder = GIFBuilder(480, 480, 20)
for i in range(40):
    t = i / 39

    if i < 20:
        # 触发前 - 使用带移动对象的空白帧
        frame = create_blank_frame(480, 480, (255, 255, 255))
        x = interpolate(50, 300, t * 2, 'linear')
        draw_emoji_enhanced(frame, '🚗', position=(int(x), 300), size=60)
        draw_emoji_enhanced(frame, '😰', position=(350, 200), size=70)
    else:
        # 触发后 - 使用抖动帧
        frame = shake_frames[i - 20]
        # 在最终位置添加汽车
        draw_emoji_enhanced(frame, '🚗', position=(300, 300), size=60)

    builder.add_frame(frame)

builder.save('scare.gif')
```

## 理念

此工具包提供构建块，而不是严格的配方。处理GIF请求时：

1. **理解创意愿景** - 应该发生什么？氛围如何？
2. **设计动画** - 将其分解为阶段（预期、动作、反应）
3. **根据需要应用基元** - 抖动、弹跳、移动、效果 - 自由混合
4. **验证约束** - 检查文件大小，特别是表情符号GIF
5. **如有需要则迭代** - 如果超过大小限制则减少帧数/颜色

**目标是在Slack技术约束内的创作自由。**

## 依赖项

要使用此工具包，如果这些依赖项尚未存在，请安装：

```bash
pip install pillow imageio numpy
```