# 从 PCL-N 主仓库生成 Patch（当前方案）

**主路径已在 PCL-N 内完成，无需再靠 repository_dispatch 才能出 Patch。**

## 每次发版自动跑

| 工作流 | 时机 |
|--------|------|
| `release-stable_publish.yml` | 正式版 assets 上传成功后 |
| `release-beta_publish.yml` | 预发布 assets 上传成功后 |
| 调用 | `generate-launcher-patches.yml` |

脚本位于主仓：`tools/pcl-n-patches/`。

行为：

1. 等待本仓库 Release 上已有资产  
2. 对 **每一个历史版本 → 当前 tag** 生成 HDiffPatch（按 RID/变体）  
3. 把 `index.json` + patches **挂回同一个启动器 Release**  
4. 若配置了 `PATCHES_REPO_TOKEN`，再镜像到专用 `PCL-N-Patches` 仓  

手动发版跳过 Patch：workflow_dispatch 勾选 `skip_patches`。

## 仅维护专用 Patch 仓时

仍可在本仓库 Actions 手动跑 **Generate version patches**，或接收旧的 `repository_dispatch`（`pcln-release`）。
