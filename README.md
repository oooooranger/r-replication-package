# 论文结构化复现 · R Replication Package

将已有实证研究整理为审稿人能阅读、逐步运行并核对的复现资料。

A reusable skill and plugin for preparing readable, independently runnable research replication packages. The default workflow uses **data.table → fixest → modelsummary**, while preserving the study's actual estimators and inference settings.

## 能做什么

- 按文章实际章节组织数据、代码、表格和图。
- 逐个明确写出回归变量、样本、控制变量、固定效应和推断方法。
- 从本次估计对象导出表格，核对系数、标准误、样本量和适用的拟合统计。
- 区分从分析数据复现与从原始数据开始的完整复现。
- 换目录、在新R进程中运行，检查外部路径依赖及输入文件是否被改写。
- 把实算与正文的差异单独列出，保留不显著结果。

适用于不同主题、年份和数据层级。目录、变量、滞后、固定效应、聚类和识别策略均由本次研究确定。它整理和复现已有研究，不替用户选择能得到显著结果的规格。

## 使用

安装技能后主动调用：

```text
$r-replication-package

根据当前论文和已有数据，整理给审稿人使用的复现资料。
按文章各部分组织，只保留必要数据和清晰代码，运行后导出表格，
核对正文结果，并验证换一个目录后能够独立运行。
输出到我指定的文件夹。
```

也可以只请求某一部分，或指定自己的目录名称、表格格式与估计方法。技能的`allow_implicit_invocation`设置为`false`，按用户明确调用启用。

## 安装

### 作为技能安装

在提供`skill-installer`的Codex环境中，可以请求：

```text
$skill-installer
安装 https://github.com/oooooranger/r-replication-package/tree/main/skills/r-replication-package
```

也可下载仓库，将`skills/r-replication-package`整个文件夹放入当前环境支持的个人或项目技能目录。各版本的发现路径以[官方技能说明](https://learn.chatgpt.com/docs/build-skills)为准；保留`SKILL.md`、`agents`和`references`的相对位置。

### 作为插件使用

仓库根目录包含可移植`plugin.json`，以及供兼容环境使用的`.codex-plugin/plugin.json`。从[Releases](https://github.com/oooooranger/r-replication-package/releases)下载插件ZIP，按所用客户端的自定义插件或本地插件导入流程使用。GitHub仓库发布与官方插件目录审核是不同流程。

插件仅包含技能与参考说明，无MCP服务器、账户连接或生命周期脚本。实际执行回归时需要可访问的R环境；常规示例用到`data.table`、`fixest`、`modelsummary`和`tinytable`。

## 输出结构

以下是可调整的起点，实际章节优先：

```text
交付目录/
  审稿人复现包/
    00_运行全部.R
    01_主回归和稳健性检验/
    02_内生性与识别检验/
    03_机制分析/
    04_进一步分析/
  复现结果/
  使用与核对说明.md
```

章节子文件夹默认只放数据、R代码和数据字典；生成的结果放在包外。未请求或原文没有的分析不凭空补齐。

## 仓库内容

```text
plugin.json
.codex-plugin/plugin.json
skills/r-replication-package/
  SKILL.md
  agents/openai.yaml
  references/r-patterns.md
  references/estimator-checks.md
```

## 验证

v1.0.0已通过`skill-creator`结构校验。参考R代码以240行模拟面板实际执行，在原目录和独立目录中运行两次；模型按有效变量分别保留233和219条观测，导出表格单元格一致，输入文件未变。

该验证覆盖技能结构和参考代码的可运行性；每篇实际研究的数据、模型和表格仍需按任务核验。此仓库提供通用工作流与示例，不附带具体论文、商业数据或论文回归结果。

## 参考

- [OpenAI：插件打包](https://developers.openai.com/plugins/build/plugins)
- [stata2R：fixest](https://stata2r.github.io/fixest.html)
- [fixest官方参考](https://lrberge.github.io/fixest/reference/index.html)
- [modelsummary](https://modelsummary.com/vignettes/modelsummary.html)
- [tinytable](https://vincentarelbundock.github.io/tinytable/)
