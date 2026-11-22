# 评估结果管理

本文档介绍评估结果的存储方式、如何上传到 Google Cloud Storage (GCS)，以及同步和回填指标的工作流程。

## 存储位置
评估结果存储在本地：
- `results/suites/`（仓库内的默认位置）。
- 或自定义的外部目录（例如 `~/guidance-evals`）。

每个任务有独立目录，包含：
- `evals.json`：所有试验和检查的摘要。
- `evals.md`：该任务的 Markdown 报告。
- 按重复次数编号的目录（`1`、`2`、`3`...），包含 agent 日志和评分器结果。

---

## 上传到 GCS
使用 `src/harness/upload-suite.ts` 将任务上传到 GCS。

### 工作原理
1. 要求任务目录中存在 `evals.json`（确保评估已运行）。
2. 将任务目录中的所有文件上传到 `gs://guidance-evals/<suite-name>/`（跳过 `node_modules`）。
3. 并发上传（每次分块 50 个文件）。

### 命令
```bash
# Upload full job from local repo results
pnpm upload <suite-name>

# Upload with flags or from custom directory
node src/harness/upload-suite.ts <suite-name> [--summary-only] [custom_results_dir]
```

---

## 工作流程

### 1. 从 GCS 同步（下载）
如果需要拉取本地没有的 GCS 任务（例如对其运行回填）：

```bash
gcloud storage rsync gs://guidance-evals ~/guidance-evals --recursive
```

*   **递归**：**必须**传入 `--recursive`（或 `-r`），否则不会复制任务文件夹内的文件！
*   **默认安全**：默认不会删除本地文件（如 `.git`）。
*   **破坏性标志**：如果要删除桶中不存在的本地文件，使用 `--delete-unmatched-destination-objects`。
    > [!WARNING]
    > 从 GCS 拉取到本地时使用此标志，**会删除你的 `.git` 文件夹**（因为 `.git` 不在 GCS 上）。使用此标志前，请先将 `.git` 文件夹移出目标目录！
*   **试运行**：查看操作效果而不改动文件：
    ```bash
    gcloud storage rsync gs://guidance-evals ~/guidance-evals --recursive --dry-run
    ```

### 2. 回填指标
当指标报告逻辑变更时，可以回填目录中的所有任务：

```bash
# Backfill local repo results
node src/harness/backfill.ts

# Backfill a custom directory (e.g., synced from GCS)
node src/harness/backfill.ts ~/guidance-evals
```
这会更新每个任务目录中的 `evals.json` 和 `evals.md`。

### 3. 推送更新到 GCS
回填后，应将更新后的摘要推送回 GCS。

可使用 **`--summary-only`** 标志仅上传 `evals.json` 和 `evals.md`，使同步极快。

要从自定义目录批量上传**所有**任务，可用简单的 shell 循环：

```bash
# Bulk upload ONLY summaries for all jobs in ~/guidance-evals
for d in ~/guidance-evals/*/ ; do
    suite=$(basename "$d")
    node src/harness/upload-suite.ts "$suite" ~/guidance-evals --summary-only
done
```

单个任务：

```bash
# Upload ONLY summaries from your custom directory
node src/harness/upload-suite.ts <suite-name> ~/guidance-evals --summary-only

# Upload EVERYTHING from your custom directory
node src/harness/upload-suite.ts <suite-name> ~/guidance-evals
```