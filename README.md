# DitChord

[BitChord](https://github.com/kushagrasinghx/BitChord) 的个人构建版本。

本仓库**不包含应用源码**，只有一个 GitHub Actions 工作流：它会拉取上游源码，在 CI 工作区里打上一组小补丁，然后构建并发布 APK。所有构建产物只发布在本仓库的 [Releases](../../releases) 中。

## 与上游的差异

| 项目 | 上游 | DitChord |
| --- | --- | --- |
| 应用 ID（`applicationId`） | `com.music.bitchord` | `com.ditchord` |
| 应用名 | BitChord | DitChord |
| 简体中文区域的 `gl` 参数 | `SG` | `US` |
| 应用内更新来源 | 上游仓库 Releases | 本仓库 Releases |
| 签名密钥 | 上游的 | 我自己的 |

补丁只在 CI 构建时应用，不会提交到任何仓库。如果上游改动导致补丁锚点失效，构建会直接失败，而不是产出一个没打补丁的包。

## 下载与安装

前往 [Releases](../../releases) 下载对应的 APK：

- `DitChord-vX.Y.Z.apk`：通用包
- `DitChord-vX.Y.Z_arm64-v8a.apk`：大多数现代手机
- `DitChord-vX.Y.Z_armeabi-v7a.apk`：旧款 32 位手机
- `DitChord-vX.Y.Z_x86_64.apk`：x86_64 设备 / 模拟器
- `DitChordWear-vX.Y.Z.apk`：Wear OS 手表端

> 由于应用 ID 与签名都和上游不同，DitChord 可以与官方 BitChord 共存，但无法覆盖安装官方版本，数据也不互通。

安装后应用内的"检查更新"会指向本仓库的 Releases。

## 如何构建

### 1. 准备 Secrets

在本仓库的 **Settings → Secrets and variables → Actions** 中添加：

| Secret | 是否必需 | 说明 |
| --- | --- | --- |
| `KEYSTORE_BASE64` | 必需 | 签名用 keystore 文件的 Base64 |
| `KEYSTORE_PASSWORD` | 必需 | keystore 密码 |
| `KEY_ALIAS` | 必需 | 密钥别名 |
| `KEY_PASSWORD` | 必需 | 密钥密码 |
| `LASTFM_API_KEY` | 可选 | Last.fm API Key |
| `LASTFM_SECRET` | 可选 | Last.fm Secret |
| `LISTEN_TOGETHER_SERVER` | 可选 | 一起听服务器地址，留空则使用代码里的默认值 |

生成 keystore 并转为 Base64（只需做一次，请妥善备份）：

```bash
keytool -genkeypair -v -keystore bitchord-release.jks \
  -alias ditchord -keyalg RSA -keysize 2048 -validity 10000

# Linux
base64 -w 0 bitchord-release.jks
# macOS
base64 -i bitchord-release.jks
```

> ⚠️ 此后所有更新都必须使用同一个 keystore 签名。丢失后用户只能卸载重装。

### 2. 触发构建

进入 **Actions → DitChord Release → Run workflow**。

- `upstream_ref`：可选，填写上游的分支、tag 或 commit。留空则使用上游默认分支的最新代码。

构建完成后，Release 会自动创建，tag 与上游版本号保持一致（如 `v1.2.3`）。带 beta 后缀的版本会被标记为 pre-release。同一版本重复构建时，旧的 Release 会被新的覆盖。

### 3. 切换源码仓库（可选）

默认直接拉取上游。如果想改用自己的 fork，修改工作流顶部的：

```yaml
env:
  UPSTREAM_REPO: 你的用户名/BitChord
```

fork 不会自动同步，需要手动 **Sync fork**。如果 fork 是私有仓库，还需要给 checkout 配置有读权限的 token。

## 已知限制

- GitHub 的 `/releases/latest` 不会返回 pre-release，因此 beta 版本无法通过应用内更新检测到，需要手动下载。
- 本仓库的 tag 指向本仓库自己的提交，实际构建所用的上游 commit 记录在每个 Release 的说明里。

## 致谢与许可

DitChord 基于 [kushagrasinghx/BitChord](https://github.com/kushagrasinghx/BitChord) 构建，所有应用代码与权利归原作者所有，使用时请遵守上游仓库的许可证。本仓库的工作流文件仅用于个人构建。
