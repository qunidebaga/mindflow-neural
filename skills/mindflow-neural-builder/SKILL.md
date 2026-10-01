---
name: mindflow-neural-builder
description: |
  控制 MindFlow Neural 神经思维导图自动生成"神经网络"（模块化思维导图）。
  当用户说"生成神经网络"、"自动生成思维导图"、"帮我创建脑图"、
  "根据XX主题生成思维导图"时触发。流程：确定主题与模块结构 → 生成
  modules/connections 数据 → 通过浏览器控制台注入 localStorage →
  刷新验证渲染。依赖 mindflow-assistant（文件结构/数据结构速查）。
  不用于：修改已有单个模块、一般问答。
---

# MindFlow Neural 神经网络生成器

## 触发与前置

- 用户提供主题（或由 agent 基于上下文拟定，如"AI数字文博3D项目"）
- 先读取 `mindflow-assistant` Skill 掌握文件结构/数据结构（本 Skill 不重复）
- 页面路径：`I:\神经思维导图\3.0\index.html`（数据源为 localStorage `mindflow_data_neural`）

## 生成规范

### 1. 模块结构（data.modules）
按主题拆 4-10 个模块，每个模块 3-6 个子项：
```javascript
{
  id: String(Date.now() + i),        // 唯一
  title: '模块名（3-6字）',
  color: COLORS[i % COLORS.length],  // 循环取色，相邻模块避免同色
  items: [
    { t: '子项标题', d: '一句话描述', n: '', tags: [] }
  ],
  note: '模块一句话备注'
}
```
颜色表（与页面 COLORS 一致）：
`['#E8630A','#2E7D32','#1565C0','#6A1B9A','#00838F','#558B2F','#C62828','#F57F17','#AD1457','#455A64']`

### 2. 手动连线（data.connections）
模块间有业务/依赖关系时生成：
```javascript
{ id: 'conn_' + Date.now() + '_' + k,
  from: 'item-{模块id}-{子项索引}',
  to: 'item-{模块id}-{子项索引}',
  color: '#f97316', label: '' }
```

### 3. 整体数据
```javascript
{ title: '主题名', modules: [...], media: [], connections: [...] }
```

## 注入执行（浏览器控制台路径，首选）

1. 打开页面：优先用浏览器访问 `file:///I:/神经思维导图/3.0/index.html`
   （中文路径若打不开，可在该目录起本地服务：`python -m http.server 8765`，
   访问 `http://localhost:8765/index.html`）
2. **先备份并存入历史**（防覆盖用户数据）：
   ```javascript
   var cur = localStorage.getItem('mindflow_data_neural');
   if (cur) { var d = JSON.parse(cur); if (typeof addToHistory === 'function' && d.modules && d.modules.length) addToHistory(d); }
   ```
3. **注入新数据**：
   ```javascript
   localStorage.setItem('mindflow_data_neural', JSON.stringify(新数据对象));
   ```
4. **刷新生效**：`location.reload()`；渲染后 `computeLayout()` 会自动计算坐标
5. **可选：打开神经网络连线模式**：`document.getElementById('btn-neural').click()`

## 验证

- 用浏览器截图确认：模块圆圈已环绕中心渲染、子项矩形正常、颜色区分
- 检查 console 无报错（`bu.console_messages()` 为空）
- 若开启神经网络连线，确认青色虚线连线可见
- 确认左下/右侧面板模块计数与注入数量一致

## 恢复用户原数据

- 方法一：点击页面左侧「历史画布」标签，找到注入前的时间快照点击恢复
- 方法二（控制台）：
  ```javascript
  localStorage.setItem('mindflow_data_neural', 备份字符串); location.reload();
  ```

## Failure handling

- 页面打不开（file:// 中文路径失败）→ 起本地 http server 后重试
- 注入后空白/报错 → 检查生成数据是否合法（modules 非空、items 数组、无多余字段）
- addToHistory 不可用（页面未初始化完）→ 等待 1-2 秒后重试，或改用外部备份文件
- 不要直接改 index.html 的 DEFAULT_DATA 作为主路径（localStorage 优先，不生效）

## Output contract

- 交付：注入完成 + 截图/在线验证 + 恢复方法说明
- 告知用户刷新浏览器即可看到新生成的神经网络
