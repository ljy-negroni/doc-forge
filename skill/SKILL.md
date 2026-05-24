# doc-forge — 软件工程文档优化工具链

## 触发方式

用户运行 `/doc-forge` 时执行本 skill。

---

## 工作流程

### 启动检测

检查当前目录下是否存在 `output/` 目录且包含已有产物：

- 存在 → 询问用户：
  ```
  检测到已有产物，请选择模式：
  [1] 全新生成（将创建新的时间戳目录）
  [2] 增量更新（选择要重新生成的模块）
      [a] 更新文档内容
      [b] 重新生成图表
      [c] 重新生成宣传图
  ```
- 不存在 → 直接进入问答流程

---

### 阶段 1 — 结构化问答

问答规则：
- 标 `*` 为必填，其余可输入 `skip` 跳过
- 输入 `?` 获取该题的详细说明
- 输入 `back` 返回上一题修改

逐题提问：

```
* Q1: 项目名称和一句话描述？
      示例：doc-forge — 一个自动生成开发文档和架构图的 CLI 工具

* Q2: 目标用户是谁？
      示例：独立开发者、小型团队的技术负责人

* Q3: 核心功能有哪些？（列举 3-5 个）
      示例：问答生成文档、PlantUML 图表渲染、GPT 宣传图生成

  Q4: 主要技术栈？（可 skip）
      示例：Node.js + OpenAI API + PlantUML

  Q5: 有哪些关键数据实体？（可 skip，输入 ? 查看说明）
      ? 说明：数据实体是系统中需要持久化的核心对象，如用户、订单、文章等
      示例：Project、Document、Diagram、Image

* Q6: 核心业务流程是什么？
      示例：用户输入项目信息 → 生成文档 → 渲染图表 → 输出产物
```

问答完成后，询问要生成的产物：

```
选择要生成的产物（默认 all）：
[1] 开发文档   [2] PlantUML 图表   [3] 宣传图 / 原型图   [all] 全部
```

---

### 阶段 2 — 生成文档草稿并优化

1. 根据问答结果，对照 `templates/doc-template.md` 的章节结构生成文档草稿
2. 缺失章节用 `⚠ 待补充` 占位
3. 展示草稿，主动提示潜在风险，例如：
   - Q5 跳过 → `⚠ 数据模型部分将较为简略`
   - Q4 跳过 → `⚠ 架构图中技术节点将使用通用占位符`
   - 核心功能少于 3 个 → `⚠ 用例图覆盖度可能不足`
4. 询问用户：`[补充信息] 返回修改` 或 `[继续] 接受当前状态`
5. 用户确认后，对文档进行润色：
   - 消除模糊表述（"尽快"、"适当"、"可能"、"一般"、"较好"）
   - 确保每条功能点有 `验收标准：` 字段
   - 确保数据模型字段标注类型
6. 生成时间戳：`YYYYMMDD-HHmm`，保存文档到：
   `output/{project-name}/docs/{timestamp}-requirements.md`
7. 更新 `output/.last-run.json`：
   ```json
   { "project": "{project-name}", "timestamp": "{timestamp}" }
   ```
8. 告知用户路径：
   ```
   ✔ 文档已生成：
     output/{project-name}/docs/{timestamp}-requirements.md
   ```

---

### 阶段 3 — 生成 PlantUML 图表

（仅当用户选择了 [2] 或 [all]）

0. 首先询问用户图表输出格式（可多选）：
   ```
   选择图表输出格式（可多选，空格分隔）：
   [1] PNG  — 位图，适合网页展示
   [2] SVG  — 矢量图，无损缩放，适合演示文稿
   [3] PDF  — 矢量 PDF，适合打印和文档嵌入
   [all] 全部格式
   ```
   用户选择后，记录为 formats 数组（如 `['png', 'svg', 'pdf']`），后续每种图按此数组循环生成。

1. 参考 `templates/plantuml-prompts.md`，基于文档内容生成以下四种图的 PlantUML 代码，每种图生成后按用户选择的格式逐一执行渲染命令：

> 渲染命令模板（以数据流图 dfd 为例，其他图替换图类型即可）：

```bash
node --input-type=module -e "
import { downloadDiagram } from './scripts/plantuml-render.js';
const uml = \`{生成的PlantUML代码}\`;
const formats = [{用户选择的格式数组}];
const base = 'output/{project-name}/diagrams/{timestamp}-dfd';
Promise.all(formats.map(fmt => downloadDiagram(uml, base + '.' + fmt, fmt)
  .then(p => console.log('✔ dfd (' + fmt + '):', p))
  .catch(e => console.error('✘ dfd (' + fmt + '):', e.message))
));
"
```

**数据流图（dfd）** — 生成代码后执行上述模板（替换图类型为 dfd）

**用例图（usecase）** — 生成代码后执行上述模板（替换图类型为 usecase）

**活动图（activity）** — 生成代码后执行上述模板（替换图类型为 activity）

**时序图（sequence）** — 生成代码后执行上述模板（替换图类型为 sequence）

> 以上命令在 `doc-forge/` 根目录下执行，输出目录由脚本自动创建。

2. 每张完成后告知路径（以选择 png + svg 为例）：
   ```
   ✔ 图表已生成：
     output/{project-name}/diagrams/{timestamp}-dfd.png
     output/{project-name}/diagrams/{timestamp}-dfd.svg
     output/{project-name}/diagrams/{timestamp}-usecase.png
     output/{project-name}/diagrams/{timestamp}-usecase.svg
     output/{project-name}/diagrams/{timestamp}-activity.png
     output/{project-name}/diagrams/{timestamp}-activity.svg
     output/{project-name}/diagrams/{timestamp}-sequence.png
     output/{project-name}/diagrams/{timestamp}-sequence.svg
   ```

---

### 阶段 4 — 生成宣传图 / 原型图

（仅当用户选择了 [3] 或 [all]）

1. 根据文档内容，生成以下三类图像的英文提示词：

   **Banner 图**（突出项目名称、核心价值、视觉风格）
   示例提示词结构：
   ```
   A clean tech-style banner for a developer tool called "{project-name}".
   Tagline: "{one-line description}". Dark background, modern typography,
   subtle code or diagram motifs. 16:9 ratio.
   ```

   **功能展示图**（以卡片形式呈现 2-3 个核心功能）
   示例提示词结构：
   ```
   A feature showcase illustration for "{project-name}". Show {feature1},
   {feature2}, and {feature3} as clean UI cards on a light background.
   Flat design style.
   ```

   **UI 原型图**（线框图风格，展示主界面布局）
   示例提示词结构：
   ```
   A wireframe mockup of the main interface for "{project-name}".
   Show the primary workflow: {core-flow}. Grayscale, minimal, annotated.
   ```

2. 展示提示词给用户预览，用户可直接确认或修改
3. 询问图像模型：
   ```
   选择图像生成模型：
   [1] gpt-image-1  — 最高质量，通常需 OpenAI Tier 1
   [2] dall-e-3     — 质量好（推荐）
   [3] dall-e-2     — 成本最低，分辨率较低
   ```
4. 调用 `scripts/image-generate.js` 的 `generateImage()` 生成图像
5. 保存到 `output/{project-name}/images/{timestamp}-{type}.png`
6. 每张完成后告知路径

---

### 完成汇总

全部产物生成完毕后输出：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✔ 所有产物已生成，保存于：
  output/{project-name}/

📄 文档
  docs/{timestamp}-requirements.md

🗂 图表
  diagrams/{timestamp}-dfd.png
  diagrams/{timestamp}-usecase.png
  diagrams/{timestamp}-activity.png
  diagrams/{timestamp}-sequence.png

🖼 图像
  images/{timestamp}-banner.png
  images/{timestamp}-features.png
  images/{timestamp}-wireframe.png

最新产物路径记录于：output/.last-run.json
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
