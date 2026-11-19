# 设计系统 - 职业健康安全管理数据中台

基于 UI/UX Pro Max 生成，风格定位：Data-Dense Dashboard（数据密集型仪表盘）+ Enterprise Gateway。

## 设计原则

- 信息密度优先：KPI 卡片、数据表、图表网格布局，最大化数据可见性
- 语义化配色：蓝色主色体系 + 琥珀色强调，状态色统一
- 一致性：三端共用同一套设计语言与组件，仅导航与功能入口不同
- 无障碍：WCAG AA，对比度 ≥ 4.5:1，不使用 emoji 作图标，使用 SVG

## 配色方案

| 用途 | 色值 | 说明 |
|-----|------|------|
| Primary | `#1E40AF` | 主色，导航/主按钮/选中态 |
| Secondary | `#3B82F6` | 次级蓝，链接/图表辅助 |
| CTA | `#F59E0B` | 强调色，关键操作/预警 |
| Background | `#F8FAFC` | 页面背景 |
| Text | `#1E3A8A` | 正文深色 |
| Success | `#16A34A` | 完成/通过 |
| Warning | `#F59E0B` | 预警 |
| Danger | `#DC2626` | 拦截/超标/异常 |
| Border | `#E2E8F0` | 边框/分隔线 |

## 字体

- 正文：Fira Sans（Google Fonts）
- 数字/编码：Fira Code（等宽，用于统计数字、企业编码）
- 中文回退：系统默认无衬线字体

## 组件规范

- 圆角：卡片 `rounded-lg`，标签 `rounded-full`
- 阴影：卡片 `shadow-sm`，浮层 `shadow-lg`
- 间距：网格 gap-4，卡片内边距 p-5
- 表格：行 hover 高亮 `bg-blue-50`
- 状态标签：色块 + 文字双通道（不仅靠颜色）
- 过渡：hover/focus 150-300ms

## 交互效果

- Hover tooltips（图表、指标卡）
- 图表点击缩放
- 表格行 hover 高亮
- 筛选平滑过渡
- 数据加载 spinner

## 应避免

- 过度装饰
- 缺少筛选能力
- 使用 emoji 作为图标
- 颜色作为唯一状态指示

## 交付前检查清单

- [ ] 无 emoji 图标（使用 SVG：Heroicons/Lucide）
- [ ] 所有可点击元素有 cursor-pointer
- [ ] Hover 状态 150-300ms 平滑过渡
- [ ] 亮色模式文字对比度 ≥ 4.5:1
- [ ] 键盘导航焦点可见
- [ ] 响应式：375/768/1024/1440px
