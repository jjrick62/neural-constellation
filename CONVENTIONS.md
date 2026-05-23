# 个人主页项目 — 接口规范与命名约定

## 文件结构

单 HTML 文件 `个人主页.html`，所有代码按 `<script>` 块分区：

```html
<!-- ===== 1. 数据层 ===== -->
<script>  // DataModel </script>

<!-- ===== 2. 基础工具层 ===== -->
<script>  // ShaderLib, MathUtils, CurlNoise </script>

<!-- ===== 3. 引擎层 ===== -->
<script>  // CameraController, PostProcessing, MouseManager, StarfieldBackground </script>

<!-- ===== 4. 组件层 ===== -->
<script>  // SynapseNetwork, Scene1_Hero, Scene2_NeuralGallery, Scene3_SkillConstellation, Scene4_About </script>

<!-- ===== 5. 调度层 ===== -->
<script>  // Stage, Preloader </script>

<!-- ===== 6. UI 层 ===== -->
<script>  // CursorUI </script>
```

---

## 命名规范

### 类名：PascalCase

```js
class SynapseNetwork {}      // ✓
class synapseNetwork {}      // ✗
class synapse_network {}     // ✗
```

### 方法名：camelCase

```js
update(delta) {}             // ✓ 公共方法
render(renderer, camera) {}  // ✓
_initParticles() {}          // ✓ 私有方法以下划线开头
```

### 常量：UPPER_SNAKE_CASE（模块级）

```js
const PARTICLE_COUNT = 1_000_000;
const MAIN_GOLD = new THREE.Color('#D4A843');
const AMBER = new THREE.Color('#B8942E');
```

### 变量/属性：camelCase

```js
this.targetPosition = new THREE.Vector3();   // ✓
this._currentScene = 0;                      // ✓ 私有属性下划线前缀
```

### Three.js 对象命名

所有添加到场景的 Mesh/Points 必须设置 `.name` 属性：

```js
node.name = 'skillNode_aiAgent';    // ✓ 便于调试
pipeline.name = 'synapse_scene2';   // ✓
```

### Uniform 命名：u 前缀

```js
{ uColor: new THREE.Uniform(new THREE.Color('#D4A843')) }   // ✓
{ uTime: new THREE.Uniform(0) }                              // ✓
{ uMouse: new THREE.Uniform(new THREE.Vector2()) }           // ✓
```

---

## 接口规范

### Stage（总调度）

```js
class Stage {
  constructor(containerId) // containerId: canvas 容器 DOM id

  init()                   // 初始化所有模块，返回 Promise<void>
  destroy()                // 销毁所有模块，释放资源

  // 内部持有
  .scene        // THREE.Scene
  .renderer     // THREE.WebGLRenderer
  .camera       // THREE.PerspectiveCamera（由 CameraController 管理）
  .currentScene // 当前激活场景索引 (0-3)
}
```

### Preloader

```js
class Preloader {
  constructor(onComplete)  // onComplete: 加载完成回调

  start()                  // 显示预加载动画
  complete()               // 溶解动画 → 调用 onComplete
}
```

### StarfieldBackground

```js
class StarfieldBackground {
  constructor(scene, count) // scene: THREE.Scene, count: 粒子总数

  update(delta, cameraPos, mouseRipple)
  // delta: 帧间隔秒数
  // cameraPos: THREE.Vector3 当前相机位置（用于 LOD）
  // mouseRipple: { x: float, y: float, strength: float } 鼠标涟漪数据

  setFocusPoint(position, radius)
  // 设置焦点区域，区域内粒子密度增大

  dispose()                 // 释放 GPU 资源
}
```

### SynapseNetwork

```js
class SynapseNetwork {
  constructor(scene)

  addNode(position, options)
  // position: THREE.Vector3
  // options: { radius, isCore, label }
  // 返回 nodeId: string

  connect(nodeIdA, nodeIdB, options)
  // options: { tubeRadius, flowSpeed, pulseInterval }

  setNodeActive(nodeId, active)
  // 激活/停用节点（影响辉光和脉冲频率）

  update(delta, mousePos)
  // mousePos: { x: float, y: float } 屏幕空间归一化坐标 (-1 到 1)

  getNodeWorldPosition(nodeId)  // 返回 THREE.Vector3

  render()
  // 每帧渲染调用 — 更新粒子位置、脉冲信号、管道亮度

  clear()                   // 清空所有节点和连线
  dispose()                 // 释放 GPU 资源
}
```

### Scene1_Hero

```js
class Scene1_Hero {
  constructor(stage)        // stage: Stage 实例，用于访问 scene/renderer/camera

  init()                    // 创建 3D 文字、副标题、箭头
  update(delta, progress)   // progress: 0-1 场景内滚动进度
  onEnter()                 // 摄像机进入本场景时调用
  onLeave()                 // 摄像机离开本场景时调用
  dispose()                 // 释放资源
}
```

### Scene2_NeuralGallery

```js
class Scene2_NeuralGallery {
  constructor(stage)

  init(projects)            // projects: DataModel.projects 数组
  update(delta, progress, mousePos)
  onEnter(currentIndex)     // currentIndex: 当前聚焦的项目索引
  onLeave()
  dispose()
}
```

### Scene3_SkillConstellation

```js
class Scene3_SkillConstellation {
  constructor(stage)

  init(skills)              // skills: { core: [], auxiliary: [] }
  update(delta, progress, mousePos)
  onEnter()
  onLeave()
  dispose()
}
```

### Scene4_About

```js
class Scene4_About {
  constructor(stage)

  init(personalData)        // personalData: DataModel.personal
  update(delta, progress)
  onEnter()
  onLeave()
  dispose()
}
```

### CameraController

```js
class CameraController {
  constructor(camera)       // camera: THREE.PerspectiveCamera

  addKeyframe(progress, position, lookAt, fov)
  // progress: 0-1 整页滚动进度
  // position: THREE.Vector3
  // lookAt: THREE.Vector3
  // fov: number (角度)

  update(scrollProgress)    // 平滑插值相机到当前进度对应位置

  getPosition()             // 返回 THREE.Vector3
  getTarget()               // 返回 THREE.Vector3（lookAt 目标）
}
```

### PostProcessing

```js
class PostProcessing {
  constructor(renderer, scene, camera)

  render(delta, scrollProgress, mousePos)
  // 单帧后处理渲染 — 内部管理所有 Pass

  setRGBStrength(value)     // RGB 色散强度
  setBloomStrength(value)   // 辉光强度

  dispose()
}
```

### MouseManager

```js
class MouseManager {
  constructor(canvas)

  // 属性（每帧自动更新）
  .normalized    // { x: float, y: float } 归一化 (-1 到 1)
  .screen        // { x: float, y: float } 屏幕像素坐标
  .velocity      // { x: float, y: float } 移动速度
  .ripple        // { x: float, y: float, strength: float } 涟漪数据

  update()                  // 每帧调用
  dispose()                 // 解绑事件
}
```

### CursorUI

```js
class CursorUI {
  constructor()             // 复用旧版 DOM 光标逻辑

  update(mouseManager)      // 同步位置和状态
  setState(state)           // 'default' | 'hover' | 'view'
  dispose()
}
```

### DataModel

```js
// 纯数据对象，非 class，模块级常量
const DataModel = {
  personal: {
    nameCn: '翟伟鑫',
    nameEn: 'Vincent',
    email: ['3320780962@QQ.com', 'jrickjrick27@gmail.com'],
    github: 'https://github.com/jjrick62',
    quote: 'AI 不是替代人，而是放大人的创造力。'
  },

  projects: [
    {
      id: 'claude-mobile',
      name: 'Claude Code Mobile Bridge',
      tagline: '将桌面 AI Agent 包装为手机可远程操控的服务',
      tech: ['AI Agent 编排', 'WebSocket', 'Python Flask'],
      github: 'https://github.com/jjrick62/claude-mobile-chat',
      star: true  // 标星项目
    }
    // ...其余 5 个同理
  ],

  skills: {
    core: [
      { name: 'AI Agent 编排', icon: null },
      { name: 'Three.js 3D 可视化', icon: null },
      { name: 'WebSocket 实时通信', icon: null },
      { name: '计算机视觉', icon: null }
    ],
    auxiliary: [
      { name: 'React', icon: null },
      { name: 'Python Flask', icon: null },
      { name: 'PWA', icon: null },
      { name: 'Flutter', icon: null },
      { name: 'Prompt Engineering', icon: null },
      { name: 'Canvas', icon: null },
      { name: 'MediaPipe', icon: null },
      { name: 'Git', icon: null }
    ]
  }
};
```

---

## 事件总线规范

Stage 内部维护一个简单事件总线，场景间通信通过它：

```js
// 发布
Stage.emit('scene:enter', { index: 2, name: 'skills' });

// 订阅
Stage.on('scene:enter', (data) => { /* ... */ });

// 事件名称: 冒号分隔，动词在前
// 'scene:enter'  'scene:leave'  'node:hover'  'node:click'  'resize'
```

---

## 颜色常量

```js
const COLORS = {
  bg:           new THREE.Color('#000000'),
  mainGold:     new THREE.Color('#D4A843'),
  amber:        new THREE.Color('#B8942E'),
  darkGold:     new THREE.Color('#8B6914'),
  warmWhite:    new THREE.Color('#FFF8E7'),
  starCool:     new THREE.Color('#8899AA'),
  starWarm:     new THREE.Color('#CCAA66'),
  pipelineGlow: new THREE.Color('#C9A040'),
  nodeCore:     new THREE.Color('#D4A843'),
  nodeAux:      new THREE.Color('#B8942E'),
};
```

---

## 性能目标

| 指标 | 目标值 |
|------|--------|
| 首屏渲染 | < 2 秒（含 Preloader） |
| 帧率（桌面） | ≥ 60 FPS |
| 帧率（移动端） | ≥ 30 FPS |
| 总粒子数 | 100-150 万（分三层） |
| GPU 内存 | < 512 MB |
| DrawCall 数 | < 50（通过 InstancedMesh 合并） |
