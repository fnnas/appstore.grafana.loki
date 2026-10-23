# Repository guide

This repository packages upstream Grafana Loki binaries as fnOS application archives.

## Key files

- `.github/workflows/main.yml`: manually dispatched **Test Release** workflow; owns the upstream and package versions and calls both architecture workflows.
- `.github/workflows/pack_amd64.yml`, `pack_arm64.yml`: download binaries, substitute manifest fields, build archives, and upload artifacts. Keep shared packaging behavior consistent between them.
- `app/`: Loki configuration template and application icons. The build adds the executable as `app/loki-linux`.
- `package/`: manifest template, Bash lifecycle hooks in `cmd/`, JSON resource/privilege configuration, and localized descriptions in `i18n/`.
- `ui/`: included in the application archive; preserve its tracked placeholder.
- `README.md`: Chinese installation, usage, and build instructions.

## Making changes

- For version bumps, verify the official stable release at `https://github.com/grafana/loki/releases/latest` and confirm both `loki-linux-amd64.zip` and `loki-linux-arm64.zip` exist.
- Set `target_version` without a `v` prefix and `this_pack_manifest_version` to `<upstream-version>-<package-revision>` in `main.yml`. Increment the revision for packaging changes to the same upstream release. Keep the README version example synchronized.
- Preserve `package/manifest` build placeholders (`this_pack_manifest_version`, `this_pack_manifest_arch`, `this_pack_manifest_platform`, and `target_version`). `${manifest.desc}` is resolved through `package/i18n/`.
- Preserve these architecture mappings:

  | Upstream suffix | Manifest `arch` | Manifest `platform` |
  | --- | --- | --- |
  | `amd64` | `x86_64` | `x86` |
  | `arm64` | `arm` | `arm` |

- The outer archive is `appstore.grafana.loki.<package-version>.<architecture>.tgz` containing `package/`. Its `package/app.tgz` contains `app/` and `ui/`; `app/loki-linux` must be executable. Keep downloaded binaries and generated archives out of commits.
- Runtime hooks use `TRIM_APPDEST` for the installed application and `/var/apps/grafana.loki/shares/grafana.loki/` for persistent configuration and data. The default HTTP port is `3100`. The configuration template is copied only when the user's configuration is absent; preserve existing configuration during upgrades.
- When changing application descriptions, update the relevant entries in `package/i18n/zh-CN`, `en`, and `ja-JP`, preserving their existing format and language style.

## Validation

- Run `git diff --check`. For workflow edits, run `actionlint` when available and distinguish existing findings from new ones.
- For Bash edits, run `bash -n <changed-script>` under Linux or WSL. Execute lifecycle hooks only in an appropriate fnOS test environment.
- For packaging changes, reproduce both architecture workflows in a temporary copy under Linux/WSL: their `sed -i` commands modify the manifest in place. Check both archive layers with `tar -tzf`, rendered manifest values, and executable permissions.
- For version updates, verify downloads against the release's `SHA256SUMS`. For version or configuration changes, run the matching binary with `-config.file=app/config.loki.template.yaml -verify-config=true` on a compatible architecture.
- A CI validation build uses **Test Release** with `make_release: false`; setting it to `true` creates a GitHub prerelease and uploads assets.
- Documentation-only changes need a content review and whitespace check.

## Commit messages

- Use a Chinese `feat: <变更摘要>` title, following the style in `appstore.grafana`.
- Follow the title with Chinese overview paragraphs describing the changes and resulting behavior.
- Then list each changed repository-relative file path on its own line, followed by bullets in the form `- <主题> <新增|修改|修复|删除>：<具体变化>`. Related files may share a group; separate groups with blank lines.
- Describe only changes included in the commit. Keep validation results in the work report or PR description; omit a separate validation section from the commit message.

Example:

```text
feat: 更新 Loki 版本及打包说明

更新 Loki 上游版本与应用包版本，使 amd64 和 arm64 工作流使用同一版本构建安装包，并同步版本更新说明。

.github/workflows/main.yml
- 版本配置 修改：同步更新 target_version 和 this_pack_manifest_version。

README.md
- 打包说明 修改：同步版本更新示例。
```
