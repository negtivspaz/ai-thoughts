# Hermes connects Nvidia Nemotron Model via OpenRouter for Free

## Background

- Installed Hermes
- Wanted to test a few free models

## Takeaways

- You can set OpenRouter with a primary model and fallback models
- Setup should be almost the same if you use OpenClaw 🦞

![hermes-tui](./img/260422_hermes_nvidia_main.png)

## Register OpenRouter

- Create an account and obtain `OPENROUTER_API_KEY`. Run the following command, the key will be saved in `~/.hermes/.env`
- Create an account and get your `OPENROUTER_API_KEY`. Run this command and the key will be saved in `~/.hermes/.env`.

  ```sh
  hermes config set OPENROUTER_API_KEY sk-or-v1-...
  ```

- OpenRouter has a lot of free models your agent can use. Some are not strong enough for production, but they are great for testing your setup. A few free ones are actually pretty capable, like Nvidia Nemotron and Google Gemma.

- You can also pick paid models directly from providers or through OpenRouter.

- Right now, there are no free image/video generation models on OpenRouter. Hopefully that changes later.

- Limitation: free models may hit rate limits and get lower priority during busy times.

A quick comparison chart is attached to show differences between Nvidia Nemotron, Google Gemma, and Claude's current flagship model, Opus 4.7.

![modelCompare](./img/260422_hermes_nvidia.png)

There is also a short explanation of reasoning vs completion at the end of this article.

## In Hermes

#### Backup your current config.yaml

```sh
cp ~/.hermes/config.yaml ~/.hermes/config.yaml.backup
```

#### Show current model configuration

```sh
hermes config show
```

You should see any other configured model similar to this
You should see a configured model similar to this:
```
◆ Model
  Model:        {'default': 'elephant-alpha', 'provider': 'openrouter', 'base_url': 'https://openrouter.ai/api/v1', 'api_mode': 'chat_completions'}
  Max turns:    90
```

#### Set the primary model

```sh
hermes config set model.provider openrouter
hermes config set model "nvidia/nemotron-3-super-120b-a12b:free"

✓ Set model = nvidia/nemotron-3-super-120b-a12b:free in /home/jeff/.hermes/config.yaml
```

#### Configure fallback models

```sh
hermes config edit
```

Then edit it manually:
```yaml
model:
  default: nvidia/nemotron-3-super-120b-a12b:free
  provider: openrouter
  base_url: https://openrouter.ai/api/v1
  api_mode: chat_completions
providers: {}
fallback_providers:
- provider: openrouter
  model: openrouter/free
- provider: openrouter
  model: openrouter/elephant-alpha
```


> Note: `openrouter/free` provides best available free models from OpenRouter

Restart gateway
```sh
hermes gateway restart
```

#### Other free available models

You can also browse free and paid models from OpenRouter and NousResearch here:

https://openrouter.ai/models?q=free
https://portal.nousresearch.com/models

#### Free usage limit

When using free model via OpenRouter, you may receive a warning:

```txt
API call failed after 3 retries: HTTP 429: Rate limit exceeded: free-models-per-day. Add 10 credits to unlock 1000 free model requests per day
```

This message comes from OpenRouter.ai. You can add 10 credits to avoid this message without being charged.


## Difference between Reasoning and Completion

The main difference is how the model gets to the answer. A **completion** model predicts the most likely next word from learned patterns, while a **reasoning** model works through steps before giving the final response.

Core Definitions
- **Completion** (Pattern Matching): The model predicts the next most likely word (token) from patterns in training data. It usually responds in a single pass, so it is often faster and more fluent, but less deep.
- **Reasoning** (Logical Deliberation): The model goes through an internal thinking phase, breaks a problem into steps, explores options, and checks itself before replying.

---

## 中文翻译（简体）

# Hermes 通过 OpenRouter 免费连接 Nvidia Nemotron 模型

## 背景

- 已安装 Hermes
- 想测试几个免费模型

![hermes-tui](./img/2026-04-23-113217_hyprshot.png)

## 重点

- 你可以在 OpenRouter 里设置一个主模型，再配几个备用模型
- 如果你用 OpenClaw 🦞，配置方式基本也差不多

## 注册 OpenRouter

- 创建账号并获取 `OPENROUTER_API_KEY`。运行下面这个命令，密钥会保存到 `~/.hermes/.env`

  ```sh
  hermes config set OPENROUTER_API_KEY sk-or-v1-...
  ```

- OpenRouter 有很多免费模型可以给你的 agent 用。有些模型不太适合生产环境，但很适合拿来测试配置。也有一些免费模型其实还行，比如 Nvidia Nemotron 和 Google Gemma。

- 你也可以直接从模型提供商，或者通过 OpenRouter，选择付费模型。

- 目前 OpenRouter 还没有免费的图像/视频生成模型。希望后面会有。

- 限制：免费模型在高峰期可能会遇到速率限制，优先级也会更低。

这里附了一张对比图，展示 Nvidia Nemotron、Google Gemma 和 Claude 当前旗舰模型 Opus 4.7 之间的差异。

![modelCompare](./img/260422_hermes_nvidia.png)

文章最后还补充了一个简短说明，讲 reasoning 和 completion 的区别。

## 在 Hermes 里

#### 备份你当前的 config.yaml

```sh
cp ~/.hermes/config.yaml ~/.hermes/config.yaml.backup
```

#### 查看当前模型配置

```sh
hermes config show
```

你应该会看到一个已配置模型，大概像这样：
```
◆ Model
  Model:        {'default': 'elephant-alpha', 'provider': 'openrouter', 'base_url': 'https://openrouter.ai/api/v1', 'api_mode': 'chat_completions'}
  Max turns:    90
```

#### 设置主模型

```sh
hermes config set model "nvidia/nemotron-3-super-120b-a12b:free"

✓ Set model = nvidia/nemotron-3-super-120b-a12b:free in /home/jeff/.hermes/config.yaml
```

#### 配置备用模型

```sh
hermes config edit
```

然后手动改成这样：
```yaml
model:
  default: nvidia/nemotron-3-super-120b-a12b:free
  provider: openrouter
  base_url: https://openrouter.ai/api/v1
  api_mode: chat_completions
providers: {}
fallback_providers:
- provider: openrouter
  model: openrouter/free
- provider: openrouter
  model: openrouter/elephant-alpha
```


> 说明：`openrouter/free` 会提供 OpenRouter 当前可用的最佳免费模型

重启 gateway
```sh
hermes gateway restart
```

#### 其他可用的免费模型

你也可以在下面这两个地方查看 OpenRouter 和 NousResearch 的免费或付费模型：

https://openrouter.ai/models?q=free
https://portal.nousresearch.com/models


## Reasoning 和 Completion 的区别

最主要的区别在于：模型是怎么得到答案的。**completion** 模型会根据学到的模式，预测最可能的下一个词；而 **reasoning** 模型会先分步骤思考，再给出最终回答。

核心定义
- **Completion**（模式匹配）：模型会根据训练数据里的模式，预测最可能的下一个词（token）。通常是一趟直接生成，所以一般更快、更流畅，但深度会弱一些。
- **Reasoning**（逻辑推理）：模型会先进行内部思考，把问题拆成步骤、尝试不同路径，并在回答前做自我检查。

