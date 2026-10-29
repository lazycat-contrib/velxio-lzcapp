# Velxio for LazyCat

[Velxio](https://github.com/davidmonterocrespo24/velxio) 的懒猫 LPK v2 打包仓库，仅发布到**喵喵商店**，不重复提交官方商店的同名应用。

## 功能与运行方式

- 浏览器中编辑电路与代码、编译和仿真 Arduino、RP2040、ESP32 项目；具体支持范围以上游 OSS 自托管镜像为准，不等同于商业版全部板卡。
- 服务明确配置 `user: root`，不配置 `run_as`，不额外授予 privileged、KVM 或 Docker socket。
- 懒猫网关登录保持启用。上游 OSS 没有独立账号体系，本包不新增免密登录改造，也不保证应用内部的跨用户项目隔离；仅授权给可信用户。
- 注入懒猫官方文件选择器拦截脚本，供原生导入/导出入口使用。不声明未经适配的网盘右键打开接口。
- 本包构建目标为 Linux amd64。上游 `master` 同时提供 amd64/arm64 镜像，但本仓库不宣称已验证 arm64 应用包。

## 版本与镜像

上游 GHCR 当前只发布滚动的 `master` 标签，没有与 GitHub Release `v3.0.5` 对应的镜像标签。因此 **`0.1.0` 为本打包仓库版本，并非上游版本**。

首次打包使用 2026-10-28 构建的镜像，源码 revision 为 [`6d1bf83e90186ca96c70dc986e7ed7f042dc0328`](https://github.com/davidmonterocrespo24/velxio/tree/6d1bf83e90186ca96c70dc986e7ed7f042dc0328)。Manifest 固定 amd64 镜像摘要，通过 `ghcr.1ms.run` 分发；镜像摘要与 GHCR 上游一致。

自动化每六小时检查 `master`：目标平台摘要不变时保留版本，变化时递增本包 patch 版本；所有分发引用均固定摘要。不会把 `master` 冒充为某个上游 Release。

## 第一次启动

1. 从喵喵商店或 GitHub Release 安装 LPK。
2. 上游 entrypoint 会联网更新板卡索引、安装 Arduino 核心，然后启动 HTTP 服务。首次下载可能耗时较长，健康检查预留 30 分钟启动宽限；页面可访问不代表所有板卡核心安装成功。
3. 从示例或空项目开始，编辑代码后运行仿真。若编译提示缺少核心，先检查应用日志及到 Arduino/GitHub 下载服务的网络连接。
4. 重要项目请及时使用导出功能备份；浏览器端数据不应被视为跨设备云端账号存储。

镜像压缩层约 1.62 GB，解压镜像及首启工具链需要额外磁盘空间。首次编译通常比缓存后的编译慢。

## 数据目录

| 懒猫路径 | 容器路径 | 用途 |
|---|---|---|
| `/lzcapp/var/data` | `/app/data` | 上游运行数据与自动生成的密钥 |
| `/lzcapp/var/arduino15` | `/root/.arduino15` | Arduino CLI 配置、索引与板卡核心 |
| `/lzcapp/var/arduino-libraries` | `/root/Arduino` | Arduino 用户库 |
| `/lzcapp/cache/ccache` | `/var/cache/ccache` | 可重建编译缓存 |
| `/lzcapp/cache/build` | `/var/lib/velxio-build` | 可重建增量编译目录 |

系统可清理 cache；清理后需要重新编译，不应在 cache 内存放唯一的项目副本。不挂载整个 `/root`，避免遮蔽镜像内 ESP-IDF 工具链。

## 本地校验与构建

```sh
lzc-cli project lint .
actionlint .github/workflows/lazycat.yml
lzc-cli project release . -o dist/velxio.lpk
```

## 自动发布

使用 `ca-x/lazycat-github-action/.github/workflows/lazycat.yml@v1`：检查镜像 → 校验加速镜像摘要 → 构建 LPK → 创建版本化 GitHub Release → 发布喵喵商店。

GitHub Actions 手动运行默认 `dry-run: true`；确认发布时设为 `false`。同版本商店已存在时跳过；Release 中已存在的精确版本资产将校验 SHA256 后用于补发。

组织 Secrets 需要授权本仓库：`APPSTORE_URL`、`APPSTORE_TOKEN`，以及可选的 `PRIVATE_STORE_GROUP_CODES`。本仓库不使用官方商店凭据，也不继承其他应用的 `APP_ID`。

## 来源与许可

- 上游：[davidmonterocrespo24/velxio](https://github.com/davidmonterocrespo24/velxio)，AGPL-3.0-or-later（包括遵守 AGPL 的商业使用）；不希望承担 AGPL 源码提供义务的闭源集成，可查看上游 `COMMERCIAL_LICENSE.md`。
- 图标由上游 `frontend/public/favicon.svg` 转为 512×512 PNG。
- 网盘拦截脚本来自 [懒猫开发者文档](https://developer.lazycat.cloud/lazycat-injects/lzc-file-chooser-inject.js)，保持原样，源码入口见 `content/lazycat-injects/README.md`。
- 本仓库不修改 Velxio 业务源码；部署需要遵循上游许可及第三方组件许可。
