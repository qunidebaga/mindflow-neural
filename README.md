# MindFlow Neural · 神经思维导图

> 基于神经网络可视化理念的智能思维导图：模块化节点 + 子项 + 手动/自动连线 + AI 优化与关联分析。

## 功能特性

- **模块化神经网络画布**：中心标题 + 环形模块节点，每个模块挂载子项矩形，颜色区分业务域
- **神经网络连线模式**：关键词匹配自动生成模块间依赖连线（青色虚线）
- **手动连线**：编辑模式下点击两个子项即可创建自定义连线
- **AI 能力**（OpenAI 兼容接口）：
  - 智能优化模块（子项重组）
  - AI 补全新模块
  - 全模块关联分析（拓扑图 / 分析报告 / 聚类视图）
  - 定向关联分析（选中模块 vs 全图）
  - AI 聊天（自然语言 + JSON 指令：添加模块 / 增删子项 / 优化 / 重命名）
- **媒体节点**：拖拽或点击插入 SVG / PNG / JPG / MP4 / MP3 到画布
- **轨道动效**：模块沿轨道浮动展示
- **主题切换**：深色 / 浅色
- **历史画布**：自动快照，最多 20 份，随时恢复
- **导入 / 导出 JSON**：数据可迁移、可备份

## 快速开始

### 方式一：浏览器直接打开

双击 `app/index.html` 即可使用（数据保存在浏览器 localStorage）。

### 方式二：Electron 桌面应用

```bash
cd app
npm install        # 安装 electron-builder 等依赖
npm start          # 启动桌面应用
```

桌面版额外提供快捷键：

| 快捷键 | 功能 |
|---|---|
| Ctrl/Cmd + N | 新建画布 |
| Ctrl/Cmd + E | 导出数据 |
| Ctrl/Cmd + I | 导入数据 |
| Ctrl/Cmd + Shift + E | 编辑模式 |
| Ctrl/Cmd + 0 / 1 | 重置视图 / 铺满视图 |
| Ctrl/Cmd + Shift + N | 神经网络连线 |

## 配置 AI 接口

1. 点击右上角「配置 AI 接口」
2. 填入任意 **OpenAI 兼容** 接口：`Base URL`（如 `https://api.minimax.chat/v1`）、`API Key`、`模型名`（如 `abab6.5s-chat`）
3. 保存后即可使用 AI 优化 / 补全 / 关联分析 / 聊天

> 未配置时页面会提示一键配置入口（需自行填写 Key，代码不内置任何密钥）。

## 目录结构

```
mindflow-neural/
├── app/                         # 应用本体（纯前端，无构建依赖）
│   ├── index.html               # 全部功能单文件实现
│   ├── main.js                  # Electron 主进程（可选）
│   ├── package.json             # Electron 打包配置（可选）
│   └── mindflow.json            # 示例数据（中小企业核心需求全景图）
└── skills/                      # Agent Skill（让 AI 助手读写/生成该思维导图）
    ├── mindflow-assistant/      # 读写与修改思维导图文件的 Skill
    └── mindflow-neural-builder/ # 根据主题自动生成神经网络的 Skill
```

## 数据存储

- 页面数据保存在浏览器 `localStorage`（key: `mindflow_data_neural`）
- 分析记录：`mf_analysis_records`（上限 50 条）
- 历史画布：`mindflow_history_v1`（上限 20 份）
- 主题偏好：`mf_theme`

## Agent Skills 说明

仓库附带两个可直接安装的 Agent Skill（Doubao / Claude 等支持 Skill 机制的助手可加载）：

- **mindflow-assistant**：掌握文件结构速查（行号）、数据结构、AI 返回契约，可对思维导图文件做增删改
- **mindflow-neural-builder**：输入主题 → 自动生成模块网络 + 连线 → 注入页面 → 验证渲染

## License

MIT
