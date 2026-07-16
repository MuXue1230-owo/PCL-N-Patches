# 从 PCL-N 主仓库触发 Patch 生成

在 `PCL-F/.github/workflows/release-stable_publish.yml`（及 beta）的 `publish-assets` 成功后追加：

```yaml
  dispatch-patches:
    needs: [publish-assets]
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Notify PCL-N-Patches
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.PATCHES_REPO_TOKEN }}
          repository: MuXue1230-owo/PCL-N-Patches
          event-type: pcln-release
          client-payload: >-
            {
              "target_tag": "${{ github.event.release.tag_name || inputs.tag_name }}",
              "source_repo": "${{ github.repository }}"
            }
```

在 GitHub → Settings → Secrets 添加 `PATCHES_REPO_TOKEN`（classic PAT 或 fine-grained，scope 含 `PCL-N-Patches` 的 contents write）。

也可在 Actions 页对 **Generate version patches** 手动填写 `target_tag` 运行。
