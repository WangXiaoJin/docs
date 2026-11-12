# Claude Code

## 1. 安装

安装参考地址：https://code.claude.com/docs/en/overview

> 注意：如果安装成功，但运行 Claude 时出现 “找不到命令 ”或 “未识别 ”错误，说明安装目录不在你的 PATH 里。你的 shell 会在 PATH 目录中搜索程序，
> 安装程序会将 claude 放在 `macOS/Linux` 或 `%USERPROFILE%\.local\bin\claude.exe` Windows 上的 `~/.local/bin/claude`。
> 
> 检查安装目录是否在你的 PATH 中，如果没有则打开系统设置，进入环境变量，在用户路径变量中添加 `%USERPROFILE%\.local\bin`。重启你的终端。

## 2. 配置

在`%USERPROFILE%\.claude\settings.json`中添加以下配置：
```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.ppio.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "sk_xxxxxx",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "pa/claude-haiku-4-5-20251001",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "pa/claude-opus-4-6",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "pa/claude-sonnet-4-5-20250929",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "API_TIMEOUT_MS": "3000000"
  }
}
```

