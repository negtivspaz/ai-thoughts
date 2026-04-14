# 在 Hermes 中，配置由 OpenRouter 提供的免費 Elephant-Alpha 模型
### Take Away
- 使用 Hermes 作为 OpenClaw 的即插即用替代品。  
- 通过 OpenRouter 配置免费的 Elephant-Alpha 模型。

![img](img/260414_hermes_openrouter2.png)
### 背景  
- 在使用 OpenClaw 几个月后，我对 Hermes 产生了好奇，并决定尝试一下。  
- 本指南将介绍如何通过 OpenRouter 集成免费的 Elephant-Alpha 模型。

### 注册 OpenRouter 账号  
- 在 [OpenRouter](https://openrouter.ai/) 上创建账号。  
- 获取你的 `API_KEY`。

### 设置 Hermes 并验证状态  
按照官方说明安装 Hermes 并连接你的 Channels ——Telegram、Discord 和 WhatsApp 都是 Hermes 和 OpenClaw 支持的常用集成。  
### 验证安装：

```shell
hermes status
```

将你的 OpenRouter `API_KEY` 保存在：

```shell
~/.hermes/.env
```

需要把这个文件当作**信用卡**一样对待——务必保证安全。

### 在 Hermes 中配置 Elephant-Alpha 模型  
- 运行以下命令来配置 Hermes 并设置主模型：
`hermes model`

- 选择 **OpenRouter**，然后手动输入模型名称 `elephant-alpha`。  
### 验证配置：

`hermes config show`

期望输出：

```shell
◆ Model   Model: {'default': 'elephant-alpha', 'provider': 'openrouter', 'base_url': 'https://openrouter.ai/api/v1', 'api_mode': 'chat_completions'}   Max turns: 90
```
### 测试 Hermes and Enjoy
- 启动 WhatsApp 集成：
`hermes whatsapp`

当出现提示时，选择：“2. 单独的机器人号码（推荐）”  
输入已授权发送消息给机器人的电话号码。Hermes 将生成一个二维码——用手机上的 WhatsApp 扫描它。  
一旦连接成功，在 WhatsApp 中向机器人的号码发送消息。Hermes Agent 将立即回复一个配对码。  
回到 Hermes 终端并批准配对：

`hermes pairing approve whatsapp <PAIRING_CODE>`

- 允许两位以上用户通过 WhatsApp 访问代理  
我目前只有两位用户能够与 Hermes Agent 交互。  
如果其他用户在发送私信（例如配对码）时无法收到消息，请手动更新你的配置：

`whatsapp:   
	unauthorized_dm_behavior: pair  
	require_mention: true`

保存文件于：
`~/.hermes/config.json`

然后重启网关：

`hermes gateway restart`

此更改还可让 Hermes Agent 在 WhatsApp 群组中被 @提及（当机器人被添加后）。  
快乐使用 Hermes——探索、定制并享受其中！