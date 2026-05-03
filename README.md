# MoverTool

使用 GitHub Actions 和 rclone 在两个网盘之间复制文件。

工作流通过手动触发运行，输入源路径和目标路径后，会使用 `rclone copy --ignore-existing` 复制目标端缺失的文件，并跳过目标端已经存在的文件。

## 准备工作

1. 在本地配置并测试 rclone。

   ```powershell
   rclone config
   rclone lsd remote:
   ```

2. 找到 rclone 配置文件。

   ```powershell
   rclone config file
   ```

   命令会输出 `rclone.conf` 的路径。

3. 将 `rclone.conf` 编码为 Base64。

   如果你的本机系统是 Windows，请在 PowerShell 中执行，将路径替换为上一步输出的实际路径：

   ```powershell
   [Convert]::ToBase64String([System.IO.File]::ReadAllBytes("C:\Users\YourName\AppData\Roaming\rclone\rclone.conf"))
   ```

   如果你的本机系统是 Linux 或 macOS，请在 Bash 中执行，将路径替换为上一步输出的实际路径：

   ```bash
   base64 -w 0 ~/.config/rclone/rclone.conf
   ```

   复制输出的整段 Base64 字符串。

4. 在 GitHub 仓库中添加 Secret。

   打开 `Settings` -> `Secrets and variables` -> `Actions` -> `New repository secret`，添加：

   - Name: `RCLONE_CONFIG`
   - Secret: 上一步生成的 Base64 字符串

5. 如果你的 rclone 配置启用了密码加密，再额外添加：

   - Name: `RCLONE_CONFIG_PASS`
   - Secret: rclone 配置密码

## 使用方式

1. 打开 GitHub 仓库的 `Actions` 页面。
2. 选择 `Rclone Sync Remotes` 工作流。
3. 点击 `Run workflow`。
4. 填写两个参数：

   - `source_path`: 源路径，例如 `drive1:/folder`
   - `destination_path`: 目标路径，例如 `drive2:/folder`

5. 点击运行。

## 路径示例

```text
source_path: onedrive:/Movies
destination_path: google:/Backup/Movies
```

```text
source_path: google:/Photos/2026
destination_path: dropbox:/Archive/Photos/2026
```

## 行为说明

- 只复制源路径中目标端不存在的文件。
- 目标端已有同名文件时会跳过，不会覆盖。
- 不会删除目标端已有但源路径不存在的文件。
- rclone 配置由 `AnimMouse/setup-rclone@v1` 安装，并从 Base64 编码后的 `RCLONE_CONFIG` secret 注入。
