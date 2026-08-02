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

### `<pending>` fix(persist): 上传立即持久化，删除同步 storage
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
