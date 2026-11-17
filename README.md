# LightText 私有迁移交接包

日期：2026-11-17。当前源码、实验清单和跨服务器交接材料；不是新的训练或评价结果。

## 从这里开始

- [交接文档](HANDOVER.md)
- [实验统计](EXPERIMENTS.md)
- [迁移验收清单](MIGRATION_CHECKLIST.md)
- [286个归档实验ID清单](experiments_unique.csv)
- [408条研究候选清单](candidates_all.csv)
- [实际server3环境版本](requirements-server3-observed.txt)
- [文件清单与SHA256](UPLOAD_MANIFEST.json)

## 完整源码与小型研究证据

为通过 Edge/GitHub 网页上传并保留两万多个文件的目录结构，内容分为若干独立 ZIP。
所有分包解压到同一空目录，合起来是完整的已校验迁移快照，不是只上传少数配置。

- [lighttext-code-20261117-part01.zip](lighttext-code-20261117-part01.zip)：6171 个文件，17032918 字节。
- [lighttext-research-records-20261117-part01.zip](lighttext-research-records-20261117-part01.zip)：10626 个文件，19959685 字节。
- [lighttext-research-records-20261117-part02.zip](lighttext-research-records-20261117-part02.zip)：8450 个文件，19436950 字节。

```bash
mkdir -p extracted/lighttext_project
for archive in lighttext-*-20261117-part*.zip; do
  python -B -m zipfile -e "$archive" extracted/lighttext_project
done
cd extracted/lighttext_project
python -B handover/tools/verify_snapshot.py
```

源码在解压目录的 `lighttext/`，控制/研究文件在外层 `scripts/`、`pipeline/`。
原训练版本为 `9c81f83ab9b3e812c5b6be214060c2376d4df0e4`；网页上传的提交不是原训练SHA。

统计为306条账本记录、286个归档ID，加DNER未归档项共287个已知ID。
408条候选记录不等于408次完成训练。TCAB最好登记0.5032，目标0.5100未达。
DNER已观察到第23轮自然早停，但旧队列仍登记running，本次未规范归档、重启或做完整评价。

## 未包含的内容与边界

- 数据、权重、编译二进制、密钥、Git内部目录、环境与缓存、大原始研究输出需单独传输。
- 原Git历史bundle保存在本地交接目录，未上传；旧提交图不能从源码ZIP中恢复。
- 数据约37.33GiB，历史checkpoint根约253.51GiB；原文件没有删除。
- 原失败、挂起、拒绝与未执行状态均保留，迁移不构成重新授权或科学PASS。
- ZIP内部分状态报告是在打包前生成，含“尚未上传”等当时状态；当前上传位置以本仓库为准。
- 目标计算服务器/GPU未指定，本次仅存档，不修改原资源白名单、调度或训练。

这是私有交接仓库。现有公开 `songchuanle-1/lighttext-project` 保持不变，
内部服务器、授权和实验审计材料不发布到公开仓库。
