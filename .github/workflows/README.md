# GitHub Actions 自动打包

本仓库通过 GitHub Actions 在 push / 标签 时自动编译并产出二进制。

## 触发方式

- **自动**：推送到 `main` / `master` 或发起 PR → 运行对应平台的构建（产物为 Artifacts，保留 90 天）。
- **发布 Release**：创建并推送**标签**（如 `v1.0.0`）→ 各平台构建完成后，将 `swoole-cli-v<version>-<os>-<arch>.tar.xz` 上传到该标签的 Release。
- **手动**：在 Actions 页选择对应 workflow，点击 “Run workflow”（`workflow_dispatch`）。

## 平台与产物

| Workflow              | Runner           | 产物名称（示例）                    |
|-----------------------|------------------|-------------------------------------|
| linux-x86_64.yml      | ubuntu-latest    | swoole-cli-v*.*.*-linux-x64.tar.xz  |
| linux-aarch64.yml     | ubuntu-24.04-arm | swoole-cli-v*.*.*-linux-arm64.tar.xz |
| macos-aarch64.yml     | macos-15         | swoole-cli-v*.*.*-macos-arm64.tar.xz|
| macos-x86_64.yml      | macos-15-intel   | swoole-cli-v*.*.*-macos-x64.tar.xz  |

## 发布新版本步骤

1. 在仓库中打好标签并推送，例如：
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```
2. 在 [Actions](https://github.com/YOUR_ORG/swoole-cli/actions) 中等待各平台 workflow 完成。
3. 在 [Releases](https://github.com/YOUR_ORG/swoole-cli/releases) 中打开该标签，即可下载各平台 `.tar.xz` 包。

## 本 Fork 修改说明

- **默认内存限制**：将 PHP 默认 `memory_limit` 从 `128M` 改为 `512M`（见 `main/main.c`），便于运行大型 phar 等场景。
- **Release 权限**：各构建 job 已增加 `permissions: contents: write`，以便在 fork 中正常上传 Release 资源。
- **Release 上传失败修复**：`make.sh archive` / Cygwin 的 `cygwin-pack.php` 等产出的包均使用 **SWOOLE_VERSION** 命名。原先 workflow 用 `php -v` 的 PHP 版本作为 APP_VERSION，导致文件名不一致、上传找不到文件。已改为 `APP_VERSION=$(./bin/swoole-cli -r "echo SWOOLE_VERSION;")`（linux/macos/windows-cygwin 已统一）。
- **手动触发 (workflow_dispatch)**：job 条件中 `github.event.head_commit.message` 在手动触发时可能为空，已改为 `github.event.head_commit.message || ''`，避免 job 被误跳过。
- **云存储上传**：已从本 fork 的 workflow 中移除所有「upload to cloud object storage」步骤与 job，不再依赖腾讯云 OSS 及对应 secrets/vars。

与上游同步时，注意保留上述修改或合并后重新应用。
