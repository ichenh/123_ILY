# ✨ 手势控制 3D 粒子告白墙 Gemini 提示词

## 1. 核心功能与初始化

**目标：** 快速搭建项目的基础结构和配置，确保 MediaPipe 和 Three.js 能够协同工作。

* **提示词 A (初始化)**
    > "我正在开发一个使用 **Three.js (ESM)** 和 **MediaPipe Hand Landmarker (v0.10.0+)** 的网页应用。请求提供一个包含以下功能的 `script.js` 初始化模板：**加载手势模型**，**通过 `getUserMedia` 访问摄像头**，并设置 **Three.js 场景** (包含 `PerspectiveCamera` 和 `Points` 粒子系统)。确保 MediaPipe 在 `VIDEO` 模式下运行，并在 `predictWebcam` 循环中调用 `detectForVideo`。"
* **提示词 B (CDN 故障转移)**
    > "在 MediaPipe 初始化中，为了增强中国大陆用户的访问可用性，需要实现 **CDN 故障转移机制**。请修改 `initMediaPipe` 函数，使其首先尝试通过 CDN 加载 WASM 资源，如果失败（例如网络错误或超时），则**自动切换到加载本地 `./models/` 文件夹**中的 WASM 文件和模型文件。"

## 2. 交互逻辑与手势识别

**目标：** 解决手势识别的精度、鲁棒性以及核心的交互映射。

* **提示词 C (手势映射)**
    > "请提供一个鲁棒的 JavaScript 函数 `countFingers(landmarks)`，该函数接收 MediaPipe Hand Landmarker 输出的 21 个关键点，并准确返回 **0 到 5 之间的手指数量**。特别要求能区分 **拇指** 的张开状态（与手掌/小指基部的相对距离），以及其他四指的伸直状态（指尖与指关节的相对 Y 坐标）。"
* **提示词 D (鲁棒性与假阳性)**
    > "我使用 MediaPipe Hand Landmarker 时遇到了**假阳性问题**：当画面中只有上半身时，模型偶尔会误判为 `count=1`。请修改 `predictWebcam` 预测循环，要求在处理手势前，必须检查 `results.handedness[0][0].score` **信心分数**。只有当分数**高于 25% (0.25)** 时，才触发 `handleGesture`，否则视为手势丢失 (`count=-1`)。"

## 3. 动画与效果优化

**目标：** 优化性能和视觉效果，解决动画的延迟和流畅性问题。

* **提示词 E (粒子形态切换)**
    > "我需要将 `1`、`2`、`3`、`I ❤️ U` 等文本/图形形状映射到 Three.js 的粒子系统。请描述如何使用 **`MeshSurfaceSampler`** (TextGeometry) 或 **数学函数** (Heart Shape) 来生成目标粒子位置 (`targetPositions`)，并使用 **线性插值 (Lerp)** 结合微小随机噪声，在 `animate` 循环中平滑地将现有粒子 (`positions`) 移动到这些目标位置，以创建流畅的形态切换效果。"
* **提示词 F (I❤️U 延迟消除)**
    > "为了防止手势抖动引起告白模式 (`count=5`) 闪烁，我加入了 `300ms` 的 `setTimeout` 防抖。但用户反馈启动太慢。请优化 `handleGesture` 函数中处理 `count=5` 的逻辑：将 **`CONFESSION_THRESHOLD_MS` 缩短到 `100ms`**，并在 `setTimeout` **成功触发** `triggerConfession` 后，立即将 `lastGesture` 更新为 `5`，以确保在告白模式激活后，后续帧不会重复计时或干扰动画。"

## 4. 样式与布局调整

**目标：** 解决不同设备上的布局适应性问题。

* **提示词 G (响应式布局)**
    > "我的粒子文字需要适应横屏 (PC) 和竖屏 (Mobile) 布局。请提供一个 JavaScript 逻辑，根据 `window.innerWidth / window.innerHeight` 的 **宽高比** 判断布局模式 (`HORIZONTAL` 或 `VERTICAL`)。然后，定义一个配置对象 `LAYOUT_CONFIG`，为 `I`、`❤️`、`U` 三个部分分配不同的 **X/Y 偏移量**，以确保它们在两种模式下都能居中且间距合理。"
* **提示词 H (爱心跳动)**
    > "在 `I ❤️ U` 告白模式下，需要让心形粒子产生**跳动效果**。请描述在 Three.js `animate` 循环中，如何使用 **`Math.sin(time)`** 生成一个脉冲缩放因子，然后将其平滑地应用于心形粒子的 **原始目标位置 (`originalHeartPositions`)** 上，使其在 `CONFIG.HEART_PULSE_MIN` 和 `CONFIG.HEART_PULSE_MAX` 之间周期性缩放，营造出跳动的动画效果。"