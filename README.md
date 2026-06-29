# DeepSeek Monitor Windows

Windows 桌面端 DeepSeek API 用量监控工具，实时查看余额、消费和 Token 用量。

> Fork 自 [Joyi-code/DeepSeekMonitorWindows](https://github.com/Joyi-code/DeepSeekMonitorWindows)，在其基础上增加了以下功能。

## 新增功能

- **7 主题切换** — 暗色 / 亮色 / 海洋蓝 / 森林绿 / 暖金日落 / 樱花粉 / 薰衣草紫，循环切换，每个主题独立适配告警配色
- **余额告警** — 设置页配置告警线，余额低于阈值时仪表盘显示告警
- **缓存命中率** — 精确到小数点后两位
- **告警卡片适配** — 7 套主题各有独立的告警配色

## 技术栈

Tauri 2 + React 18 + TypeScript + Rust

## 运行

```bash
npm install
npx tauri dev
```

## 许可证

MIT License
