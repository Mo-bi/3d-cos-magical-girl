# 寒酱魔法秀场 · Magical Cos Showcase (v2.4-cos)

> 一个无需构建步骤的单页 3D cos 展示 AR 宇宙，给"寒酱"魔法少女 cos 主题的沉浸式陈列。
> 浏览器打开就是完整应用：十万粒子魔法杖 + 五角星魔法阵、八大行星 + 流星 + 雪场、3D 照片/视频挂件、捏合/挥手/握拳/数字手势、网易云音乐、可持久化的本地素材库。

## 1. 项目一览

- **形态**：单个 `index.html`，无打包工具，无后端，依赖 CDN（Three.js R145 + MediaPipe Hands 0.4 + Google Fonts）。
- **核心栈**：
  - `Three.js` 做 3D 渲染：粒子（Points + 自定义 Shader）、精灵（Sprite）、透视相机、HTML5 Canvas 输出。
  - `MediaPipe Hands` 0.4 做单手 21 关键点检测（仅追踪物理右手）。
  - 原生 `HTML/CSS/JS` 做 UI 面板、屏幕聚焦层、加载层、错误提示。
  - `IndexedDB` 持久化所有上传的照片/视频（DataURL，容量几百 MB，突破 localStorage 5MB 限制）。
- **交互通道**：右手手势 + 鼠标拖拽/滚轮/双击 + 键盘 1–4 / 方向键 / Esc。
- **可定制主题**：默认占位卡片、粒子祝福文字、控制面板标题都已经在文件中以 `乐乐 / Lele` 主题化。

## 2. 启动

由于 `getUserMedia` 需要安全上下文，请通过 `localhost` / `127.0.0.1` 访问，**不要直接双击打开 HTML**。

```bash
cd /Users/neo/Documents/neo-code/3d-interaction-systems
./serve.sh
```

看到 `no-cache server on http://127.0.0.1:4173/` 即可。浏览器打开 <http://127.0.0.1:4173> 并授权摄像头。

> `./serve.sh` 自带 `Cache-Control: no-store` 响应头，避免 Chrome 缓存旧 HTML。也可直接用 `python3 -m http.server 4173 --bind 127.0.0.1` 或其他任何静态服务器。

## 3. 文件结构

```
3d-interaction-systems/
├── index.html          # 全部代码：CSS + DOM + 单文件脚本
├── README.md           # 本文件
├── serve.sh            # 带 no-cache 头的本地服务器
└── CHANGELOG.md        # 版本变更记录
```

整个项目都集中在 `index.html` 中，按以下顺序组织：

| 区段                | 作用                                                                 |
|--------------------|----------------------------------------------------------------------|
| `<style>`          | 设计令牌（CSS 变量）、3D 场景布局、面板、加载层、捏合光标、屏幕聚焦层。 |
| `<header>` / `<aside>` / `<section>` | UI 品牌、控制面板、手势 HUD、加载层 DOM。                       |
| `<script src=...>` | CDN 依赖：Three.js、MediaPipe Hands。                                  |
| `<script>`         | 全局错误捕获 → 入口 IIFE → 状态机 → 渲染循环 → resize 监听。           |

## 4. 脚本内部分区（按出现顺序）

> 所有 `function / const / let` 都位于一个 IIFE 内（`( () => { ... } )()`），不污染全局。

| 行号（≈）  | 名称                         | 作用                                                                 |
|-----------|------------------------------|----------------------------------------------------------------------|
| 690       | IIFE 入口                    | 抓取 `THREE`，所有 DOM 引用，初始化场景/相机/渲染器。               |
| 724–728   | `universe` / `treeGroup`     | 根 Group 用来整体平移/缩放；`treeGroup` 承载蛋糕与所有挂件。       |
| 730–760   | 状态变量                     | `ornaments / focusedOrnament / hoveredOrnament / planets / meteors`。 |
| 792       | `createCenterpiece`          | 十万粒子魔法少女主题（樱花杖+五角星魔法阵+杖尖三公转星）。详见 §6。 |
| 810       | `createEnergyCores`          | 蛋糕内部随机分布的能量光团。                                          |
| 826       | `createCakeTopper`           | 顶部三支蜡烛 + 抖动火焰精灵。                                          |
| 845       | `createSpaceDust` / `createSnow` | 背景星空 + 全局降雪。                                              |
| 870       | `createPlanets`              | 八大行星（按距离、颜色、大小排列，含土星环）。                       |
| 905       | `createMeteors`              | 10 颗巨型流星（核心尺寸 40）。                                        |
| 932       | `createTextParticles`        | 9000 粒子的"粒子文字"系统（采样离屏 canvas）。                       |
| 974       | `showParticleWord`           | 将文字笔画像素映射到粒子目标位置，触发"粒子凝聚成文字"。            |
| 1007      | `addOrnament`                | 在蛋糕外缘按真实宽高比挂照片/视频，保持原始比例 + 金色边框。         |
| 1063      | `calculateFocusScale`        | 基于相机 fov + 视口宽高，把照片缩放至屏幕 78% 安全区。                |
| 1078      | `showMediaFocusOverlay`      | 在屏幕层创建 `<img>` 或 `<video>` 副本，专门用于选中态的"正面展示"。 |
| 1102      | `hideMediaFocusOverlay`      | 关闭聚焦层，暂停视频。                                                  |
| 1108      | `focusOrnament`              | 选中装饰品的核心：进入 `orient → expand` 两阶段动画（详见 §7）。   |
| 1143      | `releaseOrnament`            | 释放：把装饰品挂回蛋糕，回归原始位置和姿态。                          |
| 1147      | `STORAGE_KEY` + 持久化       | `localStorage` 保存 / 恢复所有素材。详见 §8。                         |
| 1247      | `handleFiles`                | 把 `File` 转成 DataURL 持久化，再走 `handleSingleFile`。            |
| 1305      | `initHands`                  | 首次启动 MediaPipe Hands（WASM），单例避免 Buffer 溢出。            |
| 1330      | `startCamera` / `stopCamera` | 摄像头权限、设备枚举、Hands 循环。                                    |
| 1368      | `classifyGesture`            | 用 21 关键点判定 `pinch / palm / fist / fingerCount`。              |
| 1390      | `onHandResults`              | 命中、聚焦、挥手旋转、握拳平移、张掌缩放、1–4 召唤文字。             |
| 1700+     | `animate`                    | 主循环：粒子时间轴、行星/雪/流星/装饰品更新、聚焦阶段推进、渲染。 |

> 上面行号是基于当前 `index.html` 的近似位置；文件继续演进时，请用 `rg -n "^      function |^      const " index.html` 自行核对。

## 5. UI 层级（z-index）

> 数字越大越靠上。重要的层都做了"硬压"，避免被 Three.js 画布（默认 1）挡住。

| 元素                          | z-index | 备注 |
|------------------------------|---------|------|
| `#camera-feed`               |   0     | 摄像头画面背景。 |
| `#stage`                     |   1     | Three.js 画布。 |
| `.noise` / `body::before`    |   2/3   | 颗粒 + 暗角。 |
| `.crosshair`                 |   4     | 屏幕中央准星。 |
| `.ui`                        |   5     | 品牌、状态、手势 HUD。 |
| `.panel-toggle`              |   6     | 面板内 `×` 按钮。 |
| `.media-focus-overlay`       |  40     | **选中照片/视频的屏幕层**。已压到所有 3D 之上。 |
| `.pinch-cursor`              |   8     | 蓝色捏合光标。 |
| `.loading`                   |  20     | 启动加载层（900ms 自动隐藏）。 |

## 6. 3D 场景细节

### 6.1 蛋糕粒子（`createCake`）

- 数量：`100000` 颗。
- 几何：自下而上三层圆台（半径 `37 / 29 / 21`）。
- 颜色：粉 `#f4a9bd`、浅粉 `#ffd5df`、奶油 `#fff0d7`、焦糖 `#d89070`、流金 `#e8c477`、薄荷 `#9be3c8`；分层顶部有"奶油边缘"。
- Shader：
  - 顶点：基本位置 + 相机距离透视 + 时间脉冲。
  - 片元：径向 alpha + Additive Blending。
  - Uniforms：`uTime`、`uBurst`（靠近时整体放大 0.4× 并提亮，但不会过曝）、`uPixelRatio`。

### 6.2 装饰品朝向

`addOrnament` 之后，每个挂件在动画循环里都会执行：

```js
tmpBillboardTarget.copy(camera.position);
tmpBillboardMatrix.lookAt(tmpBillboardTarget, ornament.position, camera.up);
tmpBillboardQuaternion.setFromRotationMatrix(tmpBillboardMatrix);
ornament.quaternion.slerp(tmpBillboardQuaternion, .35);
```

- `Matrix4.lookAt(eye, target)` 会让对象的 `+Z` 指向 `eye - target`。
- 因此 `lookAt(camera.position, ornament.position, ...)` 始终让**正面（+Z）面对相机**。
- `slerp(.35)` 提升跟手性，旋转蛋糕时照片几乎实时跟踪。
- 聚焦中的那张被硬锁为 `camera.quaternion`，姿态永不偏移。

### 6.3 屏幕聚焦层

为什么不复用 3D 卡片做放大？因为：

- 父级 `treeGroup` 在旋转，姿态与位置必须脱离。
- 透视 / 抖动 / 背面会干扰体验。
- 屏幕空间层天然无角度误差。

实现：

1. `focusOrnament` 调用 `scene.attach(ornament)` 把挂件从旋转父级脱离，**并隐藏**。
2. 立刻创建一个 `<img>` / `<video>` DOM 副本到 `#media-focus-overlay`。
3. 320ms 后切换 `.expanded`，从原始位置 lerp 到屏幕中心并放大到 78% 安全区。
4. 整个过程由 `focusPhase = 'orient' | 'expand' | 'locked'` 控制，避免循环每帧重算目标造成抖动。
5. `.expanded::before` 加全屏径向黑色蒙层，让粒子在视觉上退到背景。

## 7. 手势识别

`classifyGesture(landmarks)`：

- `pinchDistance = distance(thumb_tip, index_tip)`
- `palmWidth = distance(index_mcp, pinky_mcp)`（用于归一化）
- `pinch = pinchDistance < palmWidth * 0.34`
- `fist` = 4 根手指全部未伸直且 `!pinch`
- `palm` = 4 根手指伸直且拇指张开
- `count` = 伸直的手指数量 → 召唤粒子文字 1–4

`onHandResults` 流程：

1. 只接受 `handedness === 'Right'` 的手（屏蔽左手）。
2. 捏合 + 没有聚焦中目标 → 用拇指-食指中点做 NDC，命中最近装饰品并聚焦。
3. 已有聚焦时按捏合中点移动可换目标。
4. 松开捏合 → 释放。
5. 手掌横移 → `targetRotationY += Δx * 5.8`，左右旋转整个蛋糕。
6. 拳头上下 → 平移整个 universe。
7. 张掌 → 控制相机 `z` 推拉。
8. 1–4 指 → 召唤 `Happy / Birthday / Lele / Happy Birthday Lele`。

## 8. 素材持久化（localStorage）

- key：`lele.birthday.ornaments.v1`
- 形态：JSON 数组，每项 `{ name, type, dataUrl }`。
- 流程：
  1. 上传时 `FileReader.readAsDataURL` → `dataUrl` 写入 localStorage，并挂到蛋糕上。
  2. 启动时 `readSavedOrnaments()` → 如果有就 `restoreOrnaments` 走 `TextureLoader` / `VideoTexture` 重建。
  3. 没有时回到默认演示卡片。
  4. 删除单个 / "清空所有素材" 都会同步更新 localStorage。
- 注意：
  - localStorage 通常 5–10 MB 限额，**视频很多或很大时会被截断**，会弹出 `本地存储空间不足` 提示。
  - 修复方式：把 `persistOrnaments` 改为 IndexedDB（已有 `handleSingleFile` 抽象层，迁移成本小）。

## 9. 交互速查

| 操作                | 手势                            | 鼠标 / 键盘                  |
|--------------------|--------------------------------|-------------------------------|
| 推拉镜头           | 张掌前后移动                    | 滚轮                          |
| 选中照片/视频      | 拇指-食指捏合                   | 双击                          |
| 释放 / 取消聚焦    | 松开捏合                        | 按 `Esc` / 双击空白           |
| 旋转蛋糕           | 右手左右挥动                    | 拖拽                          |
| 平移星系           | 握拳上下移动                    | 方向键上下                    |
| 召唤粒子文字       | 伸出 1–4 指（保持 0.27s）       | 键盘 `1`–`4`                 |
| 切换面板           | 无                              | 面板内右上角 `×`              |

## 10. 如何继续修改

下面给出常见修改的入口，方便你或下一个 AI 接管：

- **换主题（颜色 / 文字）**：
  - 调色板 → `createCake` 里的 `palette` 数组。
  - 粒子文字词表 → `classifyGesture` 末尾的 `['Happy', 'Birthday', 'Lele', 'Happy Birthday Lele']`；键盘分支同步。
  - 品牌名 → `<header class="brand">` 内的 `LELE` / `BIRTHDAY UNIVERSE / V2.3`。
- **增加一种装饰品类型（GIF / Lottie）**：扩展 `addOrnament` + `showMediaFocusOverlay` + `handleSingleFile`。
- **改成 IndexedDB 持久化**：替换 `persistOrnaments` / `readSavedOrnaments` 内部实现，对外接口不变。
- **新增一个手势**：在 `classifyGesture` 推导新字段，并在 `onHandResults` 里添加分支。
- **调整默认相机距离**：`targetCameraZ` 初始值，以及张掌缩放的 `THREE.MathUtils.clamp(...)` 上下限。
- **蛋糕变回圣诞树**：恢复 `createTree` 的圆锥采样（已移除），保留现在的 UI/交互。

## 11. 已知坑

1. **MediaPipe Hands WASM 体积较大**（~6 MB），首次加载需要几秒，会看到加载层。若 CDN 失败，全局 `window.error` 会把堆栈写到加载层。
2. **手势误判**：食指/拇指对环境光较敏感，暗光下需要稳定的手部光线。
3. **iOS Safari 视频自动播放**：默认会失败；聚焦层有 fallback：先 `muted=true` 自动播放，再让用户点控件解除静音。
4. **localStorage 限额**：见 §8，视频量大需要切到 IndexedDB。
5. **缓存问题**：每次大改后请在浏览器 `Cmd/Ctrl+Shift+R` 强制刷新，或者加 `?v=2.3` 这样的查询参数。

## 12. 快速冒烟自检

```bash
node -e "const html=require('fs').readFileSync('index.html','utf8');const m=[...html.matchAll(/<script(?:\s[^>]*)?>([\s\S]*?)<\/script>/g)].map(x=>x[1]).filter(Boolean);for(const c of m)new Function(c);console.log('JS ok');"
curl -fsS http://127.0.0.1:4173/ | head -c 200
```

预期：第一条打印 `JS ok`，第二条返回 `<!doctype html>` 开头。

## 13. 致下一个 AI

- 务必保持"屏幕层聚焦 + 3D 装饰品挂回"的双层架构，不要为了简化把放大再次走 Three.js。
- 任何 `new THREE.*()` 都不能放进 `animate` 内的循环（已踩过坑：会产生 GC 卡顿和块作用域报错）。所有 helper 提到 IIFE 顶部。
- 装饰品朝向的 `lookAt(eye, target, up)` 参数方向是**反直觉的**，eye 和 target 写反会看到背面，请保持 `lookAt(camera.position, ornament.position, camera.up)`。
- `focusedOrnament` 的 `focusPhase` 状态机是为了防止每帧重算目标导致照片抖动，新增状态时务必保持"锁定后冻结目标"不变。
- 改任何持久化字段名时请同步 `STORAGE_KEY` 版本号，避免老用户的素材被读到新结构里崩。

Happy birthday, 乐乐 🎂

## 14. dev 分支迭代记录

> 以下变更在 `dev` 分支（最新 commit `feat/cos-magical-girl`）。`main` 分支保持 v2.3 release。

### 缩放

| commit | 改动 |
|--------|------|
| `5be94c1` | 手掌缩放范围 30–380 → **5–290**，可进入蛋糕内部 |
| `ea753e0` | 默认相机距离 145 → **95**，蛋糕占视野 65-70% |
| `fe5317f` | 最远距离 540 → **290**，蛋糕占视野 1/3 |

### 旋转（惯性）

| commit | 改动 |
|--------|------|
| `78d98d6` | 鼠标上下拖动俯仰 ±0.7 → **±π**，可完全翻转看到顶部/底部 |
| `8d85de4` | 引入 `angularVelocity` 角速度变量，挥手产生惯性 |
| `9919b55` | 反向挥手后衰减回默认方向 |
| `142dd9c` | 加载默认向右旋转 + 鼠标水平拖动走惯性逻辑（与手势一致） |

### 蛋糕 + 手势

| commit | 改动 |
|--------|------|
| `975e4e2` | 三层蛋糕分层调色（底层薄荷+柠檬 / 中层粉红 / 顶层紫+玫红，年轻有活力）；pinch 加 hysteresis（进入 0.34 / 退出 0.48）避免反复确认抖动；捏合时 cake 停止旋转方便选照片；松开后按 `lastWaveDir` 恢复旋转方向 |
| `8eaa365` | 上传立即持久化（不等 loader/loadeddata 异步）+ 删除同步 storage |
| `3899d8a` | 持久化失败时显示 ⚠ 警告 toast |


### 主体（cos 化）

| commit | 改动 |
|--------|------|
| `feat/cos-magical-girl` | 替换生日蛋糕为魔法少女主题：`createCake` → `createCenterpiece`（10w 粒子分布在杖身/杖身光晕/五角星魔法阵/杖尖樱花/全局飘散五个区域），调色板改为樱花粉+薄藤紫+月白+暖金；`cakeRadiusAtY` → `centerpieceRadiusAtY`（装饰品挂载圆周半径）；能量球改为沿杖身向上飘；顶部三蜡烛替换为三颗公转星+一颗杖尖核心；UI 标题/词表/demo 标签全面换皮。STORAGE_KEY/IDB_NAME 保留以保护已持久化素材。 |

### 持久化（IndexedDB）

| commit | 改动 |
|--------|------|
| `9534b2f` | **从 localStorage 迁移到 IndexedDB**，突破 5MB 限制（几百 MB 容量）|
| 工具函数 | `openIdb` / `idbSave` / `idbGetAll` / `idbDelete` / `idbClear` |
| 数据 schema | DB: `lele-birthday` / Store: `ornaments` / KeyPath: `dataUrl` |
| 迁移注意 | 旧 localStorage 数据不会自动迁移，需要手动清空后重新上传 |

## 15. Git 工作流

```bash
# 开发新功能（在 dev 分支）
git checkout dev
git add .
git commit -m "feat/fix: 描述"
git push

# 查看所有版本
git log --oneline

# 回退到某个版本
git reset --hard <commit-hash>
git push --force
```


## 15. 本地 vendor（v2.4-cos 起）

为解决国内访问 `cdn.jsdelivr.net` / `unpkg.com` 慢导致卡在 loading 页的问题，所有第三方依赖改为 `vendor/` 目录本地引用（6.3MB）：

| 文件 | 大小 | 用途 |
|------|------|------|
| `vendor/three.min.js` | 588KB | Three.js R145 |
| `vendor/hands.js` | 44KB | MediaPipe Hands 0.4 入口 |
| `vendor/hands.binarypb` | 550B | MediaPipe 计算图 |
| `vendor/hands_solution_packed_assets_loader.js` | 8KB | data 文件加载器 |
| `vendor/hands_solution_packed_assets.data` | 4.1MB | 内含 palm_detection tflite 模型 |
| `vendor/hands_solution_wasm_bin.js` | 270KB | WASM（非 SIMD） |
| `vendor/hands_solution_simd_wasm_bin.js` | 270KB | WASM（SIMD） |

`Hands` 构造时 `locateFile: (file) => ./vendor/${file}` 指向本地。
