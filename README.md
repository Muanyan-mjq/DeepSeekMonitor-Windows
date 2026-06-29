# DeepSeek Monitor Windows

Windows 桌面端 DeepSeek API 用量监控工具。实时查看账户余额、当月消费、模型 Token 用量和最近 7 天用量趋势，支持系统托盘常驻。

> Fork 自 [Joyi-code/DeepSeekMonitorWindows](https://github.com/Joyi-code/DeepSeekMonitorWindows)

## 预览

### 仪表盘

![仪表盘](screenshots/dashboard.png)

### 7 主题切换

| 暗色 | 亮色 | 海洋蓝 | 森林绿 | 暖金日落 | 樱花粉 | 薰衣草紫 |
|------|------|--------|--------|----------|--------|----------|
| ![暗色](screenshots/theme-dark.png) | ![亮色](screenshots/theme-light.png) | ![海洋蓝](screenshots/theme-ocean.png) | ![森林绿](screenshots/theme-forest.png) | ![暖金日落](screenshots/theme-sunset.png) | ![樱花粉](screenshots/theme-sakura.png) | ![薰衣草紫](screenshots/theme-lavender.png) |

### 设置页

![设置](screenshots/settings.png)

## 功能

### 原有功能

- **余额查询** — 通过 DeepSeek 官方接口实时查询账户余额
- **用量统计** — 当月消费金额、V4 Flash / V4 Pro 模型 Token 用量（输入/输出）、请求数
- **趋势图表** — 最近 7 天消费趋势堆叠柱状图，支持缓存命中/未命中/输出明细悬停查看
- **系统托盘** — 窗口隐藏到托盘，不占任务栏空间，左键托盘图标可切换显示
- **自动刷新** — 支持 1 分钟 / 5 分钟 / 30 分钟 / 1 小时间隔自动拉取数据

### 新增功能

- **7 主题切换** — 点击仪表盘 Shirt 按钮循环切换。暗色 / 亮色 / 海洋蓝 / 森林绿 / 暖金日落 / 樱花粉 / 薰衣草紫，共 7 套配色，每个主题的图表色、强调色、告警色均独立设计
- **余额告警** — 设置页可配置余额告警线。当账户余额低于设定值时，仪表盘余额卡片状态变为橙色警告，并弹出告警提示条。7 个主题各有独立告警配色，确保不融于背景
- **缓存命中率** — 精确到小数点后两位，直观反映输入 Token 的缓存复用情况

## 使用方式

### 1. 配置 API Key

打开设置页 → API Key 区块 → 输入 DeepSeek 开放平台的 API Key → 点击「验证并保存」。API Key 仅保存在本机，不上传任何第三方。

### 2. 同步用量数据

用量数据需要网页登录 Token（DeepSeek 未开放官方用量 API）。

**方式一（推荐）：** 点击「网页登录自动同步」→ 在弹窗中完成 DeepSeek 登录 → Token 自动从缓存提取

**方式二（兜底）：** 浏览器登录 DeepSeek 后 F12 控制台执行 `JSON.parse(localStorage.userToken).value` → 复制 Token → 手动粘贴保存

### 3. 开启余额告警

设置页 → 余额告警区块 → 输入告警线金额（如 ¥10.00）→ 保存。当余额低于该值时仪表盘自动提示。

## 技术栈

| 层级 | 技术 |
|------|------|
| 桌面框架 | Tauri 2 |
| 前端 | React 18 + TypeScript |
| 后端 | Rust |
| 构建 | Vite 5 |

## 本地运行

```bash
# 安装依赖
npm install

# 启动开发模式
npx tauri dev
```

需要 Node.js 18+、Rust 1.77.2+、Visual Studio Build Tools 2022（含 Desktop development with C++）。

## 许可证

MIT License
