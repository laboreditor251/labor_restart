# Story Nodes 撰写指南（当前实现版）

本文档说明当前项目 `assets/story.json` 的可用字段、语法与最佳实践。

## 1. 顶层结构

```json
{
  "title": "重生之我还是打工人",
  "intro": "导语（可选）",
  "trigger_warning": "首页提示（可选）",
  "start": "pre_origin",
  "initial_state": {
    "money": 0,
    "trust": 0,
    "risk": 0,
    "paperTrail": 0,
    "seed": 0,
    "parent_job": "",
    "education": "",
    "destination": "",
    "personality": "",
    "priority": "",
    "branch": "",
    "role": "",
    "mgmt_response_roll": 0
  },
  "nodes": {
    "node_id": {
      "text": "节点正文",
      "image": "/assets/images/node.jpg",
      "choices": [
        {
          "label": "选项文案",
          "to": "next_id",
          "condition": "money >= 0 AND risk < 3",
          "effects": { "risk": 1, "role": "工人" },
          "note": "条件不足时提示"
        }
      ],
      "auto_routes": [
        {
          "condition": "seed >= 2",
          "to": "branch_a",
          "effects": { "branch": "b1" }
        },
        {
          "to": "branch_b"
        }
      ],
      "ending": {
        "type": "neutral",
        "title": "结局标题",
        "text": "结局文本"
      }
    }
  }
}
```

字段说明：
- `title`：故事标题（必填）。
- `intro`：导语（可选，markdown-lite）。
- `trigger_warning`：首页触发警告（可选，markdown-lite）。
- `start`：起始节点 id（必填，必须存在）。
- `initial_state`：初始状态（primitive：number/boolean/string）。
- `nodes`：节点字典（key 为节点 id）。

## 2. Node 字段

### 2.1 `text`（建议必填）
- 类型：`string`
- 支持 markdown-lite：
  - `**加粗**`
  - `*斜体*`
  - `[文本](url)`
- 渲染安全：先 HTML escape，再转 markdown-lite；链接仅允许 `http/https/mailto`。

### 2.2 `image`（可选）
- 类型：`string`
- 推荐本地路径：
  - `"/assets/images/节点ID.jpg"`
  - `"./assets/images/scene-01.png"`
- 建议统一放在：`assets/images/`

### 2.3 `choices`（常规节点必填）
- 类型：`array`
- 空数组 `[]` 常用于结局节点。

### 2.4 `auto_routes`（可选）
- 类型：`array`
- 用于“自动分流节点”（系统自动跳转，不让用户选）。
- **按顺序命中第一条**。
- 每项结构：

```json
{
  "condition": "destination == \"上海市中心\"",
  "to": "b6_guard",
  "effects": { "branch": "b6", "role": "私企销售" }
}
```

说明：
- `condition` 可省略；省略时相当于兜底规则。
- `to` 必填。
- `effects` 可选。

### 2.5 `ending`（可选）
- 用于标记该节点为结局。

```json
"ending": {
  "type": "good",
  "title": "结局标题",
  "text": "补充说明"
}
```

- `type` 建议仅用：`good` / `bad` / `neutral`。

## 3. Choice 字段

```json
{
  "label": "选择文案",
  "to": "目标节点ID",
  "condition": "条件表达式（可选）",
  "effects": {
    "money": -200,
    "risk": 1,
    "injured": true,
    "role": "外卖骑手",
    "seed": "rand(0,3)"
  },
  "note": "条件不满足时提示（可选）"
}
```

- `label`：必填。
- `to`：必填，目标节点必须存在。
- `condition`：可选，不满足则按钮禁用。
- `effects`：可选，进入下一节点前应用。
- `note`：可选，条件不满足时显示。

## 4. condition 表达式语法

不使用 `eval/Function`，使用安全解析器。

支持：
- 比较：`== != > >= < <=`
- 逻辑：`AND OR NOT`
- 括号：`(` `)`
- 字面量：number / boolean / string
- 状态引用：直接写 key（如 `risk`, `education`）

示例：

```txt
risk >= 2 AND paperTrail >= 1
destination == "珠三角工业区" AND seed >= 2
NOT (education == "研究生+")
```

## 5. effects 语义

### 5.1 number：累加

```json
"effects": { "risk": 1, "money": -200 }
```

- number 不是“赋值”，而是“加减”。
- 若原值非 number，按 `0` 起算。

### 5.2 string / boolean：直接覆盖

```json
"effects": { "role": "工人", "injured": true }
```

### 5.3 随机数（当前实现支持）

```json
"effects": { "seed": "rand(0,3)" }
```

- 表示将 `seed` 设置为区间整数随机值（含端点）。
- 可用于分流抽签、随机事件。

## 6. 连续步骤节点（强代入推荐）

当某段信息很长（例如维权流程），建议拆成连续节点，每步只有“继续”：

```json
"x_step0": {
  "text": "你决定申请工伤补偿。",
  "choices": [{ "label": "继续", "to": "x_step1" }]
},
"x_step1": {
  "text": "【第一步】……",
  "choices": [{ "label": "继续", "to": "x_step2" }]
}
```

这样比一屏长文更易读，也更有过程感。

## 7. 运行时校验

`loadStory()` 后会进行校验。

会报错（阻止运行）：
- 缺少 `title`
- 缺少 `start`
- `nodes` 缺失或为空
- `start` 指向不存在节点

会警告（不阻止）：
- 节点缺少 `text`
- 节点缺少 `choices`
- `choice` 缺 `label` 或 `to`
- `choice.to` 指向不存在节点
- 从 `start` 不可达节点

建议写完后查看控制台 warning。

## 8. 常见坑

1. 把写作标记暴露给玩家
- 如 `A-A-A`、`转A` 不应出现在 `label`。

2. 用了 JS 逻辑符
- 错：`&&` / `||`
- 对：`AND` / `OR`

3. number effects 写成“赋值”
- `"money": 500` 在当前实现里是 `+=500`。

4. 结局节点还挂复杂分支
- 建议结局节点 `choices: []`，语义更清晰。

## 9. 最小模板

```json
{
  "text": "你来到一个新的现场。",
  "image": "/assets/images/sample.jpg",
  "choices": [
    {
      "label": "保守处理",
      "to": "n_safe",
      "condition": "paperTrail >= 1",
      "effects": { "paperTrail": 1 },
      "note": "你需要先留存基础证据。"
    },
    {
      "label": "直接推进",
      "to": "n_risk",
      "effects": { "risk": 1 }
    }
  ]
}
```

## 10. 提交前检查

1. 每个 `to` 是否都存在。
2. 长段流程是否拆成连续节点。
3. 文案里是否还有作者内部标记（如 `A-A`）。
4. 链接是否可打开、是否为 `http/https/mailto`。
5. 本地图片路径是否真实存在于 `assets/images/`。
6. 打开页面走一遍关键分支，确认无断链。
