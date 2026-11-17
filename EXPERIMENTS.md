# LightText 实验统计

统计日期：2026-11-17。只使用原账本、候选表、队列和本次只读盘点，不新增评价或改写旧结论。

## 统计口径

| 对象 | 数量 | 说明 |
|---|---:|---|
| 原训练账本行 | 306 | 归档/状态记录，不保证每行是不同独立运行 |
| 归档不同ID | 286 | 按ID去重，20条为重复ID的额外记录 |
| 账本+当前队列的不同ID | 287 | 多出的DNER尚未规范归档 |
| 研究候选 | 408 | 包括草案、拒绝、CPU测试、挂起和完整实验，不等于训练次数 |

306行原账本状态：`done_complete`=40，`stopped_unknown`=67，`done_earlystop`=191，`running`=2，`crashed`=6。

按每个ID最后一行去重后的状态：`done_earlystop`=187，`stopped_unknown`=54，`crashed`=6，`done_complete`=39。

最后一行仅指CSV中的最后出现位置，不推定旧日期字段的准确性；旧账本2条running不代表现在有2个任务。

研究候选阶段：`blocked`=10，`rejected`=323，`complete`=73，`validation`=1，`training`=1。

## 实时与登记差异

- 原server3实测时间：`2026-11-17T11:01:07.705413+00:00`。两次原PID21708缺席、compute列表空、日志SHA一致。
- DNER日志写明 `Early stopping at epoch 23: no improvement for 10 epochs (best=0.4914)`。
- DNER队列仍为running、候选仍为training；本次只读交接没有执行规范归档，因此统计分列，禁止重复启动。
- 当前数据不支持声称DNER完整评价或冻结TRAIN已经执行，也不把软件契约PASS当成疗效。

## 基线与当前最好

| 模型 | 原完整逐表达导出mIoU | 参数 | 边界 |
|---|---:|---:|---|
| 纯四层 | 0.48951876743537803 | 96199817 | 原checkpoint、9508表达/141场景，复用导出 |
| 完整模型 | 0.4970706207473738 | 153493385 | 原完整模型对照 |
| TCAB | 0.5032357239155547 | 未在当前比较清单中实测 | 原UNet倍率0.1不同，不作模块因果归因 |

目标为0.5100，未达到；不能据参数更少或个别点估计较高宣称稳定Pareto占优。
官方修复/退化阈值为point IoU > 0.5；scene bootstrap不覆盖训练seed或VAL选择不确定性。

## 历史成绩前列

以下按不同ID的历史最高登记分数排序，**是描述性账本排名，不是新验证或可比性认证**。

| ID | 最高登记mIoU | 该ID最后账本状态 | 该ID记录数 | 配置 |
|---|---:|---|---:|---|
| tcab | 0.5032 | done_earlystop | 1 | `configs/tunet_tcab/model_base_early_exit_4_tunet_tcab.yaml` |
| sdca | 0.5030 | done_earlystop | 1 | `configs/tunet_sdca/model_base_early_exit_4_tunet_sdca.yaml` |
| tunet_scontr | 0.5028 | done_earlystop | 1 | `configs/tunet_scontr/model_base_early_exit_4_tunet_scontr.yaml` |
| trdg | 0.5010 | done_earlystop | 1 | `configs/tunet_trdg/model_base_early_exit_4_tunet_trdg.yaml` |
| tsg2 | 0.5008 | done_earlystop | 1 | `configs/tunet_tsg2/model_base_early_exit_4_tunet_tsg2.yaml` |
| tunet_scontr_pyramid | 0.5007 | stopped_unknown | 1 | `configs/tunet_scontr_pyramid/model_base_early_exit_4_tunet_scontr_pyramid.yaml` |
| tcad | 0.5000 | done_earlystop | 1 | `configs/tunet_tcad/model_base_early_exit_4_tunet_tcad.yaml` |
| trwm | 0.5000 | done_earlystop | 1 | `configs/tunet_trwm/model_base_early_exit_4_tunet_trwm.yaml` |
| model_base_early_exit_4_text_unet | 0.4993 | done_complete | 1 | `configs/model_base_early_exit_4_text_unet.yaml` |
| tunet_lcd | 0.4991 | done_earlystop | 1 | `configs/tunet_lcd/model_base_early_exit_4_tunet_lcd.yaml` |
| tunet_scontr_qitc | 0.4987 | stopped_unknown | 1 | `configs/tunet_scontr_qitc/model_base_early_exit_4_tunet_scontr_qitc.yaml` |
| dpeg4_ef | 0.4986 | done_earlystop | 1 | `configs/model_base_early_exit_4_dpeg.yaml` |
| trkv | 0.4984 | done_earlystop | 1 | `configs/tunet_trkv/model_base_early_exit_4_tunet_trkv.yaml` |
| trsp | 0.4984 | done_earlystop | 1 | `configs/tunet_trsp/model_base_early_exit_4_tunet_trsp.yaml` |
| dsei-depth-selected-mask-iou-risk | 0.4984 | done_earlystop | 2 | `configs/model_base_early_exit_4_dsei.yaml` |
| dupm-depth-unmatched-physical-mask | 0.4982 | done_earlystop | 2 | `configs/model_base_early_exit_4_dupm.yaml` |
| pyr3 | 0.4980 | crashed | 1 | `configs/tunet_pyr3/model_base_early_exit_4_tunet_pyr3.yaml` |
| tcls | 0.4980 | done_earlystop | 1 | `configs/tunet_tcls/model_base_early_exit_4_tunet_tcls.yaml` |
| tunet_itc_hn | 0.4979 | stopped_unknown | 1 | `configs/tunet_itc_hn/model_base_early_exit_4_tunet_itc_hn.yaml` |
| tunet_scontr_w05 | 0.4979 | stopped_unknown | 1 | `configs/tunet_scontr_w05/model_base_early_exit_4_tunet_scontr_w05.yaml` |

## 近期候选对回读

| ID | 结构/诊断 | 登记best | 当前证据与结论 |
|---|---|---:|---|
| doam4_ef | 有序关联记忆 | 0.4961 | 完整导出abs1e-6 PASS；固定32主判据不支持；原controller退出失败，validation保留 |
| dlor4_ef | 隐空间有序旋转 | 0.4912 | 完整导出PASS；冻结TRAIN4/20/40完成，not_supported_on_fixed32 |
| dcph4_ef | 连续短语区间 | 0.4922 | 23轮终态/一次完整导出；原abs1e-6 FAILED，冻结TRAIN未准入 |
| dcsr4_ef | 覆盖分段回读 | 0.4853 | 完整导出PASS；冻结TRAIN4/20/40完成，not_supported_on_fixed32 |
| dbcr4_ef | 二叉子组合 | 0.4956 | 25轮终态/一次完整导出；原abs1e-6 FAILED，冻结TRAIN未执行 |
| dwvr4_ef | 词条件视觉value回读 | 0.4931 | 一次完整导出原abs1e-6 FAILED，原因未知 |
| dptm4_ef | 候选对token memory | 0.4951 | 一次完整导出原abs1e-6 FAILED，原因未知 |
| dsar4_ef | state-addressed回读 | 0.4876 | 一次完整导出原abs1e-6 FAILED；保存向量的dtype复核不能解决旧差异 |
| dner4_ef | 非回溯边组合回读 | 0.4914（早停日志四位数） | 原23轮自然终态只读已观察；待规范归档/一次完整评价；原TRAIN诊断未执行 |

这些是同一研究主线下的不同结构假设，不把通用attention、非可分性或新增计算图本身认证为创新。
不支持的固定TRAIN局部判据不外推为所有深度轻量化路线无效。

## 挂起项

| 稳定ID | 挂起类型 | 失败范围 | 下一步 |
|---|---|---|---|
| dvri-depth-visual-reentrant-integration | human_revision_authorization | inconclusive | Keep the same DVRI ID and all R0/R1/R2 evidence. Do not run an automatic R3, unchanged ABBA or source/cohort/seed/stage scan. Require explicit human hash-addressed authorization against R2 implementation aad6152f5dc5d65315a90d3692712533095b7fca770de3a4a472c63275945cc5 for any further model revision, or genuinely independent resolving evidence for a new scientific review. Timer alone is not permission; no admission until the unresolved task-source gate is honestly resolved. |
| droc-depth-relation-order-composition | independent_grounded_counterfactual_annotations |  | Resume SAME ID with genuinely new independent original12 camera/export deployment and physical original/swapped certification. Do not substitute four favorable controls or repeat unchanged PhraseRefer/ViGiL/span/source/geometry/consumer audits on retry alone. Independent task-evidenced original depth mechanisms remain permitted. |
| dspl-depth-mask-path-likelihood | human_revision_authorization | inconclusive | Retain SAME ID and all original/R1/R2 evidence. No automatic third patch, re-timing, queue entry or training. If explicitly authorized, preregister the concrete resource defect and discriminating test before any further implementation. |
| dser-depth-self-excluding-relay | human_revision_authorization | implementation | Preserve SAME DSER ID and all R0/R1/R2 evidence. Default two implementation revisions are used; no automatic R3 or unchanged ABBA. Further model revision requires explicit human hash-addressed authorization against R2 implementation b0c187ba67f0226fe42d9d05f057eb6da67032c6c90f4c540bd26c1d52585646, or genuinely independent resolving evidence for review. Do not weaken Pareto, alter compiler globals, select timings/cohorts/seeds or call a gradient-shape failure population harm. |
| dcss-depth-cardinality-set-semantics | independent_grounded_counterfactual_annotations | implementation | Resume SAME DCSS ID when independent frozen-cohort bindings arrive. Review all original rows without IoU selection, then preregister a count-specific matched test before any algebraic/model repair. Do not repeat unchanged captures, certify labels from language alone, weaken duplicate controls, reopen closed routes or infer R3/training consent. |
| dsar-depth-state-addressed-reread | numerical_control_failure | inconclusive | Genuinely new independent evidence resolving exact native/export numerical discrepancy against original checkpoint/source/data/log and abs1e-6. Preserve original failed result;do not replay unchanged export,relax tolerance,choose outcomes,alter model/settings or run frozen TRAIN audit without its original prerequisites. New98cc saved9508 original NumPyfloat32 mean=.4875599145889282 still differs from native by6.0498714447021484e-6;aggregation dtype alone does not explain this saved vector.Do not repeat that audit;native original per-expression vector remains unavailable. |
| dptm-depth-pair-token-memory | numerical_control_failure | inconclusive | Genuinely new independent evidence resolving exact original DPTM native/export discrepancy against original94f checkpoint/source/data/log and unchangedabs1e-6.Preserve the once-complete export and failed agreement;no unchanged re-export,dtype/seed/cohort/formula/loss selection,tolerance relaxation,model revision or frozen TRAIN audit without actual original successful prerequisites. |
| dwvr-depth-word-conditioned-visual-reread | numerical_control_failure | inconclusive | Retain once-complete DWVR9508/141 export/paired results and originalabs1e-6 FAILED,causeunknown.Only new independent resolving evidence;do not re-export,relax,run frozen TRAIN without genuine PASS prerequisite or declare efficacy.No global hold on independent design. |
| dcph-depth-contiguous-phrase-reread | numerical_control_failure | inconclusive | Preserve once-complete DCPH23 terminal/archive/full9508/141 three-comparator five-group export and originalabs1e-6 FAILED,causeunknown.User full-only reply already executed;do not re-ask,extend to TRAIN,re-export,relax or retrain.Only new independent resolving evidence;frozen TRAIN needs original successful prerequisite.Do not globally block independent work. |
| dbcr-depth-binary-composition-reread | numerical_control_failure | inconclusive | Preserve original25-epoch terminal/archive, one full9508/141 export and original FAILED agreement. No frozen TRAIN prerequisite, re-export, retrain or model/setting/threshold selection. Independent work continues. |

## 完整清单

- `experiments_raw.csv`：保留306行原账本，不清洗覆盖原日期/空值/旧状态。
- `experiments_unique.csv`：每ID配置、最新账本分数/状态、历史最高登记、记录数、候选关联、下一步。
- `candidates_all.csv`：全部408个稳定ID的假设、对照、阶段、失败范围、证据数量和下一步。
- `experiment_summary.json`：计数规则、来源文件SHA256和唯一的队列待对账项。
- `pipeline/experiments/`、`pipeline/analyses/`、`pipeline/research/`：更细来源；大原始输出按排除清单单独传输。

旧终态、失败或拒绝路线不因迁移而重开。新设计仍限固定四层文本候选对条件描述回读；不做超参/seed扫描、token裁剪、蒸馏、六层early-exit或Phase2训练消融。
