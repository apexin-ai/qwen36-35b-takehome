# 模型训练研究员笔试提交指南

以 Qwen/Qwen3.6-35B-A3B 完成自建数据、LoRA/QLoRA SFT、SFT 后第二阶段优化与四项代码基准评测。具体实验要求和评测口径以面试方发放的笔试题目为准。

## 仓库使用方式

本仓库为私有模板仓库。面试方从模板为每位候选人创建独立私有仓库，并单独邀请对应 GitHub 账号。候选人接受邀请后，只在分配给自己的仓库提交，不向模板仓库提交答案。模板设置不会自动创建候选人仓库或自动发送邀请。

在个人候选人仓库的 main 分支提交，或按面试通知提交 Pull Request。最终提供仓库或 PR 链接、最终 commit SHA 和两个 Hugging Face 模型链接及 revision。

## 推荐目录结构

以下是提交结构示例，不代表模板已包含这些实现文件。文件名可调整，请在 README 中标明实际路径。

```text
./
├── README.md                 # 实验说明、复现命令、结果和模型链接
├── report.pdf                # 一页以内分析报告
├── requirements.lock         # 或其他依赖锁定文件
├── .gitignore                # 排除密钥、缓存、权重等；保留要求的日志
├── configs/
│   ├── sft.yaml              # SFT 完整配置
│   ├── stage2.yaml           # 第二阶段训练配置
│   └── eval/                 # 四项 benchmark 的固定评测配置
├── data/
│   ├── README.md             # 来源、许可、规模、划分、清洗、去重
│   └── manifests/            # 数据版本、清单和校验值
├── src/                      # 数据处理、训练及评测实现
├── scripts/
│   ├── prepare_data.sh       # 数据获取或构建命令
│   ├── train_sft.sh
│   ├── train_stage2.sh
│   ├── evaluate.sh           # 覆盖三组模型和四项 benchmark
│   ├── summarize.py          # 汇总原始结果和统计置信区间
│   └── plot_curves.py         # 从源数据重建曲线
├── logs/
│   └── <run_id>/
│       ├── metadata.json     # 版本、硬件、配置、命令、时间、种子
│       ├── train.log         # 训练日志（训练 run）
│       ├── eval.log          # 评测日志（评测 run）
│       ├── metrics.jsonl     # 逐 step 原始指标
│       ├── errors.log        # 错误、重试、超时；无错误则说明
│       └── resources.csv     # 显存、耗时、GPU 小时和成本
├── curves/
│   ├── sft_loss.png          # 训练和独立验证集 loss
│   ├── stage2_metrics.png    # 第二阶段 loss 及方法相关指标
│   └── source/              # 对应 CSV/JSON 等原始曲线数据
├── results/
│   ├── base/<benchmark>/<run_id>/
│   ├── sft/<benchmark>/<run_id>/
│   ├── stage2/<benchmark>/<run_id>/
│   └── summary.csv          # 三组模型 × 四项 benchmark 汇总
├── analysis/
│   ├── failures.md          # 至少 20 个失败案例与归因
│   └── uncertainty.json     # 置信区间和重复运行波动
└── models/
    └── README.md            # HF 链接、revision、加载和复现方法
```

## 实验与复现说明

README 应补充研究问题、实验矩阵、环境安装、数据构建、SFT、第二阶段优化、模型加载、四项评测、结果汇总和绘图的完整可执行命令。注明命令执行目录、输入输出路径、资源需求和预计耗时。

记录基座 revision、tokenizer、chat template、代码 commit、依赖版本、数据版本、LoRA 目标模块和专家层处理、rank、量化、学习率、batch size、训练步数、上下文长度、随机种子和 checkpoint 选择方式。训练数据与独立验证集应分离；benchmark 答案、参考补丁、隐藏测试和测试反馈不得用于训练或挑选 checkpoint。

Base 指本次微调前的原始发布权重。SFT 本身属于 post-training；本题第二阶段优化指在 SFT 权重上继续训练，不能仅以推理时拒绝采样替代。允许自行租用算力，必须提交完整实验，不接受缩小模型或 dry run 替代。

## Loss 曲线与日志

- 分别提交 SFT 和第二阶段训练的训练 loss、独立验证集 loss 曲线；第二阶段同时报告适用于所选方法的指标，例如 DPO chosen/rejected reward、reward margin 或偏好准确率，并解释含义。不同目标函数的 loss 不应直接比较数值高低。
- 曲线标注 step 或处理 token 数、纵轴含义、run_id 和关键 checkpoint。保留原始数据；如采用平滑，披露窗口并保留未平滑数据。提交 PNG/PDF 图、CSV/JSON 源数据和绘图脚本，不能只交截图或外部仪表盘链接。
- 日志覆盖完整训练与评测过程，包括错误、重试、超时和恢复情况。每个 run_id 应能对应代码版本、配置、数据版本、模型 checkpoint 和结果文件。
- 提交训练/评测耗时、峰值显存、GPU 型号与数量、GPU 小时、费用及计算方式。日志可压缩；在 README 中提供位置、格式及读取方法。超出 GitHub 文件限制时，在个人仓库 Release 中附压缩日志，并在仓库提交索引、校验值和访问说明。

## 评测结果

四项 benchmark 为 SWE-bench Verified、SWE-bench Multilingual、Terminal-Bench 2.0、LiveCodeBench v6。按题目要求固定版本、任务清单、框架、容器、prompt、工具、上下文、推理预算和采样设置，三组模型使用相同口径。公开设置与官方内部框架的差异应如实披露，不将官方分数直接作为本次 Base。

每组结果应包含逐任务输出或补丁（许可允许范围内）、判定结果、运行状态、错误原因、耗时及 token 使用量。记录全部任务，不能删除失败任务后计算分数。Terminal-Bench 2.0 按题目要求提交五次运行结果、均值与标准差。

请将下表填写为实际实验结果，不能用占位符作为最终提交。分数统一为百分制，并另行报告任务数、运行失败率、成本、配对 bootstrap 置信区间及重复运行波动。

| Benchmark | Base | LoRA SFT | 第二阶段优化 | 最终相对 Base 变化（百分点） |
| --- | --- | --- | --- | --- |
| SWE-bench Verified | 待填写 | 待填写 | 待填写 | 待填写 |
| SWE-bench Multilingual | 待填写 | 待填写 | 待填写 | 待填写 |
| Terminal-Bench 2.0 | 待填写 | 待填写 | 待填写 | 待填写 |
| LiveCodeBench v6 | 待填写 | 待填写 | 待填写 | 待填写 |
| 四项等权宏平均 | 待填写 | 待填写 | 待填写 | 待填写 |

宏平均提升目标为 3 个百分点，5 个百分点作为优秀参考；如未达到仍须如实报告并分析，不得更改口径制造提升。至少分析 20 个失败案例，说明单项退化、局限和下一步实验。

## 模型权重与一页报告

LoRA SFT 与第二阶段优化的两个 checkpoint 上传 Hugging Face，可提交完整 adapter 及配置或合并权重。记录 HF URL、revision、基座 revision、tokenizer/chat template、量化信息、加载命令和所对应的 run_id。验证评审者可以访问下载并复现；GitHub 只保存链接和加载说明，不存放模型权重。

report.pdf 不超过一页，结构为摘要（150–200 个中文字符）、过程、结果。结果包含三组模型在四项 benchmark 上的分数、宏平均提升、运行失败率和主要成本。详细日志、曲线、置信区间和失败分析放在仓库其他目录，不计入报告的一页限制。

## 提交检查清单

- [ ] 提交到面试方分配的独立私有仓库，附最终 commit SHA 或 PR 链接
- [ ] README 包含可执行复现命令、版本、路径、实验矩阵与已知限制
- [ ] 数据构造代码、来源许可、清单、去重和泄漏检查可核验
- [ ] SFT 和第二阶段训练代码、配置及完整日志已提交
- [ ] loss/方法相关指标曲线、原始数据和绘图脚本齐全
- [ ] Base/SFT/第二阶段优化的四项完整评测、逐任务结果和失败记录齐全
- [ ] 汇总脚本、置信区间、至少 20 个失败案例及资源成本记录齐全
- [ ] 两个 Hugging Face checkpoint 链接、revision 和加载方法可用
- [ ] report.pdf 不超过一页，摘要、过程、结果完整
- [ ] 提交前检查日志与文件，移除密钥、token、个人数据及无权分发内容

## 面试方使用模板

通过 Use this template 创建候选人专属仓库，例如 qwen36-35b-takehome-candidate-001，选择 Private，逐仓库邀请对应候选人并给予 Write 权限。候选人无需加入组织或访问模板。检查组织默认权限和团队授权，确保其他候选人不能访问该仓库。已从模板创建的仓库不会自动同步后续模板更新。
