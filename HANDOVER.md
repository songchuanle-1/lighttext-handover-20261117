# LightText 项目交接与迁移

交接日期：2026-11-17，时区 Asia/Shanghai。本文对应独立迁移快照，不修改原训练仓库和远端进程。
最新只读盘点约为北京时间 19:01；账本统计时间、原始证据 SHA256 见同目录 JSON。

## 1. 先看结论

- 任务：ScanRefer 3D 指代分割，主指标 mIoU，Phase 1 目标 `>=0.5100`，尚未达到。
- 历史最佳登记为 TCAB，账本 `0.5032`；完整逐表达导出 `0.5032357239155547`。它不是新突破，也不能由一次成绩证明因果贡献或 Pareto。
- 归档 CSV 有 **306 条记录、286 个不同实验 ID**；DNER 另有一个未归档队列 ID，合并已知 ID 为 **287**。
- 研究工作表有 **408 条候选记录**，其中 complete 73、rejected 323、blocked 10、validation 1、training 1。这不是完成训练数量。
- DNER 原 PID21708 已缺席，GPU compute 列表为空，原日志显示第23轮 patience10 自然早停，best 四位数 `0.4914`。两次只读观测日志 SHA 一致，**本次没有执行规范归档**；原队列仍登记 running，必须先对账，不可重复启动。
- 本次没有执行新的训练、完整评价、冻结 TRAIN 诊断或旧实验重跑；现有科学结论没有因打包变成 PASS。

## 2. 阅读顺序

1. `EXPERIMENTS.md`：统计口径、历史成绩、近期机制状态。
2. `experiments_unique.csv`：286 个归档 ID 的配置、成绩、状态、候选关联和下一步。
3. `experiments_raw.csv`：原306行账本；`candidates_all.csv`：全部408条候选。
4. `MIGRATION_CHECKLIST.md`：搬迁步骤及验收。
5. 项目 `pipeline/research_direction.md`、`pipeline/codex_memory.md`、`pipeline/goal.json`、`SERVERS.yaml`。

注意：原 memory 的最后健康增长时间早于本次只读终态盘点。最新 DNER 观察以本交接目录
`remote_inventory.json`、`dner_terminal_readonly.json` 为准；这不是改写旧证据。

## 3. 仓库内容与完整性

GitHub 快照保持外层项目结构：

```text
lighttext_project/
  lighttext/                 原生模型、配置、训练入口、CUDA/C++ 源码
  framework/                 原外层补充源码，照原状保留
  configs/                   原外层补充配置，照原状保留
  scripts/                   准入、检查、分析、远端管理源码
  pipeline/                  原始账本、候选、授权、收据、研究源码及小型证据
  SERVERS.yaml               旧服务器配置，只作交接参考
  handover/                  本文、统计、环境盘点、校验和控制端工具快照
```

本地原代码提交为 `9c81f83ab9b3e812c5b6be214060c2376d4df0e4`。
原有未跟踪模型/配置和用户删除 `=1.4.0` 均未在原仓库中提交或还原。
快照保存当前存在的源码，但**包含源码不表示该候选已通过准入、有效或允许重开**。

`source_manifest.json` 保存逐文件 SHA256、来源 HEAD、原 dirty 状态及排除清单。
新 GitHub 快照的提交与原训练提交不是同一个 SHA。为保持旧实验可追溯，本地另存
`source-history.bundle`，保留原 Git commit 图；它不属于 GitHub 源码快照，需要单独传输。
不能把新快照 SHA 伪写成旧训练的 code_commit。

数据集、checkpoint、二进制扩展、环境目录、缓存、密钥、完整大日志和大型研究输出不进普通 Git。
所有被排除的条目有清单，原文件未删除。要延续旧哈希依赖的研究，必须另外复制其原证据，
不能因为文件缺失而改 gate、补假结果或重跑已关闭实验。

## 4. 实际环境，不照抄旧 README

2026-11-17 在原 server3 的实际解释器观察：

| 项目 | 观察结果 |
|---|---|
| 服务器 | `songcl@192.168.1.120`，hostname `sysu311-MS-7C59` |
| Python | `/home/songcl/.conda/envs/project_lighttext/bin/python`，3.8.20 |
| PyTorch | 1.12.1+cu113 |
| torchvision / torchaudio | 0.13.1+cu113 / 0.12.1+cu113 |
| spconv | spconv-cu113 2.3.6 |
| transformers | 4.31.0 |
| gorilla-core | 0.2.7.8 |
| NumPy / SciPy / numba | 1.24.4 / 1.10.1 / 0.58.1 |
| GPU / driver | RTX3090 24GB ×2 / 470.182.03 |
| nvcc | 此次解释器环境的 PATH 中未找到；不等于机器上没有 Toolkit |

完整包版本见 `requirements-server3-observed.txt`。这是观察锁表，不保证新硬件可以直接安装：
旧 torch/cu113 与新 GPU 是否兼容需要在目标机单独验收。不要直接拿旧 README 的
Python3.10/torch2.8/cu128 或当前 requirements 中 spconv-cu126 当作原训练环境。
控制端 Python 与训练 Python 分开：pipeline 使用 `Path.is_relative_to` 等接口，不能照搬到训练端 Python3.8 执行。

CUDA 扩展源码在 `lighttext/pointnet2/` 和 `lighttext/framework/lib/`。
在新环境重编译，不复制旧 `.so` 冒充兼容；也不要把重编译通过当成科学数值一致。

## 5. 数据与权重在哪里

原数据根：`/sdc1/songcl/IPDN/data/`，只读 `du` 约 **37.33 GiB**。
包含大小写不同的 `ScanRefer/scanrefer`、`Multi3DRefer/multi3drefer`，以及 `scannetv2`、
`roberta-base`、`clip_feat`、`features`、`lookup.json`、标签 TSV、
`sp_unet_backbone.pth`、`free_embed.pth`、`dependency_map.pth`。
保持大小写和目录结构；仅复制 GitHub 不能训练。

原 checkpoint 根：`/sdc1/songcl/Project/LightText_project/LightText_checkpoints/`，
当前总占用约 **253.51 GiB**，包含大量历史运行，不要求每次完整搬走。
最少要保留将继续研究的原 best/last、原 YAML、原日志及其来源证据。

| 用途 | 原路径 |
|---|---|
| DNER 本次正式运行 | `.../LightText_checkpoints/model_base_early_exit_4_dner/20261116_232004/` |
| 纯四层 | `.../LightText_checkpoints/model_base_early_exit_4/20260703_144243/` |
| 完整模型 | `.../LightText_checkpoints/model_base/20260703_235302/` |
| TCAB | `.../LightText_checkpoints/model_base_early_exit_4_tunet_tcab/`；具体子目录以盘点和原 manifest 为准 |

DNER `best.pth` 与 `last.pth` 各625387860字节，SHA256：

```text
best e9162ec6f2d41e1e9f8ee4db12f1c620853aad976b1cdbeb1615dbf07bf7bba0
last 0207be8dc21120a7f09fa086012efdcb16142824fce0f411daf6f318eb85b69c
```

checkpoint 未进行反序列化或 GPU 加载；只读计算文件哈希。
原 DNER 目录还有 `20261116_231053` 短检，不可当作正式 best 来源。
权重、数据传输方案与命令见迁移清单。

## 6. 接下来从哪里继续

优先处理 DNER 的旧登记与真实终态，不启动同名训练：

1. 在原控制端、原 owner 和新鲜 lease 下，核验原 PID/boot/start、原日志、原源和终态，规范归档一次。
2. 原完整评价入口：`pipeline/research/nonbacktracking_export_c7f351cc/execute_complete.py`。
   仅原 best，9508表达/141场景，纯四层/完整/TCAB三对照、五组、原 abs1e-6。
3. 只有上述原协议真实 PASS 后，才用当前
   `pipeline/research/nonbacktracking_lifecycle_a8b8c2b1/execute_guarded.py`
   接续原固定 TRAIN 诊断。不是直接执行旧 tick 的写死 token。
4. 原计划 SHA `476464e20cfe93a4d3b033320c92843c8e2ab891f5bf57c01d7c73b3857780fd`；
   cohort SHA `fd420e6b46d73acb11851c9ce55db100b88c1a3a0174583581fa94f7180e753f`。
   8房间32表达、5臂、4原生forward/20独立rematch/40 parity，
   abs2e-6/rel1e-6，原匹配/停止规则不变。registry实际164/保守167/预留48。
5. 不重复已完成 CPU 契约、TRAIN2、正式 launch、DOAM 生命周期探针或旧完整评价。

以上入口含原 server3、GPU1、owner、source 和 lease 绑定。**搬到新机不等于绑定自动获准改变**：
先记录迁移授权、目标机身份/GPU/数据/环境、同一原 checkpoint/config/source 的新绑定与资源证据；
保留原计划、checkpoint 和失败历史，不通过修改历史快照绕过检查。

## 7. 不得当作已解决的事项

- DOAM 科学4/20/40已完成、固定32主判据不支持；原控制器退出失败且 failed raw缺失。
  保留 validation/inconclusive，不重跑原suite或原正常退出探针、不补PASS。
- DSAR、DPTM、DWVR、DCPH、DBCR 原 native/export abs1e-6数值检查失败，原因未知。
  不放宽阈值、不重导挑值、不在缺原前置PASS时跑冻结TRAIN。
- DVRI/DSPL/DSER 修订预算耗尽，不能自动R3；DROC/DCSS缺独立物理反事实标注。
- DCSR/DLOR等固定TRAIN“不支持”是局部结论，不是全部描述回读路线无效。
- TCAB UNet lr_mult0.1与纯四层/近期0.01不同；成绩比较不能直接当模块因果归因。
- 原taxonomy86条初查失败与后续outcome-blind compatibility并存；scene bootstrap不是seed不确定性或多重校正。

## 8. 调度与权限交接

原控制端：`D:/project/lighttext_project`；`PROJECT_ROOT=D:/project`，
`PYTHONDONTWRITEBYTECODE=1`，使用 `python -B`。
原 owner：`01a0498a-ec28-77f3-8a92-43d67ba679cd`。
旧 Windows 任务 `Pipeline-lighttext_project` 当前核验为 Disabled。
本次没有修改 heartbeat、调度、白名单、goal、queue 或 research_work。

旧配置仅授权 server3 GPU0/1，每卡1任务；server1禁派发、server2/4空白名单。
迁移意图不等于所有新服务器/GPU已授权。新机身份、路径和资源确定后再正式更新运行绑定。
不复制锁、PID、预约和旧 lease 作为新机当前状态，也不让两套控制端同时接管。
平台旧 shell 同步脚本有破坏性历史行为；不要在受保护 checkout 上直接运行 `scripts/sync.sh`。

当前 GitHub 目标尚未提供。此交接不会声称“已上传”，直到远端提交 SHA 与快照校验通过。
