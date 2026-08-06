# models.dev API Mirror

A mirror repository that tracks [`https://models.dev/api.json`](https://models.dev/api.json) and keeps a local copy of the data in sync automatically via GitHub Actions.

本仓库是 [`models.dev/api.json`](https://models.dev/api.json) 的镜像，通过 GitHub Actions 自动保持本地数据与上游同步，无需手动下载上传。

## Repository contents / 仓库内容

| Path | Description / 说明 |
|---|---|
| `api.json` | Latest snapshot of the upstream models metadata / 上游模型元数据的最新快照 |
| `.github/workflows/sync-api-json.yml` | Auto-sync workflow / 自动同步工作流 |

## How the auto-sync works / 自动同步机制

The workflow [`.github/workflows/sync-api-json.yml`](.github/workflows/sync-api-json.yml) performs the following on every run:

工作流 [`.github/workflows/sync-api-json.yml`](.github/workflows/sync-api-json.yml) 每次运行执行以下步骤：

1. **Schedule** — runs daily at `03:17 UTC` ([workflow:5-6](.github/workflows/sync-api-json.yml#L5-L6)) via a cron trigger, and can also be triggered manually (`workflow_dispatch`, [workflow:7](.github/workflows/sync-api-json.yml#L7)).
   **定时调度** — 每日 `03:17 UTC` 自动运行（cron 触发，[workflow:5-6](.github/workflows/sync-api-json.yml#L5-L6)），同时支持手动触发（[workflow:7](.github/workflows/sync-api-json.yml#L7)）。
2. **Fetch** — downloads the upstream file to a temporary path with `curl -fsSL`; a failed download aborts the run with an error instead of overwriting the local copy ([workflow:19-22](.github/workflows/sync-api-json.yml#L19-L22)).
   **拉取** — 用 `curl -fsSL` 将上游文件下载到临时路径；下载失败会报错终止，不会覆盖本地文件（[workflow:19-22](.github/workflows/sync-api-json.yml#L19-L22)）。
3. **Compare** — byte-compares the downloaded file with the local one via `cmp -s`. If identical, the run stops without committing anything, so no empty commits are produced ([workflow:24-29](.github/workflows/sync-api-json.yml#L24-L29)).
   **比对** — 用 `cmp -s` 做字节级比对；内容一致则直接结束、不产生提交，避免空提交刷屏（[workflow:24-29](.github/workflows/sync-api-json.yml#L24-L29)）。
4. **Commit & push** — on a difference, replaces `api.json` and auto-commits as `github-actions[bot]` with the message `chore(sync): update api.json from upstream`, then pushes ([workflow:36-44](.github/workflows/sync-api-json.yml#L36-L44)).
   **提交推送** — 检测到差异时替换 `api.json` 并以 `github-actions[bot]` 身份自动提交，提交信息为 `chore(sync): update api.json from upstream`，随后推送（[workflow:36-44](.github/workflows/sync-api-json.yml#L36-L44)）。

The workflow uses the automatically injected `GITHUB_TOKEN` with `contents: write` permission ([workflow:9-10](.github/workflows/sync-api-json.yml#L9-L10)) — no personal access token is needed.

工作流使用 GitHub 自动注入的 `GITHUB_TOKEN`（权限 `contents: write`，[workflow:9-10](.github/workflows/sync-api-json.yml#L9-L10)），无需配置个人访问令牌（PAT）。

## Manual trigger / 手动触发

To run the sync on demand:

如需手动执行一次同步：

1. Open the repository **Actions** tab / 打开仓库 **Actions** 页面
2. Select **Sync api.json from upstream** in the left sidebar / 在左侧选择 **Sync api.json from upstream**
3. Click **Run workflow** → **Run workflow** / 点击 **Run workflow** 按钮

## Changing the sync frequency / 修改同步频率

Edit the cron expression at [workflow:6](.github/workflows/sync-api-json.yml#L6). For example, `30 2 * * *` would run daily at 02:30 UTC.

修改 [workflow:6](.github/workflows/sync-api-json.yml#L6) 的 cron 表达式即可，例如 `30 2 * * *` 表示每天 02:30 UTC 运行。

## Local usage / 本地使用

```bash
git clone https://github.com/JochenYang/models.dev.git
```

Use the data in `api.json` directly, or filter by your language SDK of choice — the file is a plain JSON object keyed by provider id (e.g. `zhipuai`).

克隆后直接使用 `api.json` 即可，或按需用任意语言 SDK 解析。文件是普通 JSON 对象，以提供商 id（如 `zhipuai`）为键。
