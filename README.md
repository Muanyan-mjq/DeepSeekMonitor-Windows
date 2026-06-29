# DeepSeek Monitor Windows

Windows 桌面端 DeepSeek API 用量监控工具。实时查看账户余额、当月消费、模型 Token 用量和最近 7 天用量趋势，系统托盘常驻。

> Fork 自 [Joyi-code/DeepSeekMonitorWindows](https://github.com/Joyi-code/DeepSeekMonitorWindows)（v1.1.0），感谢原作者 [Joyi-code](https://github.com/Joyi-code) 的开源工作。原项目基于 [JayHome137/deepseek-monitor](https://github.com/JayHome137/DeepSeekMonitor) 的思路做 Windows 适配。

郑重声明：本项目不是 DeepSeek 官方产品。

## 页面截图

点击仪表盘 Shirt 按钮循环切换 7 套主题。

<p>
<img src="screenshots/theme-light.png" width="200" height="320">
<img src="screenshots/theme-sunset.png" width="200" height="320">
<img src="screenshots/theme-dark.png" width="200" height="320">
<img src="screenshots/theme-ocean.png" width="200" height="320">
<img src="screenshots/dashboard.png" width="200" height="320">
<img src="screenshots/theme-forest.png" width="200" height="320">
<img src="screenshots/theme-sakura.png" width="200" height="320">
</p>

> 如果您有其他喜欢的配色方案，欢迎提交 [Issue](https://github.com/Muanyan-mjq/DeepSeekMonitor-Windows/issues) 或 PR。

## 相较原项目的改动

### 7 主题切换

原项目仅暗色/亮色两套。本 fork 扩展为 7 套，每套独立设计面板底色、卡片渐变、品牌强调色、Flash/Pro 模型色、图表分段色。点击 Shirt 按钮循环切换，所有视觉元素同步变化。

### 余额告警

设置页可配置余额告警线。当账户余额低于阈值时，仪表盘余额卡片状态变为橙色「余额偏低」，同时弹出告警提示条显示当前余额与告警线对比。7 套主题各有独立告警配色，确保不同背景下清晰可见。

### 缓存命中率

命中率精确到小数点后两位。

## 功能

- 查询 DeepSeek API 账户余额（官方接口）
- 平台用量数据：当月消费、Token 总量、请求数、缓存命中/未命中、输出 Token
- V4 Flash / V4 Pro 两类模型独立展示
- 最近 7 天消费趋势堆叠柱状图，支持悬停查看明细
- **7 主题循环切换**（暗色 / 亮色 / 海洋蓝 / 森林绿 / 暖金日落 / 樱花粉 / 薰衣草紫）
- **余额告警**（可配置阈值，仪表盘实时提醒）
- 系统托盘常驻，不占任务栏
- API Key 本地保存，不上传任何第三方
- 用量 Token 网页登录自动同步 + 手动粘贴兜底
- 自动刷新（1 分钟 / 5 分钟 / 30 分钟 / 1 小时）

## 使用说明

1. **配置 API Key**：打开设置页，输入 DeepSeek 开放平台的 API Key，点击验证并保存
2. **同步用量**：点击「网页登录自动同步」完成 DeepSeek 登录，Token 自动从缓存提取；也可手动从浏览器控制台获取后粘贴
3. **开启余额告警**：设置页 → 余额告警区块，输入告警线金额（如 10.00）并保存

所有数据存储在本地 `%APPDATA%\DeepSeekMonitorWindows\config.json`，不会上传。

## 系统要求

- Windows 10 或 Windows 11
- Microsoft Edge WebView2 Runtime（Win11 已内置）

## 安装

从 [Releases](https://github.com/Muanyan-mjq/DeepSeekMonitor-Windows/releases) 下载最新 `.exe` 安装包运行。

## 本地开发

```powershell
git clone https://github.com/Muanyan-mjq/DeepSeekMonitor-Windows.git
cd DeepSeekMonitorWindows
npm install
npx tauri dev
```

需要 Node.js 18+、Rust 1.77.2+、Visual Studio Build Tools 2022（含 Desktop development with C++）。

## 构建

```powershell
npm run build
```

产物：`src-tauri/target/release/bundle/nsis/DeepSeekMonitorWindows_x64-setup.exe`

## 技术栈

Tauri 2 + React 18 + TypeScript + Rust

## 许可证

MIT License
