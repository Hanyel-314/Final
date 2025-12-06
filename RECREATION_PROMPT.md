# 迷你游戏大厅 - 完整创建指南

## 项目概述

创建一个专业级的迷你游戏大厅网页应用，包含5个独特的小游戏，每个游戏都有创新的解锁机制。整个项目使用单个HTML文件实现，包含所有CSS和JavaScript代码。

## 技术要求

- **文件格式**：单个 `index.html` 文件
- **技术栈**：纯HTML5 + CSS3 + JavaScript (ES6+)
- **Canvas尺寸**：1200 x 700 像素（固定）
- **渲染**：使用Canvas 2D API
- **动画**：requestAnimationFrame游戏循环

## 视觉风格

### 整体配色方案
- **背景渐变**：深色调（#1a1a2e → #16213e → #0f3460）
- **主标题**：白色，粗体48px，"Mini Game Lab - Professional Edition"
- **副标题**：半透明白色，18px
- **背景装饰**：80个随机星星，闪烁动画（CSS keyframes）

### 画布背景
- **中央平台**：900x500px圆角矩形（radius: 20px）
- 颜色：rgba(30, 30, 60, 0.6)
- 阴影：蓝紫色光晕（rgba(99, 102, 241, 0.3)，blur: 30px）

### 底部状态栏
- 位置：距底部70px
- 尺寸：宽度为canvas宽度-200px，高度50px
- 背景：rgba(0, 0, 0, 0.4)
- 文字：16px，白色，居中

### 帮助按钮
- 位置：右上角（x: CANVAS_WIDTH - 70, y: 30）
- 样式：圆形，半径20px
- 颜色：悬停时为#6366f1，否则为半透明
- 内容：白色问号"?"（24px粗体）

## 游戏大厅设计

### 5个游戏图标布局

**圆形排列**：
- 中心点：(CANVAS_WIDTH/2, CANVAS_HEIGHT/2 + 20)
- 半径：180px
- 图标半径：55px
- 起始角度：-Math.PI/2（顶部）
- 间隔：72度（Math.PI * 2 / 5）

**图标配置**：

1. **Lightning Reaction（⚡ 闪电反应）**
   - 位置：顶部（12点钟方向）
   - 颜色：#fbbf24（金黄色）
   - 符号：闪电emoji ⚡
   - 提示：'Hover to charge the orb to 100%'

2. **Fruit Slice（🍉 切水果）**
   - 位置：右上（2点钟方向）
   - 颜色：#ef4444（红色）
   - 提示：'Drag the knife to slice the fruit'

3. **Maze Runner（🏃 迷宫挑战）**
   - 位置：右下（4点钟方向）
   - 颜色：#6366f1（蓝紫色）
   - 提示：'Slide the handle all the way to the right'

4. **Memory Match（🧠 记忆翻牌）**
   - 位置：左下（8点钟方向）
   - 颜色：#a855f7（紫色）
   - 提示：'Type the magic word: MEMORY'

5. **Space Dodge（🚀 太空躲避）**
   - 位置：左上（10点钟方向）
   - 颜色：#3b82f6（蓝色）
   - 提示：'Drag the planet in a full circle (360°)'

### 图标动画效果
- **悬停缩放**：scale从1.0到1.1，lerp平滑过渡（速度0.2）
- **发光效果**：shadowBlur: 30px，颜色为图标主色
- **解锁后颜色**：变为绿色#10b981
- **解锁后文字**：图标下方显示"Click to Start"（绿色，12px粗体）

## 五种解锁机制详细实现

### 1. 充能解锁（Game 1 - Lightning）

**机制**：
- 初始能量：0%
- 悬停充能速度：+1.5/帧
- 离开衰减速度：-0.8/帧
- 解锁阈值：100%

**视觉反馈**：
- 进度条：图标下方100px宽，12px高
- 进度条背景：rgba(0, 0, 0, 0.4)
- 进度条填充：金黄色#fbbf24
- 状态文字：`Charging... ${Math.floor(energy)}%`

**实现要点**：
```javascript
if (pointInCircle(mouseX, mouseY, icon1.x, icon1.y, icon1.radius)) {
    hub.game1.energy = Math.min(100, hub.game1.energy + 1.5);
    hub.statusText = `Charging... ${Math.floor(hub.game1.energy)}%`;
} else {
    hub.game1.energy = Math.max(0, hub.game1.energy - 0.8);
}
```

### 2. 拖拽切割解锁（Game 2 - Fruit）

**机制**：
- 小刀初始位置：水果图标右侧+80px
- 拖拽小刀到水果上释放即解锁
- 释放后播放切割动画

**小刀绘制**：
- 刀柄：棕色#92400e，16x25px圆角矩形
- 刀刃：银灰色#d1d5db，三角形（顶点-25px）
- 拖拽时旋转45度并放大1.1倍

**切割动画**：
- 水果分裂成两半
- 分离动画：sliceAnim从0到1，速度0.02
- 两半分别向对角移动offset = sliceAnim * 30

**实现要点**：
```javascript
// 拖拽检测
if (pointInCircle(mouseX, mouseY, knife.x, knife.y, 25)) {
    knife.dragging = true;
}
// 释放检测
if (knife.dragging && mouseup) {
    if (pointInCircle(knife.x, knife.y, icon2.x, icon2.y, radius + 20)) {
        hub.game2.sliced = true;
    }
    knife.x = knife.homeX;
    knife.y = knife.homeY;
}
```

### 3. 滑动解锁（Game 3 - Door）

**机制**：
- 滑块轨道：水平线，长度120px
- 位置：门图标上方80px
- 滑块位置：0到1（百分比）
- 解锁阈值：≥0.99

**门的绘制**：
- 门框：70x100px，描边4px，颜色#6366f1
- 门板：填充#4338ca，中央分割线#312e81
- 开门动画：两扇门向两侧分离，使用easeOutQuad缓动

**滑块绘制**：
- 轨道：3px粗线，颜色#6366f1
- 滑块：圆形，半径12px
- 颜色：拖拽时#fbbf24，否则#8b5cf6
- 白色描边2px

**实现要点**：
```javascript
if (sliderDragging) {
    const localX = mouseX - (icon3.x - 60);
    hub.game3.sliderPos = Math.max(0, Math.min(1, localX / 120));
}
if (hub.game3.sliderPos >= 0.99 && doorAnim < 1) {
    hub.game3.doorAnim += 0.02;
    // 开门动画：offset = easeOutQuad(doorAnim) * (doorW/2 + 5)
}
```

### 4. 密码输入解锁（Game 4 - Book）

**机制**：
- 魔法书：80x60px紫色圆角矩形
- 书脊：中央10px宽深紫色
- 火花装饰：两个✨emoji在对角
- 浮动效果：sin(time * 0.002) * 5

**输入框**：
- 位置：书下方
- 尺寸：140x30px
- 提示文字："Type: MEMORY"（紫色，14px等宽字体）
- 输入框背景：rgba(0, 0, 0, 0.6)
- 边框：激活时金色#fbbf24，否则#666
- 光标闪烁：500ms切换（|符号）

**解锁按钮**：
- 位置：输入框下方10px
- 尺寸：140x28px
- 正确时绿色#10b981，否则灰色半透明
- 错误时书本抖动（shake值10帧衰减）

**实现要点**：
```javascript
// 键盘监听
if (hub.game4.inputActive) {
    if (e.key === 'Backspace') inputText = inputText.slice(0, -1);
    else if (e.key === 'Enter') checkPassword();
    else if (e.key.length === 1) inputText += e.key.toUpperCase();
}
// 正确密码
if (inputText === 'MEMORY') hub.game4.unlocked = true;
```

### 5. 旋转解锁（Game 5 - Planet）

**机制**：
- 轨道半径：70px
- 星球半径：55 * 0.7 = 38.5px
- 累计旋转进度：0到1（360度）
- 使用atan2计算角度差

**轨道绘制**：
- 虚线圆：rgba(59, 130, 246, 0.3)，虚线[5, 5]
- 进度弧：金色#fbbf24，4px粗

**星球绘制**：
- 主体：蓝色#3b82f6圆形
- 陨石坑：深蓝色#1e40af，两个小圆
- 火箭：🚀emoji居中

**实现要点**：
```javascript
if (dragging) {
    const dx = mouseX - icon5.x;
    const dy = mouseY - icon5.y;
    const currentAngle = Math.atan2(dy, dx);
    let angleDiff = currentAngle - lastAngle;
    // 处理角度跳变
    if (angleDiff > Math.PI) angleDiff -= Math.PI * 2;
    if (angleDiff < -Math.PI) angleDiff += Math.PI * 2;
    progress += Math.abs(angleDiff) / (Math.PI * 2);
    lastAngle = currentAngle;
}
```

## 五款游戏详细实现

### Game 1: Lightning Reaction Test（闪电反应测试）

**界面设计**：
- 背景色：#0f0f1e（深蓝黑）
- 标题：金黄色#fbbf24，36px粗体
- 统计信息：分数 | 生命（❤️emoji）| 剩余时间

**游戏逻辑**：
- 6个按钮：2行3列对称布局
- 位置：(CANVAS_WIDTH * [0.25, 0.5, 0.75], CANVAS_HEIGHT * [0.35, 0.65])
- 按钮半径：50px
- 初始反应时间：900ms
- 最小反应时间：400ms
- 每次成功-20ms难度递增

**按钮状态**：
- 未激活：灰色#374151
- 激活：金黄色#fbbf24，发光shadowBlur: 40
- 点击成功：scale放大到1.3然后衰减

**失败条件**：
- 超时未点击：-1生命
- 生命归零：游戏结束
- 时间耗尽：游戏结束

**游戏结束画面**：
- 黑色半透明遮罩
- 金黄色"Game Over!"（56px）
- 最终分数（32px）
- 最佳反应时间

### Game 2: Fruit Slice Master（切水果大师）

**界面设计**：
- 背景色：#fef3c7（浅黄色）
- 标题：棕色#92400e，36px
- 统计：分数/目标 | 剩余时间

**水果生成**：
- 刷新间隔：初始900ms，最小400ms
- 从底部随机x位置向上抛出
- 初始vy：-(10到25)随机
- 横向vx：-2.5到2.5随机
- 重力加速度：+0.6/帧

**水果类型**：
- 普通水果：80%概率
- 颜色：红#ef4444、橙#f59e0b、绿#10b981、蓝#3b82f6、紫#8b5cf6
- 炸弹💣：20%概率，颜色#1f2937

**切割机制**：
- 鼠标按下时记录轨迹（最多25个点）
- 水果碰撞检测：任一轨迹点距水果中心<半径
- 切中水果：+1分，生成12个粒子爆炸
- 切中炸弹：立即游戏结束

**刀光效果**：
- 白色轨迹线，5px粗
- 圆角lineCap: 'round'
- 白色发光shadowBlur: 15

**粒子系统**：
- 12个粒子，随机速度(-5到5, -5到5)
- 重力+0.3/帧
- 生命周期30帧
- alpha随生命衰减

**胜利条件**：
- 达到50分
- 显示"Victory!"绿色文字

### Game 3: Maze Runner Challenge（迷宫挑战）

**界面设计**：
- 背景色：#0f172a（深蓝黑）
- 标题：蓝紫色#6366f1，32px
- 统计：时间 | 重置次数

**迷宫数据**：
- 17x11网格
- 1=墙壁，0=通道，2=终点
- 单元格尺寸：35x35px
- 迷宫居中显示

```javascript
maze = [
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
    [1,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,1],
    [1,0,1,0,1,0,1,1,1,0,1,0,1,1,1,0,1],
    [1,0,1,0,0,0,0,0,0,0,0,0,0,0,1,0,1],
    [1,0,1,1,1,1,1,0,1,1,1,1,1,0,1,0,1],
    [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
    [1,1,1,0,1,1,1,1,1,1,1,0,1,1,1,0,1],
    [1,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1],
    [1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,0,1],
    [1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,2,1],
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
];
```

**玩家**：
- 初始位置：(1.5, 1.5)
- 半径：cellSize * 0.4
- 颜色：蓝色#3b82f6，发光效果
- 移动速度：0.12单位/帧
- 操作：方向键或WASD

**敌人系统**（3个）：
1. 横向巡逻：(7, 3)，速度0.025
2. 纵向巡逻：(5, 7)，速度0.03
3. 对角巡逻：(12, 5)，速度0.02
- 颜色：红色#ef4444，发光
- 碰撞检测：距离<0.6单位
- 碰撞结果：玩家重置到起点，重置次数+1

**墙壁碰撞**：
- 计算玩家网格坐标
- 检查目标格子是否为墙(值为1)
- 是则拒绝移动

**胜利条件**：
- 到达终点格(值为2)
- 显示用时和重置次数

**视觉元素**：
- 墙壁：#475569填充
- 通道：#1e293b填充
- 终点：绿色#10b981 + 旗帜🏁
- 网格线：#334155，1px

### Game 4: Memory Match Master（记忆翻牌大师）

**界面设计**：
- 背景色：#1e293b（深灰蓝）
- 标题：紫色#a855f7，36px
- 统计：步数 | 已配对/总对数 | 剩余时间

**卡片布局**：
- 4x4网格，共16张（8对）
- 卡片尺寸：90x90px
- 间距：15px
- 圆角：10px
- 网格居中

**颜色配对**：
```javascript
colors = [
    '#ef4444', // 红
    '#f59e0b', // 橙
    '#10b981', // 绿
    '#3b82f6', // 蓝
    '#8b5cf6', // 紫
    '#ec4899', // 粉
    '#f97316', // 深橙
    '#06b6d4'  // 青
]
```

**卡片状态**：
1. **hidden**（隐藏）：
   - 背景：#475569
   - 显示问号"?"（40px，灰色#64748b）

2. **shown**（翻开）：
   - 背景：对应颜色
   - 无额外标记

3. **matched**（已配对）：
   - 背景：对应颜色
   - 白色对勾✓（48px）
   - scale放大到1.15

**游戏流程**：
1. 点击翻开第一张卡片
2. 点击翻开第二张卡片，步数+1
3. 等待700ms判断：
   - 颜色相同：标记为matched
   - 颜色不同：翻回hidden
4. 8对全部配对完成则胜利

**星级评定**：
- ≤16步：⭐⭐⭐
- ≤24步：⭐⭐
- >24步：⭐

**胜利画面**：
- 绿色"Perfect Memory!"（56px）
- 金色星星（48px）
- 完成步数（28px）

### Game 5: Space Dodge Extreme（太空躲避极限）

**界面设计**：
- 背景色：#0c0a1f（深空黑）
- 动态星星：40个白点随机透明度，垂直滚动
- 标题：蓝色#3b82f6，36px
- 统计：分数 | 生存时间 | 陨石速度

**飞船**：
- 位置：底部中央(CANVAS_WIDTH/2, CANVAS_HEIGHT - 100)
- 形状：三角形
- 尺寸：50x50px（高度上下各25px）
- 颜色：蓝色#3b82f6，发光
- 驾驶舱：顶部蓝色圆圈（半径10px）
- 引擎：底部两个火焰三角（橙色#f59e0b）

**飞船操作**：
- 左右方向键或A/D键
- 速度：8px/帧
- 移动时倾斜±0.2弧度
- 边界限制：船身不能超出画布

**陨石生成**：
- 初始刷新间隔：700ms
- 最小间隔：300ms
- 从顶部随机x位置生成
- 初始速度：4px/帧，逐渐加速到8px/帧
- 每10秒速度+0.2

**陨石特性**：
- 半径：20-45px随机
- 颜色：棕色#78350f
- 红色发光shadowBlur: 10
- 陨石坑：深棕色#451a03圆点
- 拖尾：红色半透明线条
- 自旋转：随机速度

**碰撞检测**：
- 圆形碰撞：dist(meteor, ship) < meteor.radius + ship.width/2
- 碰撞即游戏结束

**得分机制**：
- 陨石离开屏幕底部：+1分
- 生存时间每秒累计

**游戏结束画面**：
- 红色"Game Over!"（56px）
- 最终分数和生存时间（32px）

## 通用功能实现

### 返回按钮（所有游戏通用）

**位置与样式**：
- 左上角(30, 30)
- 尺寸：140x40px
- 圆角：20px
- 背景：rgba(71, 85, 105, 0.9)
- 白色边框2px
- 文字："← Back to Hub"（白色，16px粗体）

**功能**：
- 点击返回currentScene = 'hub'
- 重置鼠标样式

### 工具函数

```javascript
// 距离计算
function dist(x1, y1, x2, y2) {
    return Math.sqrt((x2-x1)**2 + (y2-y1)**2);
}

// 点在圆内
function pointInCircle(px, py, cx, cy, radius) {
    return dist(px, py, cx, cy) <= radius;
}

// 点在矩形内
function pointInRect(px, py, rx, ry, rw, rh) {
    return px >= rx && px <= rx+rw && py >= ry && py <= ry+rh;
}

// 圆角矩形路径
function roundRect(x, y, w, h, r) {
    ctx.beginPath();
    ctx.moveTo(x+r, y);
    ctx.lineTo(x+w-r, y);
    ctx.quadraticCurveTo(x+w, y, x+w, y+r);
    ctx.lineTo(x+w, y+h-r);
    ctx.quadraticCurveTo(x+w, y+h, x+w-r, y+h);
    ctx.lineTo(x+r, y+h);
    ctx.quadraticCurveTo(x, y+h, x, y+h-r);
    ctx.lineTo(x, y+r);
    ctx.quadraticCurveTo(x, y, x+r, y);
    ctx.closePath();
}

// 缓动函数
function easeOutQuad(t) {
    return t * (2 - t);
}

// 线性插值
function lerp(start, end, t) {
    return start + (end - start) * t;
}
```

### 事件监听器

**鼠标事件**：
- `mousemove`：更新mouseX/mouseY，记录轨迹（game2）
- `mousedown`：设置mousePressed，检测拖拽开始
- `mouseup`：重置mousePressed，检测拖拽结束/释放
- `click`：图标点击、按钮点击、卡片翻转

**键盘事件**：
- `keydown`：
  - hub场景 + inputActive：处理输入框文字
  - game3：设置keys[e.key] = true
  - game5：设置keys[e.key] = true
- `keyup`：
  - game3/game5：设置keys[e.key] = false

### 场景状态机

```javascript
let currentScene = 'hub'; // 'hub', 'game1', 'game2', 'game3', 'game4', 'game5'

function gameLoop() {
    switch (currentScene) {
        case 'hub':
            updateHub();
            drawHub();
            break;
        case 'game1':
            updateGame1();
            drawGame1();
            break;
        // ... 其他场景
    }
    requestAnimationFrame(gameLoop);
}
```

## 性能优化建议

1. **Canvas渲染**：
   - 使用requestAnimationFrame
   - 每帧清空canvas后重绘

2. **对象池**：
   - 粒子系统使用数组过滤
   - 及时清理离屏对象

3. **事件处理**：
   - 使用节流避免过度计算
   - 统一管理鼠标/键盘状态

4. **阴影效果**：
   - 只在需要时开启shadowBlur
   - 绘制后立即重置shadowBlur = 0

## 调试提示

1. **控制台打印关键状态**：
   - 解锁进度
   - 碰撞检测结果
   - 场景切换

2. **调整难度参数**：
   - 反应时间、生成间隔可调
   - 方便测试胜利/失败画面

3. **边界情况**：
   - 测试所有解锁机制
   - 验证游戏结束条件
   - 检查边界碰撞

## README文件内容

需要创建一个中文README.md，包含：
- 项目简介
- 在线体验方式（GitHub Pages）
- 5种解锁方式说明表格
- 5款游戏玩法说明
- 操作指南（键盘/鼠标）
- 技术特点列举
- 部署步骤
- 设计理念

## 最终检查清单

✅ 所有代码在单个HTML文件中
✅ Canvas尺寸固定1200x700
✅ 5个图标圆形排列，间距均匀
✅ 5种解锁机制互不相同且有趣
✅ 5款游戏类型多样化
✅ 所有游戏有明确的胜利/失败条件
✅ 返回按钮在所有游戏中可用
✅ 视觉风格统一（深色背景+亮色元素）
✅ 动画流畅（缩放、发光、拖尾、粒子）
✅ 无控制台错误
✅ 响应式鼠标悬停效果
✅ README文档完整

## 实现顺序建议

1. **第一阶段**：HTML结构和CSS背景星星
2. **第二阶段**：Canvas基础设置、工具函数、场景管理
3. **第三阶段**：游戏大厅UI（平台、标题、状态栏、帮助按钮）
4. **第四阶段**：5个图标圆形布局和基础绘制
5. **第五阶段**：实现5种解锁机制（一个一个实现和测试）
6. **第六阶段**：实现游戏1（Lightning Reaction）
7. **第七阶段**：实现游戏2（Fruit Slice）
8. **第八阶段**：实现游戏3（Maze Runner）
9. **第九阶段**：实现游戏4（Memory Match）
10. **第十阶段**：实现游戏5（Space Dodge）
11. **最后阶段**：返回按钮、事件监听器整合、全面测试、创建README

---

**重要提示**：这个prompt包含了所有必要的细节，包括精确的颜色代码、尺寸、位置、动画参数和游戏逻辑。严格按照这些规格实现可以确保生成与原版完全一致的游戏体验。
