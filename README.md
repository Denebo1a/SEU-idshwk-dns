# 说明文档
这个仓库整理了我们实验所需要的材料，对老师发布的材料做了目录上的整理，删除了文件夹中无意义的嵌套和MacOS产生的.DS_Store。

由于老师在作业要求中提到vibe coding和agent的使用占20%的分值，这个仓库引入了Claude Code进行了一些前期规划，内容整理在CLAUDE.md当中。

大作业实验分为三个任务，我们自己的工作目录位于 `dns-homework`下。
参考资料位于 `dns-homework-reference/`，包含学长的实验源码、文档与结果，可用于对照思路，但应以本组目录和当前题目要求为准。

## 这个仓库的构成
- dns-homework：我们小组自己的实验工作目录
  - src: 我们的实验源码目录
  - docs：存放三个实验题目的描述文档
  - answers：我们的实验结果目录
- dns-homework-reference：github上学长的实验源码/文档/结果参考
  - src: 学长的实验源码目录
  - docs：存放学长的实验文档
  - answers：学长的实验结果目录

## 补充说明
- `CLAUDE.md` 记录了当前对题目、提交规范、参考实现和实现计划的整理结果。
- `dns-homework/docs/` 是当前实验要求的第一信息源。
- `dns-homework-reference/` 是参考资料目录，适合借鉴思路，不适合直接照搬为最终提交内容。

## 更完整的目录说明
- `dns-homework`
  - `src/`：我们自己的实验源码目录
  - `docs/`：三个实验题目的说明文档与提交说明
  - `answers/`：我们最终要提交的结果文件目录
  - `questions/`：实验数据集目录，通常体积较大；非必要不要全量扫描或随意改动
- `dns-homework-reference`
  - `src/`：参考 notebook / 代码
  - `docs/`：参考实验报告
  - `answers/`：参考结果文件

## 三道实验题概览

### DNS1：恶意域名家族发现与分类
- 目标：从全部域名中找出未标注的恶意域名，并进一步预测其恶意家族编号 `family_no`。
- 特点：已知标签只覆盖约一半恶意域名，因此通常需要先判断“是否恶意”，再细分“属于哪个家族”。
- 关键数据：`fqdn.csv`、`ip.csv`、`ipv6.csv`、`access.csv`、`flint.csv`、`label.csv`、`whois.json`
- 输出文件：`answers/dns1.csv`
- 表头要求：`fqdn_no,family_no`

### DNS2：黑白域名二分类
- 目标：根据流量与解析特征，对域名进行黑/白二分类。
- 特点：这是标准监督学习任务，训练集带标签、测试集无标签。
- 关键数据：训练集/测试集中的 `fqdn.csv`、`ip.csv`、`ipv6.csv`、`access.csv`、`flint.csv`、`label.csv`
- 输出文件：`answers/dns2.csv`
- 表头要求：`fqdn_no,label`

### DNS3：域名聚类
- 目标：根据请求特征对域名进行聚类，簇数不能小于 3。
- 特点：这是无监督任务，输出的是聚类簇编号。
- 关键数据：`fqdn.csv`、`ip.csv`、`ipv6.csv`、`access.csv`、`flint.csv`
- 输出文件：`answers/dns3.csv`
- 表头要求：`fqdn_no,label`

## 协作成员使用建议
- 新成员建议先阅读：
  1. `README.md`
  2. `CLAUDE.md`
  3. `dns-homework/docs/` 下的题目说明与提交说明
  4. `dns-homework-reference/docs/` 与 `dns-homework-reference/src/` 中的参考材料
- 开始开发前，先确认当前正在处理哪一题，以及对应数据是否已经准备到本地。
- 写代码时优先在 `dns-homework/src/` 中实现，不要直接修改参考目录中的内容。
- 讨论结果时，以 `answers/` 中的最终 CSV 为准。

## 参考实现能提供什么帮助
根据目前对 `dns-homework-reference/` 的整理，参考实现主要采用“手工特征工程 + 传统机器学习”的路线：
- DNS1：先识别恶意域名，再做恶意家族分类
- DNS2：直接做监督式二分类
- DNS3：归一化后使用 KMeans 聚类

其中比较值得借鉴的是：
- 统一复用 `fqdn / access / flint / ip / whois` 特征工程框架
- 将按小时、按日期的访问/解析分布作为重要特征
- 使用去重计数特征，如访问国家数、城市数、ISP 数、nameserver 数
- DNS1 的“两阶段建模”思路与题目目标高度一致

但也要注意：
- 参考实现里有一些写死的常量，迁移时要改成动态推断
- 参考实现更适合理解思路，不适合直接原样提交

## 提交要求速览
- 最终评分程序只读取 `answers/` 下三个文件：
  - `answers/dns1.csv`
  - `answers/dns2.csv`
  - `answers/dns3.csv`
- 所有 CSV 文件都应满足：
  - UTF-8 编码
  - 英文逗号分隔
  - 第一行为表头
  - 每个文件仅两列
- 额外注意：
  - `fqdn_no` 必须使用题目原始编号
  - 第二列只能填写数字编号，不要写类别名或概率
  - DNS1 第二列名必须是 `family_no`
  - DNS2 和 DNS3 第二列名必须是 `label`

## 实现方向速览
如果后续继续按照 `CLAUDE.md` 的规划推进，实现方向可以概括为：
- DNS1：统一特征工程 → 恶意检测 → 家族多分类 → 生成 `dns1.csv`
- DNS2：统一特征工程 → 二分类训练与验证 → 生成 `dns2.csv`
- DNS3：统一特征工程 → 归一化/聚类实验 → 生成 `dns3.csv`
