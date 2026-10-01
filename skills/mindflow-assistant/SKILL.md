---
name: mindflow-assistant
description: |
  读写和修改 MindFlow Neural 神经思维导图 HTML 文件。当用户说"思维导图"、
  "神经导图"、"mindflow"、"添加模块"、"AI 分析思维导图"、"帮我改思维导图"时触发。
  不用于：新建一个普通 HTML 文件、一般的代码问题。
---

# MindFlow Neural 思维导图助手

## Inputs to collect

- 思维导图文件路径（用户没说就问，常见路径如 `I:\神经思维导图\3.0\index.html`）
- 用户的意图：读结构 / 添加内容 / 修改内容 / AI 分析 / 导出数据

## 文件与数据源（重要）

- 主文件：`I:\神经思维导图\3.0\index.html`（约 3500 行，Electron 通过 main.js 加载）
- **页面运行时数据源是 localStorage**（key=`mindflow_data_neural`），不是 index.html 里的
  DEFAULT_DATA。`loadData()` 优先读 localStorage，无数据才回退 DEFAULT_DATA。
  因此：改 index.html 里的 DEFAULT_DATA 只影响全新用户；已有数据的用户要改内容，
  应通过浏览器控制台执行 JS 操作 `data` 后 `saveData(); render();`，
  或读取/修改 `mindflow.json`（导出文件，含模块与 _x/_y 坐标）。
- 数据导出文件：`mindflow.json` / `mindflow_export.json`（JSON，模块数组带坐标）。

## 文件结构速查（行号按 2026-10 版实测，index.html 共 3497 行）

```
- 第 595 行：COLORS 颜色常量（10色）
- 第 597 行：DEFAULT_DATA（默认模块数据模板）
- 第 646 行：ensureHRModule（启动时自动补"人力资源"模块，若缺失）
- 第 694 行：injectSkillLinkModule（⚠️ 启动注入：若模块无 id 以 'sl' 开头，
  会把 data.title 重置为 'SkillLink 创业计划书' 并覆盖全部模块！修改时注意）
- 第 665-691 行：HISTORY_KEY / MAX_HISTORY / 撤销栈 / 画布常量
  （CX=600, CY=400, CENTER_R=55, BR_R=40, CH_W=140, CH_H=32）
- 第 688 行：CX/CY 画布中心常量
- 第 760 行：aiConfig 配置（base/key/model/temp，存 localStorage mf_ai_*）
- 第 799-818 行：loadData() / saveData()
- 第 820-900 行：历史画布（loadHistory/saveHistory/addToHistory/restoreHistory）
- 第 905-930 行：computeLayout() 布局计算（环形分布）
- 第 932-974 行：getNeuralConnections()（关键词匹配神经网络连线）
- 第 999 行：render() 画布渲染
- 第 1314-1423 行：轨道动效（toggleOrbitMode / runOrbitTick）
- 第 1429-1559 行：手动连线模式（toggleConnectMode / createConnection / drawCustomEdges）
- 第 1573-1816 行：媒体节点（renderMediaItems / insertMediaFromFiles / 拖拽插入）
- 第 1821-1843 行：requestAIStream（AI 请求核心）
- 第 1848-1945 行：AI 优化（aiOptimizeCurrentModal / aiOptimizeModule）
- 第 1950-2017 行：AI 补全模块（aiSuggestModules）
- 第 2022-2104 行：分析记录（analysisRecords，localStorage 'mf_analysis_records' 上限50）
- 第 2106-2169 行：aiRelationAnalysis（全模块关联分析）
- 第 2188-2316 行：findRelatedModules / aiAnalyzeSelectedRelations（定向关联分析）
- 第 2318-2458 行：拓扑图/报告/聚类渲染（_doRenderTopology / renderReport / renderCluster）
- 第 2463-2849 行：AI 聊天（triggerChatInteraction，支持 JSON 指令操作）
- 第 2946-3038 行：select / updateLegend / updateStats / updateDetail
- 第 3062-3200 行：弹窗逻辑（openAddModal / openEditModal / saveModal / 标签）
- 第 3213-3239 行：newCanvas（新建画布，清空+存历史）
- 第 3241-3360 行：工具栏按钮事件绑定
- 第 3432-3460 行：主题（initTheme / toggleTheme，localStorage 'mf_theme'）
- 第 3465-3497 行：autoConfigMiniMax 一键配置提示（含硬编码 API key，勿外泄/勿引用）
- 工具栏按钮 ID（HTML 第 385-416 行）：
  btn-theme / btn-ai-setup / btn-export / btn-import / btn-edit / btn-neural /
  btn-connect / btn-orbit / btn-add / btn-del / btn-ai-optimize / btn-ai-suggest /
  btn-ai-relation / btn-new-canvas / btn-undo / btn-fit / btn-insert-media
- 左侧栏：tab-outline（结构大纲）/ tab-history（历史画布）
- 右侧栏：tab-detail（属性详情）/ tab-chat（AI 聊天）/ tab-records（分析记录）
```

## 数据结构速查

模块数据结构（data.modules 数组）：
```javascript
{
  id: '唯一ID',
  title: '模块名称',
  color: '#十六进制颜色',
  items: [{ t: '子项标题', d: '描述', n: '备注', tags: [] }],
  note: '模块备注',
  _x: 画布X坐标,  // 布局自动计算，也可手动指定
  _y: 画布Y坐标
}
```

子项标签 tags 数组元素：
```javascript
{ type: 'text', text: '文字标签' }   // 文字标签
{ type: 'image', src: 'data:image/...' }  // 图片标签（base64）
```

媒体 data.media 数组：
```javascript
{ id, type: 'image'|'video'|'audio', src, name, width, height, _x, _y }
```

手动连线 data.connections（customEdges）数组：
```javascript
{ id, from: 'item-{modId}-{idx}', to: 'item-{modId}-{idx}', color, label }
```

可用颜色常量：`COLORS`（第 595 行）
`['#E8630A','#2E7D32','#1565C0','#6A1B9A','#00838F','#558B2F','#C62828','#F57F17','#AD1457','#455A64']`

## 常用修改模式

**添加模块**：`data.modules.push({id:String(Date.now()), title, color, items, note})` 后
`computeLayout(); saveData(); render();`

**修改模块**：直接改 `data.modules` 中对应项，然后 `saveData(); render();`

**删除模块**：`data.modules = data.modules.filter(m => m.id !== targetId);`

**添加子项**：`mod.items.push({ t:'标题', d:'描述', n:'', tags:[] })`

**修改子项**：直接改 `mod.items[idx]`

**添加/删除标签**：`item.tags.push({type:'text', text})` 或 `item.tags.splice(i,1)`

**插入媒体**：`data.media.push({id:'media_'+Date.now(), type:'image', src:base64, name, width, height, _x, _y})`

**创建手动连线**：`customEdges.push({id:'conn_'+Date.now(), from:'item-A-0', to:'item-B-1', color:'#f97316', label:''}); saveData(); render();`

**保存撤销快照**：改数据前先 `saveUndo()`（栈上限 50，Ctrl+Z 可回退）

**触发 AI 全模块关联分析**：`aiRelationAnalysis()`

**触发 AI 定向关联分析**：先 `select(moduleId)` 再 `aiAnalyzeSelectedRelations()`

**触发 AI 优化选中模块**：`select(moduleId)` 后 `aiOptimizeModule()`

**触发 AI 补全**：`aiSuggestModules()`

**切换主题**：`toggleTheme()`（或改 `data-theme` 属性 + localStorage `mf_theme`）

**历史画布**：`addToHistory(data)` 保存快照；`restoreHistory(snapshot)` 恢复

## AI 请求与返回契约（requestAIStream，第 1821 行）

- 请求：`POST {aiConfig.base}/chat/completions`，Header `Authorization: Bearer {key}`，
  body `{model, messages:[{role:'system'},{role:'user'}], temperature, max_tokens:4000}`
- 返回：非流式，读 `resJson.choices[0].message.content`
- 解析：`extractJSON()` / `parseAIJSON()`（自动修复尾逗号）
- 各功能期望 AI 返回的 JSON：
  - **AI 优化**：`{name, items:[{t,d}], note}`（模块名 3-6 字）
  - **AI 补全**：`{suggestions:[{name, reason, color}]}`（最多3个，颜色限定）
  - **全模块关联分析**：`{modules:[{name,domain,summary}], relations:[{from,to,type,desc}], summary}`
    （type ∈ 数据流|业务协同|资源关联）
  - **定向关联分析**：`{focusModule, relations:[{target,type,desc,suggestion}], overallInsight}`
    （type ∈ 数据流|控制依赖|业务协同|资源关联|逻辑支撑）
  - **AI 聊天指令**：`{action:'add'|'addItem'|'optimize'|'setItems'|'delete'|'deleteItem', ...}`

## 写入文件

用 `edit` 工具精准替换（old_string 必须精确匹配文件内容，包括缩进）。修改后用 `grep` 验证没有破坏周围代码。

## 亮色/暗色主题

- 主题由 `data-theme` 属性控制（`document.documentElement.setAttribute('data-theme', 'light'|'dark')`）
- 偏好保存在 `localStorage.setItem('mf_theme', theme)`
- 按钮图标：暗色=🌙，亮色=☀️，通过 `updateThemeIcon()` 更新

## Output contract

- 直接修改 `I:\神经思维导图\3.0\index.html` 文件（或通过浏览器控制台操作 localStorage 数据）
- 修改完成后告知用户刷新浏览器生效
- 如果需要可导出修改后的 JSON 数据

## Failure handling

- 文件路径错误 → 询问用户正确路径
- 修改后语法错误 → 用 `grep` 检查最近的修改块，确保括号/引号平衡
- 特别注意：`requestAIStream` 里的 `try {` 和 `} catch` 要成对
- 不要大幅重写整个 JS，只做精准的插入/替换
- 如果涉及大型重构（如本次 UI 重构），用子 agent 委托执行
- ⚠️ 若用户已有 localStorage 数据，仅改 DEFAULT_DATA 不会生效，需同时说明数据源在 localStorage

## Examples

**用户**："在思维导图里添加一个'人力资源'模块，包含招聘和培训两个子项"

→ 读取文件 → 找到 data.modules.push 位置（或浏览器控制台操作 data）→ 添加新模块对象 →
`computeLayout(); saveData(); render();` → 告知刷新

**用户**："切换到暗色主题"

→ 直接调用 JS：`document.documentElement.setAttribute('data-theme', 'dark')`

**用户**："把'财务规划'模块下的'成本管控'子项改成'成本优化'，描述改成'降低运营支出'"

→ 读取文件 → 找到该模块 → 修改 items 数组 → saveData + render

**用户**："打开关联分析"

→ 直接调用 `aiAnalyzeSelectedRelations()` 或 `aiRelationAnalysis()`

**用户**："在画布上插入一张图片/视频"

→ `insertMediaFromFiles(files)` 或推送 `data.media` 后 render（支持 SVG/PNG/JPG/MP4/MP3）

**用户**："给'市场营销'模块添加文字标签"

→ `mod.items[i].tags.push({type:'text', text:'标签'})` 后 saveData + render

## Windows (win32) platform notes

文件路径含中文，用完整路径引用：
`I:\神经思维导图\3.0\index.html` → `I:\神经思维导图\3.0\index.html`（直接写路径字符串即可，read/edit 工具支持中文路径）
