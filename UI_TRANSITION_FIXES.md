# PENGWH UI 修复说明

本次对首页、移动端导航、桌面端导航及静态生成的文章/分类/标签/关于页面统一更新。

- 顶部导航在页面顶部保持透明、无明显边界；随着滚动逐步变成胶囊。
- 胶囊增加更强的背景模糊、饱和度与对比度，并添加多层高光、边缘折射和内外阴影。
- 深色主题在 CSS 首次绘制前同步读取 `halcyon-theme`，避免页面先显示浅色再变暗。
- 同站页面切换加入 View Transition 动画；不支持跨页面 View Transition 的浏览器使用渐隐/渐入回退效果。
- 尊重系统的“减少动态效果”设置。

验证：`npm run build` 已运行通过，其中包含源码检查、导航变形测试、滚动显现测试、Twikoo 适配器测试和最终静态页面构建。


## 第二轮无缝融合复核（2026-10-24）

- 修正顶部零滚动状态的玻璃效果：顶部由一层向下渐隐的 30px 全宽磨砂层提供背景模糊，胶囊自身的模糊则从 0 连续增强到 38px，避免全宽导航框在底缘产生硬切线；玻璃底色在顶部由全宽渐隐层提供，滚动后的胶囊保持约 72% 的混合透明度，让背景模糊与折射更容易被看见，不会像纯白/纯黑实心块。
- 新增延伸到导航栏下缘之外的全宽磨砂渐隐层，并用渐变遮罩逐渐淡出，减少背景模糊在导航底部突然截止形成的硬边。
- 顶部反射高光移至会渐隐的全宽背景层；胶囊本体的反射与描边只随胶囊进度增强，避免全宽矩形下缘留下 1px 硬线。
- 为这些条件增加导航回归断言，验证手机与桌面滚动进度为 0 和 1 时的核心参数。
- 复核深色主题引导脚本位于首屏样式之前，独立文章/分类/标签/关于页面均使用统一的主题初始化与页面过渡脚本。


## Follow-up audit: safe-area continuity and stronger refraction

- Extended the top frosted veil through the mobile safe-area inset so the status-bar area and navigation visually share one continuous surface on notched phones.
- Increased capsule specular streaks, cool edge glints, and inner top-edge highlights while keeping them progress-driven; they still emerge smoothly rather than appearing at the very top.
- Added regression assertions for the full-bleed safe-area layer, refraction cues, and generated-page theme-bootstrap order.
