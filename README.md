# dsh-session-management · DSH 会话管理

中文 | [English](README.en.md)

![会话管理设置页](docs/screenshots/settings.png)

dsh-session-management 是 [DeepSeek Harness](https://github.com/deepseek-ai/dsh)（DSH）Web 的会话管理插件：在「设置」面板内集中管理聊天会话——归档、取消归档、**真正删除本地聊天记录**、导出数据。界面采用克制的 Apple/macOS 风格，支持中英双语（跟随 DSH 语言设置）。

![已归档聊天管理弹窗](docs/screenshots/manage.png)

## 功能

- **已归档的聊天管理**：点击「管理」弹出管理窗口
  - 两种分组视图：**按工作区** / **按月份**（`2026年8月`、`2026年7月`…）
  - 按创建日期 / 更新日期排序，升序 / 降序
  - 分组折叠 / 全部展开 / 全部折叠
  - 按组批量操作：取消归档该组、**删除该组已归档的聊天**（仅作用于该组的归档会话）
- **归档所有聊天**：一键归档全部（保留记录，仅从列表隐藏）
- **删除所有聊天**：真正删除全部本地聊天记录（跳过运行中的会话）
- **导出数据**：与官方一致的 ZIP 导出（会话日志 `session.jsonl` + 子代理 + 媒体附件）
- **中英双语**：跟随「设置 → 通用设置 → 语言」即时切换

## 安装

DSH 插件通过 **profile** 挂载（`dsh web` 对应 `web` profile），安装后需**重启 `dsh web`** 生效。

### 方式一：从 npm 安装（推荐，若已发布）

```sh
dsh plugin --profile web add dsh-session-management
```

### 方式二：从 GitHub 安装

```sh
git clone https://github.com/cokiscarazo-rgb/dsh-session-management.git
cd dsh-session-management

# Windows（PowerShell）
powershell -ExecutionPolicy Bypass -File scripts/install.ps1

# macOS / Linux
bash scripts/install.sh
```

安装脚本会完成两件事（幂等，可重复执行）：

1. 复制插件包到 `$DSH_HOME/profiles/node_modules/dsh-session-management/`
2. 在 `$DSH_HOME/profiles/web/cordis.patch.yml` 注册 loader 条目：

```yaml
- insert:
    - id: dsh-session-management
      name: dsh-session-management
```

### 手动安装

不适用安装脚本时，也可手动完成上述两步：把 `lib/` 与 `package.json` 放进
`$DSH_HOME/profiles/node_modules/dsh-session-management/`，并在
`$DSH_HOME/profiles/web/cordis.patch.yml` 末尾追加上面的 insert 条目。

## 使用

1. 重启 `dsh web` 后，打开 **设置（Settings）**
2. 左侧列表出现「**会话管理**」条目
3. 进入后按需执行：已归档的聊天（管理）/ 归档所有聊天 / 删除所有聊天 / 导出数据

## 卸载

```sh
# 移除安装目录与 patch 条目后重启
rm -rf ~/.dsh/profiles/node_modules/dsh-session-management
```

同时从 `$DSH_HOME/profiles/web/cordis.patch.yml` 中删除对应 insert 条目，重启 `dsh web` 即可。

## 工作原理与边界

- **归档**：使用官方 `workspaceRegistry.archiveSession`，归档集持久化于 workspace 域，客户端经官方帧机制自动同步；**取消归档**为官方未提供的操作，插件直接更新 workspace 域归档集合（官方会经 `domain/changed` 自动广播给客户端）。
- **删除 = 真正删除**：定位会话日志文件（`session.jsonl` / `session.jsonl.zstd`，含对偶压缩文件），经系统命令删除；随后清理 workspace 记账与归档标记；搜索索引由官方 SQLite 自动 reconcile 清理。
- **导出**：复用官方 `/api/session.export` 端点，逐 root 会话下载 ZIP（`dsh-session-<id>.zip`），格式与官方一致。
- **边界说明**：
  - 运行中的会话拒绝删除（避免日志被重新写回）；
  - 聊天中的图片附件（content-addressed 存储，可能被多会话共享）不会随会话删除；
  - 子代理会话为独立记录，删除父会话不会级联删除（可单独删除）。

## License

[MIT](LICENSE) © cokiscarazo-rgb

## 致谢

安装机制与文档结构参考 [dsh-web-ui](https://github.com/zhu1090093659/dsh-web-ui)；界面视觉参考其 [Pinguo/Apple 设计系统](https://github.com/zhu1090093659/dsh-web-ui)。
