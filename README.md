# 文档链接检查

本仓库使用 [Lychee](https://lychee.cli.rs/) 自动检查 Markdown 和 HTML 中的链接，
及时发现失效的公开网址及缺失的相对文件路径。初始检查对象为本 README 中的真实公开链接。

## 自动运行

[链接检查工作流](.github/workflows/links.yml) 在以下情况运行：

- 创建或更新 Pull Request；
- 推送到 `main`；
- 每周一 02:17 UTC（北京时间 10:17）；
- 在 GitHub 仓库的 **Actions → Links → Run workflow** 手动触发
  （工作流首次合并到默认分支后可用）。

失败会使检查变红；在 Actions 的 **Lychee** 步骤日志和运行摘要中查看失败网址及原因。
没有找到任何链接也会失败，避免“空检查”误报成功。GitHub 的定时任务可能延迟。

## 本地运行

从 [Lychee 官方发布页](https://github.com/lycheeverse/lychee/releases/tag/lychee-v0.24.2)
安装 **v0.24.2**，在仓库根目录运行与 CI 相同的命令：

```sh
lychee --version
lychee --config lychee.toml --verbose --no-progress './**/*.md' './**/*.html'
```

也可通过 Rust 工具链安装固定版本：

```sh
cargo install lychee --version 0.24.2 --locked
```

非零退出码表示检查失败。CI 额外启用 `failIfEmpty`，因此不会把无链接输入视为成功。

## 处理失效链接

1. 根据日志确认网址或相对路径，修正拼写、替换为有效地址，或删除过时引用。
2. 超时或限流时先确认站点状态，再重新运行；默认仅有限重试，不将 403、429 或其他错误统一当作成功。
3. 对需要登录、长期阻止自动访问等确实无法检查的地址，才在
   [配置文件](lychee.toml) 的 `exclude` 正则列表中添加尽可能精确的规则，
   并用注释说明原因；不要屏蔽整个域名或大类错误。
4. 修改后重新运行上述命令并提交 PR。

## 安全与维护

工作流仅授予 `contents: read`，不创建 Issue、不写入仓库，检出后不保留 Git 凭据；
使用 GitHub 自动提供的只读令牌，无需配置个人令牌或大模型密钥。
不使用 `pull_request_target`。Action 固定到完整提交 SHA，Lychee 固定到版本；
升级时同步更新工作流及此处说明，并重新验证。

配置限制并发与超时，跳过邮件地址和私有网络目标，不启用跨运行缓存，以免旧结果掩盖失效链接。
链接检查不是安全扫描，也不能保证目标网页内容正确；私有地址过滤并非完整的网络隔离。
