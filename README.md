

# dsh-session-management · DSH 会话管理

中文 | [English](README.en.md)

![会话管理设置页](docs/screenshots/settings.png)

dsh-session-management 是 DeepSeek Harness（DSH）Web 的会话管理插件，**补足官方会话管理的缺失能力**。在「设置」面板内集中管理聊天会话：归档、取消归档、**真正删除本地聊天记录**、批量导出数据。支持中英双语，随 DSH 语言设置即时切换。

![已归档聊天管理弹窗](docs/screenshots/manage.png)

## 为什么需要这个插件

DSH Web 官方目前不提供完整的会话管理，长期使用会遇到三个痛点：

- **会话无法真正删除**：官方只提供「归档」，归档只是把会话从侧边栏隐藏，聊天记录仍完整保留在磁盘上，没有任何入口可以真正删掉它们；
- **归档后无法恢复**：官方没有「取消归档」功能，会话一旦归档就再也找不回来，只能「看不见但永远存在」；
- **导出只能逐个手动**：官方导出一次只能导出一个会话，会话攒多了要逐个点击很多次。

本插件逐一补齐：**真删**（删除磁盘上的会话日志文件）、**恢复**（单个 / 按组批量取消归档）、**批量导出**（与官方格式完全一致的 ZIP）。

| 能力 | DSH 官方 | 装本插件后 |
| --- | --- | --- |
| 归档会话 | 支持 | 支持，另有一键全量归档 |
| 取消归档 | 无入口 | 单个 / 按工作区 / 按月份批量 |
| 真正删除聊天记录 | 仅隐藏，不删除 | 删除磁盘日志文件，运行中会话自动跳过 |
| 导出会话数据 | 单个手动导出 | 一键批量导出，格式与官方字节级兼容 |

## 功能

### 已归档的聊天管理

点击「管理」打开管理窗口，两种分组视图随心切换：

- **按工作区**：每个工作区一个分组，组内按时间排序；
- **按月份**：按年月分组（`2026年8月`、`2026年7月`…），月份自动携带年份，跨年归档一目了然。

支持按创建日期 / 更新日期排序（升序 / 降序）、分组折叠与一键展开、按组批量操作——取消归档该组、**删除该组已归档的聊天**（仅作用于该组的归档会话，不影响未归档记录）。

| 分组与排序 | 组头批量操作 |
| --- | --- |
| ![归档管理弹窗](docs/screenshots/manage.png) | ![归档管理弹窗](docs/screenshots/manage.png) |

### 归档所有聊天

一键归档全部会话：记录完整保留，仅从侧边栏列表隐藏，随时可在管理窗口取消归档。

### 删除所有聊天

**真正删除**本地聊天记录：删除磁盘上的会话日志文件（`session.jsonl` / `session.jsonl.zstd` 及对偶文件），并同步清理工作区记账与归档标记。运行中的会话自动跳过，防止日志被写回。

### 导出数据

一键批量导出全部会话：自动遍历所有根会话（含其子代理会话），逐个生成与官方格式一致的 ZIP 归档（`dsh-session-<id>.zip`），无需再一个个手动点击。

- **内容完整**：会话日志、子代理会话与媒体附件一并打包；
- **格式兼容**：字节级兼容官方导出格式，可直接用官方工具查看或迁移；
- **删除前备份**：可先「导出数据」再「删除所有聊天」，清理的同时留住需要的数据。

### 中英双语

界面文案随「设置 → 通用设置 → 语言」即时切换，中英文档与界面同步维护。

## 安装

DSH 插件通过 **profile** 挂载（`dsh web` 对应 `web` profile）。安装后需**重启 `dsh web`** 生效。

**前置要求**：Node.js（含 npm）；`dsh plugin` 依赖 `pnpm`（缺失时见下方「常见问题」）。

### 方式一：从 npm 安装（推荐）

插件已发布到 npm（`dsh-session-management`）。安装命令取决于你如何运行 dsh：

- **全局安装 dsh**（`dsh` 命令可直接使用）：

  ```sh
  dsh plugin --profile web add dsh-session-management
  ```

- **通过 npx 运行 dsh**（未全局安装，平时用 `npx @deepseek-ai/dsh web` 启动）：

  ```sh
  npx -y @deepseek-ai/dsh plugin --profile web add dsh-session-management
  ```

装完重启 `dsh web`，打开「设置」即可看到「会话管理」入口。升级：将 `add` 换成 `update`（非全局安装时同样加 `npx -y @deepseek-ai/dsh` 前缀）。

> **版本提示**：若 npm/pnpm 的元数据缓存或镜像源同步延迟，导致安装到旧版本（提示 `declares no dsh.bundle`），请指定最新版本号重装：
>
> ```sh
> npx -y @deepseek-ai/dsh plugin --profile web add dsh-session-management@<最新版本号>
> ```
>
> 查询最新版本：`npm view dsh-session-management version`。

### 方式二：从 GitHub 仓库安装

```sh
git clone https://github.com/cokiscarazo-rgb/dsh-session-management.git
cd dsh-session-management

# Windows（PowerShell）
powershell -ExecutionPolicy Bypass -File scripts/install.ps1

# macOS / Linux
bash scripts/install.sh
```

安装脚本使用环境变量 `DSH_HOME` 定位 DSH 配置目录，未设置时默认 `$HOME/.dsh`。

安装脚本幂等，重复执行安全。它会完成两步：

1. 复制插件包到 `$DSH_HOME/profiles/node_modules/dsh-session-management/`；
2. 在 `$DSH_HOME/profiles/web/cordis.patch.yml` 注册 loader 条目：

```yaml
- insert:
    - id: dsh-session-management
      name: dsh-session-management
```

### 验证与卸载

安装成功后重启 `dsh web`，打开「设置」出现「会话管理」即生效；也可用 `dsh --profile web --dump-config`（非全局安装加 `npx -y @deepseek-ai/dsh` 前缀）确认插件配置层已挂载。若没有新入口，多半是安装后没有重启。

卸载：`dsh plugin --profile web remove dsh-session-management`（非全局安装加 `npx -y @deepseek-ai/dsh` 前缀），重启 `dsh web`；手动安装的则移除 `$DSH_HOME/profiles/node_modules/dsh-session-management/` 目录并删除 `cordis.patch.yml` 中的 insert 条目。

### 常见问题

- **提示 `'pnpm' 不是内部或外部命令` / `pnpm: command not found`**：`dsh plugin` 内部依赖 pnpm，先安装它：

  ```sh
  npm install -g pnpm
  ```

  或使用 Node 自带的 corepack：`corepack enable pnpm`。验证：`pnpm --version`。

- **安装后仍是旧版本 / 提示 `declares no dsh.bundle`**：npm/pnpm 元数据缓存或镜像源同步延迟所致。先确认 registry 源：`pnpm config get registry`（若为 npmmirror 等镜像，可切回官方源 `pnpm config set registry https://registry.npmjs.org/`），然后指定版本号重装（见上「版本提示」）。

- **首次安装提示 `ERR_PNPM_IGNORED_BUILDS`**：pnpm 拒绝依赖的构建脚本，按提示把相关包加入 profile 的 `pnpm-workspace.yaml` `allowBuilds` 后重新执行即可。

- **设置里没有「会话管理」入口**：确认已重启 `dsh web`；再确认 `$DSH_HOME/profiles/web/cordis.patch.yml` 中存在 `id: dsh-session-management` 的 insert 条目。

## 工作原理与边界

- **归档**：基于官方 `workspaceRegistry.archiveSession`，归档集持久化于 workspace 域，客户端经官方帧机制自动同步；**取消归档**为官方未提供的操作，插件直接更新 workspace 域归档集合，变化经官方 `domain/changed` 事件自动广播。
- **删除即真删**：定位会话日志文件（含 zstd 对偶文件）后经系统命令删除，随后清理工作区记账与归档标记；搜索索引由官方 SQLite 自动 reconcile 清理。
- **边界说明**：
  - 运行中的会话拒绝删除（避免日志被重新写回）；
  - 聊天中的图片附件采用 content-addressed 存储、可能被多会话共享，删除会话不会连带删除附件；
  - 子代理会话为独立记录，删除父会话不级联删除（可单独删除）。

## License

[MIT](LICENSE) © cokiscarazo-rgb
