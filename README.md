# YOUR_STORY_TITLE

项目简介：以中国劳动者为题的互动叙事新闻，读者在多分支现场中做选择，体验现实约束、风险与后果，并可重启对比路径。

## 技术栈
- Nginx 静态托管
- Vanilla HTML
- Alpine.js（CDN）
- Tailwind CSS（CDN）
- JSON 故事数据（`/assets/story.json`）

## 项目结构
```text
.
├── index.html
├── assets
│   ├── app.js
│   ├── images
│   │   ├── .gitkeep
│   │   └── README.md
│   ├── story.json
│   └── styles.css
├── nginx
│   └── nginx.conf
└── README.md
```

## 本地运行（任意静态服务器）
在项目目录启动一个静态服务，并访问 `index.html`。

示例（Python 自带静态服务）：
```bash
python3 -m http.server 8080
```
然后打开：`http://localhost:8080`

## 使用 Nginx 运行
1. 准备站点目录，例如：`/usr/share/nginx/html`
2. 将本项目文件复制到该目录（保留 `index.html` 与 `assets/`）
3. 使用 `nginx/nginx.conf` 作为配置启动 Nginx

示例：
```bash
nginx -c /path/to/project/nginx/nginx.conf
```
访问：`http://localhost:8080`

## 功能清单
- node graph 故事引擎（`title/start/initial_state/nodes`）
- 可扩展全局文案：`intro`（导语）与 `trigger_warning`（首页触发警告）
- 状态系统：数值增减、布尔/字符串赋值
- 安全条件表达式：tokenizer + shunting-yard + 安全求值（无 `eval`）
- markdown-lite：`**bold**`、`*italic*`、`[text](url)`（URL 白名单）
- 单页交互：继续、返回、重来、路径记录
- 本地持久化：当前节点、状态、历史、undo 快照
- 运行时验证：字段、目标节点、可达性检查（含不可达 warning）
- 严重错误时页面显示简体中文提示

## 内容撰写文档
- Story nodes 详细写法见：`STORY_NODES_GUIDE.md`

## 本地图片引用
- 建议将图片放在 `assets/images/`
- 在节点里使用本地路径：
  - `"image": "/assets/images/节点ID.jpg"`
- 示例：`assets/story.json` 中 `b1_start` 已配置本地图片字段

## 免责声明
故事为虚构模拟体验，不对应真实人物或真实事件。
