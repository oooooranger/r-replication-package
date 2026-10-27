# 简明R写法

仅作为代码形状参考；示例变量、固定效应和聚类层级不构成实际研究设定。每次按目标研究替换。不要把历史论文的变量或时间窗口写入通用技能。

## 一个表对应一个容易阅读的脚本

以下完整示例要求`analysis.csv`包含`y, x, control_a, control_b, unit_id, period`。它演示两个面板OLS规格；不自动适用于截面、计数模型或其他研究设计。文件放在章节目录，入口通过`source(..., chdir=TRUE)`执行。

```r
library(data.table)
library(fixest)
library(modelsummary)

d <- fread("analysis.csv", encoding = "UTF-8")

# 本例数据键：观察单位×时期。实际设计允许重复键时应改用完整记录键。
stopifnot(!anyDuplicated(d[, .(unit_id, period)]))

# 表1第(1)列：Y=y，X=x，无额外控制；单位+时期FE，按单位聚类。
# 不预先删除其他列的缺失值；本列有效样本由其公式决定。
m1 <- feols(y ~ x | unit_id + period,
            data = d, vcov = ~unit_id)

# 表1第(2)列：增加control_a、control_b；其他设定相同。
m2 <- feols(y ~ x + control_a + control_b | unit_id + period,
            data = d, vcov = ~unit_id)

# 输出在复现包之外，数据文件保持不变。
out <- file.path("..", "..", "复现结果", "01_主回归和稳健性检验")
dir.create(out, recursive = TRUE, showWarnings = FALSE)

tab <- modelsummary(
  list("(1) 基准规格" = m1, "(2) 加入控制" = m2),
  coef_map = c("x" = "核心解释变量",
               "control_a" = "控制变量A", "control_b" = "控制变量B"),
  estimate = "{estimate}{stars}",
  statistic = "({std.error})",
  stars = c("*" = 0.10, "**" = 0.05, "***" = 0.01),
  fmt = 4,
  gof_map = data.frame(
    raw = c("nobs", "r.squared", "r2.within"),
    clean = c("观测数", "总体R²", "组内R²"),
    fmt = c(0, 4, 4)
  ),
  notes = "两列均含单位与时期固定效应；括号内为单位聚类标准误。",
  output = "tinytable"
)
tab <- tinytable::theme_html(tab, portable = TRUE)
tinytable::save_tt(tab, file.path(out, "表1_主回归.html"), overwrite = TRUE)
```

示例采用`fixest`默认小样本修正；若既定模型有特定修正，显式传入经核对的`ssc()`。不要把某一项目曾使用的`K.fixef="full"`等参数固定成所有论文的要求。

`coef_map`可以决定呈现顺序和省略项，但不能伪造控制变量或固定效应行。`gof_map`的提取名随模型/包版本核对；用`get_gof()`、`get_estimates()`与模型输出检查，不能见空值就随手填原表数字。对特殊模型可通过`add_rows`加入本次计算的统计量，清楚注明口径。

## 根入口示例

安装依赖的命令放注释，正式回归不自动升级包。入口不写本机用户名、盘符或私有路径。命令行支持从任意工作目录启动；交互方式先把工作目录设到包根目录。

```r
# 初次使用：install.packages(c("data.table", "fixest", "modelsummary", "tinytable"))
# RStudio：在包根目录运行 source("00_运行全部.R", encoding = "UTF-8")。
# 命令行：Rscript --vanilla /路径/审稿人复现包/00_运行全部.R
# Windows非UTF-8区域设置可能影响中文路径；仅调整字符编码。
if (.Platform$OS.type == "windows" && !l10n_info()[["UTF-8"]]) {
  invisible(Sys.setlocale("LC_CTYPE", "English_United States.utf8"))
}
script_arg <- grep("^--file=", commandArgs(), value = TRUE)
if (length(script_arg) == 1) {
  setwd(dirname(normalizePath(sub("^--file=", "", script_arg))))
}

source("01_主回归和稳健性检验/01_主回归.R", chdir = TRUE, encoding = "UTF-8")
# 依据实际交付继续显式source各章节，不创建不存在的分析。
capture.output(sessionInfo(), file = "../复现结果/R运行环境.txt")
```

## 数据准备保持简短而可追溯

- 原始清理复杂时，分开“准备分析数据”和“按表估计”；不要把多年抓取、解析和模糊匹配塞进每个回归脚本。
- 分析数据保留审稿人需要验证的分子、分母、基期、处理时间等构成变量，或给出精确来源与上游构造代码。是否交付原始库取决于本次范围和已有使用限制。
- 预先计算的滞后变量写清楚日历时间。非平衡面板中按行`shift()`不必然等于滞后一年；构造前处理年份缺口。取对数、比例和缩尾都要说明对象、单位、零值及样本边界。
- 面板主键、事件记录键、堆叠DID的个体×时间×批次键可能不同，不用城市×年份键约束所有数据。
- 不为简化代码丢掉无法在包内重建的关键构造；同样不要为了“完整”复制无关个人或商业记录。

## 按需查官方参考

常规写法无需每次重新搜索；遇到函数版本或推断口径不明确时查当前文档和本机帮助。

- [stata2R：fixest](https://stata2r.github.io/fixest.html)
- [fixest官方参考](https://lrberge.github.io/fixest/reference/index.html)
- [modelsummary官方说明](https://modelsummary.com/vignettes/modelsummary.html)
- [tinytable官方参考](https://vincentarelbundock.github.io/tinytable/)
