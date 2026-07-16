# PCL-N-Patches

PCL N 启动器 **二进制差分（Patch）** 专用仓库。

主程序仓库：[PCL-N](https://github.com/MuXue1230-owo/PCL-N)  
本仓库 **不** 存放完整安装包，只生成、索引并发布 **版本间 Patch**。

## 策略

每当有新版本 `V_n` 发布时，对 **每一个历史版本** `V_0 … V_{n-1}`：

1. 下载对应 RID / 产物变体的完整包（zip / tar.gz）
2. 解出主程序二进制（`PCL.Desktop(.exe)`）
3. 用 [HDiffPatch](https://github.com/sisong/HDiffPatch) 生成  
   `from → to` 差分：`V_i → V_n`
4. 写入 manifest，并作为 GitHub Release 资产上传

```
历史: v0.1.0  v0.2.0  v0.3.0
新增:                 v0.4.0

生成 Patch:
  v0.1.0 → v0.4.0
  v0.2.0 → v0.4.0
  v0.3.0 → v0.4.0
  （每个 RID × SelfContained|NoRuntime × WithPlugin|NoPlugin）
```

客户端更新路径：

1. 查本仓库最新 manifest（或 `index.json`）
2. 若存在 `current → latest` 且 patch 体积明显小于全量包 → 下载 patch 并应用
3. 否则回退全量下载
4. 替换本地二进制并重启

## 目录结构

| 路径 | 说明 |
|------|------|
| `scripts/generate_patches.py` | 主生成器：全历史 → 目标版本 |
| `scripts/apply_patch.py` | 本地应用 patch（测试 / 启动器可参考） |
| `scripts/bootstrap_hdiffpatch.py` | 下载 HDiffPatch 预编译工具 |
| `schema/patch-manifest.schema.json` | manifest JSON Schema |
| `manifests/` | 生成结果示例 / 本地输出（CI 产物为主） |
| `docs/CLIENT.md` | 启动器接入说明 |
| `.github/workflows/generate-patches.yml` | 新版本时自动生成并发布 |

**注意：** 大型 `.hdiff` 文件 **不要** 提交进 Git。CI 发布到 **本仓库的 GitHub Release**。

## 本地用法

### 依赖

- Python 3.11+
- 网络（访问 GitHub API / Releases）
- 可选：`GH_TOKEN` 提高 API 限额；私有资源时必需

```powershell
cd D:\PCL-F\PCL-N-Patches
python -m pip install -r requirements.txt
python scripts/bootstrap_hdiffpatch.py
```

### 为某个目标 tag 生成相对所有历史版本的 Patch

```powershell
$env:GH_TOKEN = "ghp_..."   # 可选
python scripts/generate_patches.py `
  --source-repo MuXue1230-owo/PCL-N `
  --target-tag v1.0.0 `
  --out-dir artifacts/v1.0.0 `
  --max-from-versions 50
```

产物：

```
artifacts/v1.0.0/
  index.json                          # 总索引
  manifests/<variant>.json            # 分变体清单
  patches/<rid>/<variant>/<from>-to-<to>.hdiff
  patches/.../*.sha256
```

### 应用 Patch（测试）

```powershell
python scripts/apply_patch.py `
  --old path\to\PCL.Desktop.exe `
  --patch path\to\from-to.hdiff `
  --out path\to\PCL.Desktop.new.exe
```

## 与启动器发版 GA（主路径）

**每次 PCL-N 发布新版本，主仓库 GitHub Actions 会自动生成 Patch**（脚本在主仓 `tools/pcl-n-patches/`）：

1. `release-stable_publish.yml` / `release-beta_publish.yml` 上传完整包  
2. Job **`generate-patches`** → `generate-launcher-patches.yml`  
3. 对每一个历史版本 → 当前 tag 生成 HDiffPatch  
4. 将 `index.json` + patches **挂到同一启动器 Release**  
5. （可选）配置 `PATCHES_REPO_TOKEN` 后镜像到本专用仓  

手动发版若要跳过：workflow_dispatch 勾选 `skip_patches`。

## 专用 Patch 仓（可选镜像）

1. 新建 `MuXue1230-owo/PCL-N-Patches` 并推送本目录  
2. 主仓 Secret：`PATCHES_REPO_TOKEN`  
3. 主仓 Variable（可选）：`PATCHES_REPO`

## 许可证

与 PCL N 主项目一致（Apache-2.0），见 [LICENSE](LICENSE)。
