# 迁移验收清单

此清单不自动执行训练、停止旧调度、变更白名单或迁移运行进程。新服务器地址和GPU尚未提供。

## A. 代码与历史

- [ ] 指定 GitHub 仓库，建议私有；确认有写权限后推送并核对远端 SHA。
- [ ] 在新机 clone 到 `<project-parent>/lighttext_project`，保留外层/内层结构。
- [ ] 先运行 `python -B handover/tools/verify_snapshot.py`，检查每个导出源文件 SHA。
- [ ] 单独传输本地 `source-history.bundle`，不要把它误当成训练权重。
- [ ] 验证 bundle，再恢复原训练 commit 图。快照 Git 与原训练 Git 是两套身份，不能混用 SHA。

```bash
git clone /transfer/source-history.bundle /new/project/lighttext_original_history
git -C /new/project/lighttext_original_history bundle verify /transfer/source-history.bundle
git -C /new/project/lighttext_original_history rev-parse 9c81f83ab9b3e812c5b6be214060c2376d4df0e4
```

原工作区未跟踪候选已保存在 GitHub 的源码树中；bundle只保存原已提交历史，不包含这些未跟踪文件。
保留原目录备份，在确认全量文件和历史都可恢复前不删除任何来源。

## B. 数据、原权重与研究证据

- [ ] 复制原数据根，包括大小写不同的目录、标注、分词器、features和backbone。
- [ ] 复制 DNER 原正式运行目录的 best、last、原YAML与日志，不选择短检目录。
- [ ] 复制纯四层/完整/TCAB原best，以及将继续研究的其他原checkpoint。
- [ ] 复制原DNER私有runtime与其源码/环境/数据fingerprint，以保持来源证据。
- [ ] 复制 `source_manifest.json` 中标为外部的原研究输出和完整日志；原文件名和哈希不改。
- [ ] 对关键checkpoint用SHA256核对；数据目录至少验证文件数、总大小与原数据pin。

在**新机**手动设置真实目标路径后，可用下面的只增量复制方式；不要加 `--delete`：

```bash
mkdir -p /new/data /new/checkpoints
rsync -a --partial --info=progress2 \
  songcl@192.168.1.120:/sdc1/songcl/IPDN/data/ /new/data/
rsync -a --partial --info=progress2 \
  songcl@192.168.1.120:/sdc1/songcl/Project/LightText_project/LightText_checkpoints/model_base_early_exit_4_dner/20261116_232004/ \
  /new/checkpoints/model_base_early_exit_4_dner/20261116_232004/
sha256sum /new/checkpoints/model_base_early_exit_4_dner/20261116_232004/best.pth
```

密码/SSH私钥不通过仓库或命令参数传递。优先用现有SSH key，不据账号推断sudo权限。
如果需要完整历史checkpoint，原根总占用约253.51GiB，复制前先核验目标磁盘空间。
上述 `/new/...` 是占位示例，本次未执行。

## C. 环境

- [ ] 记录新机hostname、GPU UUID/架构、driver、磁盘和CUDA Toolkit。
- [ ] 先按 `requirements-server3-observed.txt` 比对，而不是直接安装旧 README 的另一套环境。
- [ ] 判断 Python3.8/torch1.12/cu113 是否支持目标GPU；如硬件不兼容，记录环境迁移，不宣称与历史完全一致。
- [ ] 在新环境编译 pointnet2 与 pointgroup_ops，不搬运旧 `.so` 冒充可用。
- [ ] 在新机检查 model、dataset、gorilla、transformers、spconv及两个扩展能否导入。
- [ ] 控制端工具使用独立的合适Python；不要用训练端Python3.8运行有较新标准库API的pipeline脚本。

```bash
cd /new/project/lighttext_project/lighttext/pointnet2
python -B setup.py build_ext --inplace
cd /new/project/lighttext_project/lighttext/framework/lib
python -B setup.py build_ext --inplace
```

这些只是编译位置示例，必须在正确torch/CUDA/编译器/架构配置后执行。本次没有在新机测试。

## D. 配置和控制端

- [ ] 仅在部署副本中做路径适配：`work_dir`、`train.pretrain`、`data.*.data_root`、`model.data_root`。
- [ ] 不修改原档YAML、seed1999、50epochs、原PolyLR、patience10、4文本层、6解码层和loss。
- [ ] 不把driver升级、compiled-kernel变化或路径改动隐去；分别保留原manifest和迁移绑定。
- [ ] 获得目标服务器/GPU明确授权后再修改当前资源表，不照搬旧白名单作为新机权限。
- [ ] 旧后台保持Disabled；切换前明确唯一控制端和owner，避免旧heartbeat与新控制端双派发。
- [ ] 不在新机重放旧token、PID、running.json、GPU预约、锁或带旧token的execute命令。
- [ ] 先只读对账DNER的自然终态，再通过原规范入口归档一次，不盲resume。

`handover/runtime/songcl_research_pipeline` 是控制工具源码快照，不是已经安装或已迁移的服务。
历史研究脚本可能硬编码Windows skill路径和server3身份；迁移前逐项核对，不全局替换旧证据文本。

## E. 研究续接

- [ ] 复用已存在pure4/full/TCAB完整导出，不重训或重导基线。
- [ ] DNER原best完整评价一次：9508/141、3对照5组、原abs1e-6。
- [ ] 真正原前置PASS之后，才执行原固定32表达TRAIN诊断4/20/40。
- [ ] DOAM退出失败、5个数值hold、R3/标注授权问题继续保留。
- [ ] 本地/远端/新机全量校验完成后，才考虑旧机清理；本次没有删除旧数据或停止原服务。

## 本次未完成

- GitHub目标地址未提供，因此尚未推送。
- 目标服务器地址、GPU、磁盘与账号未提供，因此未部署或实际搬运数据/权重。
- DNER规范归档、完整评价及冻结TRAIN诊断未在本次交接工作中执行。
