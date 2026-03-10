# json-to-llm-context

把 `JSON` / PostgreSQL `jsonb` 转成更适合大模型读取的紧凑上下文。

这个仓库包含一个完整的 skill 包，目标是：
- 减少直接喂原始 JSON 时的 token 浪费
- 保留实体、状态、关系、数量和样本
- 用更稳定的摘要报告风格提升 LLM 理解效果

## 包含内容

```text
json-to-llm-context/
├── SKILL.md
├── agents/openai.yaml
├── references/rules.md
└── scripts/json_to_readable_context.py
```

- `SKILL.md`：skill 说明、触发场景、用法
- `agents/openai.yaml`：UI 元数据
- `references/rules.md`：摘要规则和风格说明
- `scripts/json_to_readable_context.py`：核心转换脚本

## 适用场景

适合：
- API 返回的 JSON
- 数据库导出的 `jsonb`
- 深层嵌套对象
- 大数组 / 列表型结构
- 需要压缩后喂给 LLM 的结构化数据

不适合：
- PDF / DOCX
- 图片 OCR
- 普通长文本文章

## 快速开始

### 直接读取文件

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py --input payload.json
```

### 从 stdin 读取

```bash
cat payload.json | python3 json-to-llm-context/scripts/json_to_readable_context.py
```

### 输出到文件

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --output summary.txt
```

## 输出风格

### 默认：`sectioned`

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --style sectioned
```

示例：

```text
User[123]: Tom

Summary
- Status: active.
- Profile: a@b.com.

Collections
- Roles: 2 total; values: admin and editor.
```

### 简化：`flat`

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --style flat
```

示例：

```text
User[123]: Tom
- Status: active.
- Profile: a@b.com.
- Roles: 2 total; values: admin and editor.
```

## 安全控制

### `--strict`

更保守地压缩，尽量保留更多显式结构。

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --strict
```

### `--preserve`

强制保留指定 key 或 dotted path，即使这些值本来会被压缩掉。

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --preserve status,profile.email,orders
```

### `--expand`

对摘要进行局部展开，避免压缩过头。

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --expand collections
```

可选值：
- `collections`
- `details`
- `all`

### `--show-paths`

给每一行增加来源路径标记，方便模型或人类回溯来源。

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --show-paths
```

示例：

```text
User[123]: Tom [@root]

Summary
- Status: active. [@active]

Collections
- Roles: 2 total; values: admin and editor. [@roles]
```

## 常用组合

### 给 LLM 的默认推荐

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --style sectioned
```

### 保守压缩

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --style sectioned \
  --strict \
  --preserve status,profile.email
```

### 保留来源路径

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --style sectioned \
  --show-paths
```

### 对复杂对象局部展开

```bash
python3 json-to-llm-context/scripts/json_to_readable_context.py \
  --input payload.json \
  --style sectioned \
  --expand all \
  --max-samples 2
```

## 参数说明

| 参数 | 说明 |
| --- | --- |
| `--input` | 输入 JSON 文件路径；省略时从 stdin 读取 |
| `--output` | 输出文件路径；省略时输出到 stdout |
| `--style` | `sectioned` 或 `flat` |
| `--max-samples` | 数组样本数量上限 |
| `--max-depth` | 递归展开深度上限 |
| `--max-string-len` | 长字符串截断长度 |
| `--strict` | 保守压缩模式 |
| `--preserve` | 强制保留 key / path |
| `--expand` | 局部展开 `collections` / `details` / `all` |
| `--show-paths` | 附加来源路径标记 |

## 设计原则

- 默认输出给 LLM 更友好的**摘要报告风格**
- 尽量保留高价值字段：`id`、`name`、`status`、时间、关系、计数
- 对大数组优先输出：
  - 总数
  - 状态分布
  - 少量样本
- 避免直接倾倒原始路径树
- 需要更稳时使用：
  - `--strict`
  - `--preserve`
  - `--expand`
  - `--show-paths`

## 错误处理

- 非法 JSON：明确报错退出
- 空输入：报错退出
- 深层对象：用 `--max-depth` 限制展开
- 超长文本：自动截断并保留长度提示

## 本地验证

```bash
python3 -m py_compile json-to-llm-context/scripts/json_to_readable_context.py
```

## License

当前仓库未单独声明许可证；如需开源发布，建议补充 `LICENSE` 文件。
