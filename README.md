# Linux 上使用 VS Code Continue 配置 DeepSeek API

本文档说明如何在一台 Linux 电脑上，为 VS Code 的 Continue 插件配置 DeepSeek API。适用于没有 VPN、只使用中国大陆网络的环境。

## 1. 准备条件

需要提前准备：

- Linux 系统
- VS Code
- DeepSeek API Key
- 可以访问 `api.deepseek.com`

如果 VS Code 扩展市场访问正常，可以直接在线安装 Continue 插件。如果扩展市场下载失败，可以提前准备 Continue 的 `.vsix` 离线安装包。

## 2. 安装 Continue 插件

### 在线安装

打开 VS Code，进入左侧 Extensions 面板，搜索：

```text
Continue
```

确认插件信息：

```text
插件 ID: Continue.continue
发布者: Continue
```

然后点击安装。

也可以使用命令行安装：

```bash
code --install-extension Continue.continue
```

### 离线安装

如果扩展市场无法下载插件，可以使用 `.vsix` 文件离线安装：

```bash
code --install-extension /path/to/continue.vsix
```

也可以在 VS Code 中操作：

```text
Extensions 面板右上角 ... -> Install from VSIX...
```

## 3. 创建 Continue 配置目录

在 Linux 终端执行：

```bash
mkdir -p ~/.continue
```

Continue 的配置文件路径为：

```text
~/.continue/config.yaml
```

## 4. 写入 DeepSeek 配置

创建或编辑配置文件：

```bash
nano ~/.continue/config.yaml
```

写入以下内容。

注意：把 `YOUR_DEEPSEEK_API_KEY` 替换成自己的 DeepSeek API Key。

```yaml
name: DeepSeek Continue
version: 1.0.0
schema: v1

models:
  - name: DeepSeek V4 Flash
    provider: deepseek
    model: deepseek-v4-flash
    apiKey: YOUR_DEEPSEEK_API_KEY
    roles:
      - chat
      - edit
      - apply
      - summarize
    capabilities:
      - tool_use
    defaultCompletionOptions:
      temperature: 0.2
      maxTokens: 4096

  - name: DeepSeek V4 Pro
    provider: deepseek
    model: deepseek-v4-pro
    apiKey: YOUR_DEEPSEEK_API_KEY
    roles:
      - chat
      - edit
      - apply
      - summarize
    capabilities:
      - tool_use
    defaultCompletionOptions:
      temperature: 0.2
      maxTokens: 4096
```

如果只想使用一个模型，可以只保留 `DeepSeek V4 Flash`。日常编程、问答和代码修改一般优先使用 Flash，速度更快、成本更低。

## 5. 重载 VS Code

配置保存后，在 VS Code 中按：

```text
Ctrl + Shift + P
```

执行：

```text
Developer: Reload Window
```

然后打开 Continue 面板，选择：

```text
DeepSeek V4 Flash
```

或：

```text
DeepSeek V4 Pro
```

## 6. 测试 API 连通性

可以在 Continue 面板里直接提问：

```text
你好，介绍一下你自己
```

如果想在终端测试 DeepSeek API 是否可访问，可以执行：

```bash
curl https://api.deepseek.com/models \
  -H "Authorization: Bearer YOUR_DEEPSEEK_API_KEY"
```

如果网络和 API Key 正常，会返回模型列表。

## 7. 常见问题

### Continue 插件安装失败

可以尝试：

- 检查 VS Code 是否可以访问扩展市场
- 换用 `.vsix` 离线安装
- 确认命令行中 `code` 命令可用

### Continue 找不到模型

检查：

- `~/.continue/config.yaml` 路径是否正确
- YAML 缩进是否正确
- `schema: v1` 是否存在
- VS Code 是否已执行 `Developer: Reload Window`

### API 请求失败

检查：

- DeepSeek API Key 是否正确
- 电脑是否能访问 `https://api.deepseek.com`
- API Key 是否还有额度
- 公司、校园或机房网络是否拦截 HTTPS 请求

### 模型没有主动说自己是 V4

模型自我介绍不一定等于实际 API 请求中的模型 ID。Continue 实际调用哪个模型，主要看 `config.yaml` 中的 `model` 字段。

例如：

```yaml
model: deepseek-v4-flash
```

表示 Continue 会按 `deepseek-v4-flash` 这个模型 ID 发起请求。
