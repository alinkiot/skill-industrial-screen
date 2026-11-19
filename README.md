# skill-industrial-screen

创建工业监控大屏与 SCADA/HMI 的 AI Skill。核心是"控制 + 操作防误"：工艺流程、设备启停、SBO 两步确认、Faceplate 操作小面板、联锁闭锁，遵循 ISA-101 灰度哲学和 P&ID 图元库规范。

输出 1920×1080 自适应的独立 HTML，使用本地 Vue 3、ECharts 5、Three.js，离线可运行，不依赖 CDN。通用数据看板、3D 展厅类需求请使用姊妹仓库 [skill-bigscreen-studio](https://github.com/alinkiot/skill-bigscreen-studio)。

## 五种模式

| 模式 | 形态 | 校验脚本 |
| --- | --- | --- |
| `dashboard` | 只读工业驾驶舱（工艺、资产、安全态势） | `validate-dashboard.js` |
| `hmi-classic` | 西门子博途风格经典上位机（ISA-101 灰度） | `validate-hmi.js` |
| `scada` | 可写控制 Web SCADA，含 SBO、Faceplate、联锁 | `validate-scada.js` |
| `light-3d` | 浅色 3D 水务数字孪生 HMI | `validate-light-3d-hmi.js` |
| `local-panel` | 单机就地控制屏（LCP / 现场触控 HMI） | `validate-dashboard.js` |

此外支持把成品场景集成进 Vue 3 + Vite 的边缘网关前端，见 `references/gateway-integration.md`。

## 目录结构

```text
SKILL.md          Skill 入口（模式判断、共享输出契约、硬规则）
references/       ISA-101、控制安全契约、Faceplate、报警生命周期、点表、P&ID 与图元库等 16 份文档
scripts/          校验脚本，以及图元库、位图图标库、图元画廊的构建脚本
assets/
  base-dashboard.html   基础页面骨架
  symbol-library/       154 个 FUXA 矢量工艺图元（currentColor，可随状态变色）
  vendor/               本地依赖：Vue 3 / ECharts 5 / Three.js / OrbitControls（约 1.7 MB）
  iotview-icons/        1530 个 iotview 位图图标（约 118 MB，不可改色）
out/              示例场景与输出目录
```

## 安装

将本仓库放到 AI 工具的 skills 目录下，目录名使用 `industrial-screen`，或打包为 zip 上传：

```bash
git clone git@github.com:alinkiot/skill-industrial-screen.git industrial-screen
```

仓库约 119 MB，主要来自位图图标。若目标平台有上传大小限制，可去掉 `assets/iotview-icons/`，并同步删除 `SKILL.md` 与 `references/raster-icon-library.md` 中对它的引用。

## 用法

向 AI 描述需求即可，例如"做一个污水厂 SCADA 操作终端"。Skill 会按需求选择模式，读取对应参考文档，生成页面到 `out/<场景名>/index.html`。生成后运行对应校验脚本：

```bash
node scripts/validate-dashboard.js out          # dashboard / local-panel
node scripts/validate-hmi.js out                # hmi-classic
node scripts/validate-scada.js out              # scada
node scripts/validate-light-3d-hmi.js out       # light-3d
```

重新生成图元库资料：

```bash
node scripts/build-symbol-library.js
node scripts/build-raster-icons.js
node scripts/gen-symbol-gallery.js
```

### 模式标记与目录校验

`out/` 会混放多种模式的产物，目录校验按模式过滤。非 dashboard 页面在 `<head>` 加一行标记：

```html
<meta name="screen-mode" content="scada">   <!-- 或 hmi-classic / light-3d / local-panel -->
```

`validate-dashboard.js out` 会跳过带其他模式标记的页面和参考页 `symbol-gallery/`；`validate-scada.js out` 只校验标记 `scada` 的页面。显式传单个文件时始终按该脚本契约校验。这样一条目录命令即可覆盖全部产物，不会误报。

## 测试产物（实测记录）

`out/` 中随仓库提供并经校验脚本实测通过的交付样例，可直接双击打开：

| 目录 | 模式 | 校验 |
| --- | --- | --- |
| `water-plant-overview/` | dashboard · 污水厂运行态势（工艺流程主视觉 + 水质趋势） | `validate-dashboard.js` 通过 |
| `pump-station-scada/` | scada · 加压泵站操作终端（Faceplate + SBO 两步确认 + 审计日志 + 消音确认） | `validate-scada.js` 通过 |

`out/` 另有 5 个既有 dashboard 样例（换热站、热泵站、设备健康、污水厂写实、管网）与 1 个参考图元画廊，全部通过各自校验。目录级命令：

```bash
node scripts/validate-dashboard.js out   # 6 通过
node scripts/validate-scada.js out       # 1 通过
```

## 共享约定

- 颜色只表达一种语义：青蓝为主数据，绿为正常，琥珀为警告，红为报警，灰为离线或未知。
- 画布固定 1920×1080 并等比缩放，保留拼接屏安全边距。
- 主视觉用内联 SVG，图表只承担趋势、分布和对比。
- 演示数据需明确标注，未确认数据协议前不伪造现场接入。

## 许可

与原 big-screen 工作区一致，见上游仓库 LICENSE。
