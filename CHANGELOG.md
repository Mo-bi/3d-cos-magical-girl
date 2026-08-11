# Changelog · 版本变更记录

所有重要变更按时间倒序记录。每次 commit 完成后追加一段。

---

## [dev] — 2026-08-02

### `78d98d6` feat(rotation): 鼠标上下拖动俯仰范围 ±0.7 → ±π
- 鼠标按住上下拖动现在可以完全翻转蛋糕看到顶部/底部
- 俯仰范围从 ±0.7 弧度（约 ±40°）扩大到 ±π 弧度（±180°）
- 拖动灵敏度从 0.005 rad/px 提升到 0.008 rad/px

### `ea753e0` feat(zoom): 默认相机距离 145 → 95
- 蛋糕默认占视野约 65-70%，更沉浸
- camera.position.set(0, 5, 95)、targetCameraZ = 95

### `5be94c1` feat(zoom): 手掌缩放范围 30-380 → 5-540
- 引入 palmRatio 归一化映射，palmMin=0.05, palmMax=0.45
- 相机最近距离 5 单位，可进入蛋糕内部
- 相机最远距离 540 单位，覆盖整个 AR 场景
- 鼠标滚轮范围同步 5-540，灵敏度 0.07 → 0.15
- cake gl_PointSize 加 min(..., 32.0) 上限，避免近距离粒子过大遮挡

### `<pending>` feat(persist): 迁移到 IndexedDB（突破 5MB 限制）
- 新增 openIdb / idbSave / idbGetAll / idbDelete / idbClear 工具函数
- 创建 IndexedDB 数据库 `lele-birthday`，object store `ornaments`（keyPath: dataUrl）
- readSavedOrnaments / saveOrnamentImmediately / persistOrnaments 改为 async + IDB
- 移除 localStorage（5MB 限制），改用 IDB（几百 MB 容量）
- removeOrnament 调用 idbDelete，clearAll 调用 idbClear
- 初始化改为 async IIFE 等待 IDB 读取完成

### `3899d8a` fix(persist): 上传立即持久化 + 配额警告
- saveOrnamentImmediately 返回 boolean 标识保存成功
- 配额满时显示明确警告 toast（不再被 addOrnament 成功 toast 覆盖）
- addOrnament 后只在保存成功时显示「已挂载 ✓ 自动保存」
- 新增 saveOrnamentImmediately：fileToDataUrl 完成后立即写入 localStorage，不等 loader/loadeddata
- persistOrnaments 改为累加式：保留 storage 中已有 + 当前 ornaments 合并去重
- removeOrnament 调用 persistOrnaments 同步删除 storage 记录
- 修复视频上传后刷新丢失的问题（loadeddata 异步未触发前刷新）
- 修复手动删除装饰品后刷新又出现的问题（之前 removeOrnament 没同步 storage）

### `975e4e2` feat(cake+gesture): 三层分层调色 + 捏合优化
- 蛋糕三层不同色调，年轻有活力：
  - 底层（半径 37）：薄荷绿 + 柠檬黄 + 奶白
  - 中层（半径 29）：粉红 + 奶白 + 玫瑰
  - 顶层（半径 21）：薰衣草紫 + 玫红 + 奶白
- 每层独立的 palette，避免六色随机到三层导致颜色混杂
- palette 元素改为 new THREE.Color(...) 避免 color.copy(number) 失败
- pinch 加 hysteresis（进入 0.34 / 退出 0.48），避免反复抖动
- 捏合时 angularVelocity = 0，cake 停止旋转方便选照片
- 捏合松开后 angularVelocity = lastWaveDir * BASE_ANGULAR，按最后挥手方向恢复
- 焦点只在 pinch 上升沿（lastPinch false → true）触发 focusOrnament，避免每帧重复确认
- 挥手/鼠标拖动都记录 lastWaveDir，确保松开捏合后能恢复旋转方向

### `8d85de4` feat(rotation): 左右挥手旋转 + 惯性 + 方向保持
- 新增 angularVelocity 角速度状态变量（初始 0.18 rad/s）
- 挥手时 angularVelocity += horizontalMovement * 38（clamp ±2.6）
- 每帧 angularVelocity 衰减到 signedBase（保持方向的基础速度 ±0.18）→ 改为衰减到 +BASE_ANGULAR
- 旋转方向：挥手方向决定 → 衰减后回到默认正向
- 挥手后惯性自动平滑过渡到默认速度

---

## [v2.3] — 2026-08-02 (初始 release on main)

### `2ba078c` feat: 乐乐生日 AR 宇宙 v2.3 — 十万粒子三层蛋糕 + 手势交互 + 媒体聚焦
- 十万粒子三层生日蛋糕（粉金奶油色 + 顶部三蜡烛）
- 八大行星、流星、雪场、宇宙星空（Camera Feed AR 背景）
- MediaPipe Hands 单手追踪：捏合聚焦 / 拳头平移 / 张掌缩放 / 1-4 指召唤文字
- 屏幕层聚焦（z-index 40）保持照片视频正面朝向
- localStorage 持久化装饰品 + 网易云音乐 ID
- 装饰品列表 + 清空所有素材 + 网易云/本地音乐加载与移除
- 78% 屏幕安全区放大 + 锁定后冻结目标
- 控制面板折叠设计 + 1 大于号折叠按钮

---

## [v2.4-cos] — 2026-08-11 (dev 分支，魔法少女 cos 化)

### feat(cos): 主体从生日蛋糕替换为魔法少女主题（樱花杖 + 五角星魔法阵 + 杖尖三公转星）
- 替换 `createCake` → `createCenterpiece`，10w 粒子分布在五个区域：
  - 25% 杖身表面（y=-30~30，圆柱半径 1.6，月白+薄藤紫+深紫）
  - 15% 杖身光晕（围绕杖身扩散晕，薄藤紫+樱花粉+深紫）
  - 25% 五角星魔法阵（y=-30 水平面，五角星轮廓，樱花粉+暖金+深粉）
  - 15% 杖尖樱花绽放（y=31~37 球状，樱花粉+深粉+月白）
  - 20% 全局飘散（围绕主体球壳，自由调色）
- 调色板重做：樱花粉 0xffb7d5 / 薄藤紫 0xc8a2ff / 月白 0xfff0fa / 暖金 0xffe699 / 深粉 0xff9ed2 / 深紫 0xb78dff
- `cakeRadiusAtY` → `centerpieceRadiusAtY`，按 y 段返回装饰品挂载圆周半径（32/38/40/36/26），照片/视频围绕魔法杖悬浮
- 能量球 (`createEnergyCores`) 改为沿杖身向上飘（32 个 sprite，driftSpeed/phase/random 飘动），颜色粉/紫/白
- 顶部装饰 (`createCakeTopper`) 替换三蜡烛：
  - 三颗星（金/粉/紫）绕杖尖 y=32 半径 9 公转
  - 一颗月白核心在杖尖 y=31 闪烁
- UI 文案："乐乐生日" → "寒酱的魔法秀场 · Magical Cos Showcase"
- 1-4 指召唤词表：`['Happy', 'Birthday', 'Lele', 'Happy Birthday Lele']` → `['Wish', 'Magic', 'Bloom', 'Magical Cos']`
- demo 装饰品标签：`LELE'S MEMORY` / `乐乐 · 2026` / `MAKE A WISH` → `COS PLAY` / `魔法少女` / `STAR WISH`
- 全局 `cakePoints` 变量名保留（指向 `createCenterpiece` 产物，animate 中两处引用无需改）
- STORAGE_KEY / IDB_NAME / MUSIC_STORAGE_KEY 全部保留，保护用户已持久化的素材数据

---

## [v2.4-cos-fork] — 2026-08-11 (项目拆分)

### 仓库物理隔离
- 创建独立 GitHub repo: `Mo-bi/3d-cos-magical-girl` (cos-origin)
- cos 化 3 个 commit (49beaac / 28ef241 / b92cd5e) 作为新 repo 初始历史
- 原 repo `Mo-bi/3d-interaction-systems` (origin) 保留乐乐生日 v2.3 main + dev 迭代
- 删除原 repo 上的 `feat/cos-magical-girl` 分支（避免混淆）
- worktree `feat/cos-magical-girl` 保留检出，开发者下次 push 用 `git push cos-origin feat/cos-magical-girl`
