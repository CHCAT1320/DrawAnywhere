# 路线图

## 已完成
- [x] 矩形 / 椭圆笔类型
- [x] Hover 笔触预览（屏幕空间）
- [x] 橡皮颜色 / 透明度不持久化（默认 LightGray）
- [x] 双指 / 三指手势（pan/zoom/tap）
- [x] 手指绘制开关
- [x] 缩放锁定 + HUD
- [x] `TapGestureDetector` 独立
- [x] `drawing/` 包：`StrokeTool` 接口 + `FreehandTool` / `ShapeTool`
- [x] 像素橡皮（`CanvasSnapshot` undo/redo）
- [x] `PenType` 加入 `labelResId` / `isEraser` / `icon`
- [x] 取色器（HSV 色轮 / 预设色板 / 最近颜色 / Hex 输入）
- [x] 橡皮擦禁用调色板 + Alpha 滑块（disabled 而非隐藏）
- [x] `Spacing` 设计常量统一 dp 魔数
- [x] 激光笔（TTL 3s + 淡出 + glow 效果）
- [x] 能力对象 `Renderer` / `HitTester`（`Capabilities.kt`）
- [x] `DrawAction.withoutEphemeral()` — 统一过滤临时笔画
- [x] 109 单元测试

## 待做

### 笔 / 工具
- [ ] 箭头形状
- [ ] 文本框
- [ ] 图像插入
- [ ] 常见几何形状（三角形、菱形、星形、多边形）
- [ ] 形状填充（矩形 / 椭圆内部填色）
- [ ] 笔 profile（颜色 / 宽度 / alpha 预设，可与调色板整合）
- [ ] 套索工具

### 工具栏
- [ ] 自定义工具栏按钮（菜单项可单独拖出，如橡皮）
- [ ] 工具栏 profile（可保存/切换不同按钮布局）
- [ ] 工具栏可调整大小
- [ ] 闲置时工具栏透明度可配置（0%–100% + timeout）
- [ ] 退出按钮
- [x] 拖拽工具栏到屏幕底部关闭
- [ ] 次工具条（垂直方向）

### 保存 / 恢复
- [ ] 保存画布（含背景 / 不含背景）—— 缩放问题待讨论
- [ ] 分享按钮
- [ ] 保存 / 恢复绘制状态（可编辑，即恢复后继续画）

### 交互
- [x] 手指绘制时也显示 hover 圆圈（手指离开后 300ms 延迟，再 200ms 淡出；笔直接淡出，200ms）
- [ ] 缩略图 / minimap（无限画布的全局导航小窗）
- [ ] remap 手势
- [ ] 笔按键环形菜单（中央橡皮 + 外圈功能扇区）
- [ ] 有些 stylus 的按键无法触发橡皮
- [ ] Palm rejection

### 平台 / 窗口
- [ ] DeX 模式下外接显示器绘制
- [x] 软件运行时再次打开软件则退出
- [ ] 覆盖层窗口大小适配（取色器、设置等 popup 受 WRAP_CONTENT 限制）

### 像素橡皮扩展
- [ ] 支持 Rectangle / Ellipse（边缘轮廓采样）
- 已知限制：只擦 Pen stroke（Rectangle / Ellipse / Laser 不受影响）；点密度不足时残留碎片

### 待修（低优先级）
- [ ] `ToolbarLifecycleOwner` 补齐 ON_START / ON_RESUME 生命周期回调（当前无 Observer，无实际影响）

---

## 自动透传触摸 — 探索记录

需求：关闭手指绘制时，若笔未靠近屏幕，手指触摸透传给下方应用，笔事件仍由画布接收。

### 已尝试方案

1. **Classic（FLAG_NOT_TOUCHABLE）** — 已实现（三指双击切换）。全有或全无：透传时笔和手指一起透传，无法绘画。简单可靠。

2. **定时探测（probing）** — 尝试后放弃。每 300ms 短暂移除 FLAG_NOT_TOUCHABLE 嗅探笔靠近，其余时间保持透传。
   - Android 12+ 强制将 FLAG_NOT_TOUCHABLE 的 overlay 窗口变半透明，导致频繁闪烁 —— **致命 UX 缺陷**。
   - 探测窗口内可能吃掉用户手指点击。

3. **Shizuku + injectInputEvent(targetUid)** — 尝试后放弃。
   - `targetUid` 不绕过 Z-order，注入事件仍先到 overlay → UID 不匹配被丢弃。
   - 临时 FLAG_NOT_TOUCHABLE + 注入产生 CANCEL 冲突，前台收不到完整手势。
   - `app_process` 冷启动延迟不可控。

4. **AccessibilityService dispatchGesture** — 未实现。
   - 只能做预录制 gesture，不支持实时连续触控。
   - 多点手势限制（不能加减手指）。
   - tap 类短手势可用，scroll/pinch 不可用。

### 结论

Android 输入管线按 Z-order 派发事件，不支持按 tool type（笔/手指）分流。所有"选择性透传"方案最终都受限于窗口模型本身。Classic 方案（三指双击手动切换 FLAG_NOT_TOUCHABLE）是唯一零缺陷的实现。
