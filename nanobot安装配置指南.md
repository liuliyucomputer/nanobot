# nanobot 安装配置完整指南

## 📋 项目概述

**nanobot** 是一个超轻量级个人AI助手，仅用约4,000行代码实现了核心代理功能，比Clawdbot的43万行代码小99%。

### 🏗️ 核心架构

```
nanobot/
├── agent/          # 🧠 核心代理逻辑
├── skills/         # 🎯 打包技能 (github, weather, tmux...)
├── channels/       # 📱 WhatsApp集成
├── bus/            # 🚌 消息路由
├── cron/           # ⏰ 定时任务
├── heartbeat/      # 💓 主动唤醒
├── providers/      # 🤖 LLM提供商 (OpenRouter等)
├── session/        # 💬 对话会话
├── config/         # ⚙️ 配置
└── cli/            # 🖥️ 命令行界面
```

## 🔧 安装步骤

### 1. 系统环境准备

#### 检查 Python 版本
```bash
python3 --version
# 需要 Python 3.11+
```

#### 安装系统依赖
```bash
# 更新包管理器
apt update

# 安装 pip
apt install -y python3-pip

# 安装 venv 支持
apt install -y python3.12-venv
```

### 2. 创建虚拟环境

```bash
# 进入项目目录
cd /root/workspace/nanobot

# 创建虚拟环境
python3 -m venv venv

# 激活虚拟环境
source venv/bin/activate
```

### 3. 安装 nanobot

```bash
# 使用虚拟环境的 pip 安装
./venv/bin/pip install -e .
```

### 4. 初始化配置

```bash
# 初始化 nanobot 配置
./venv/bin/nanobot onboard
```

这会创建：
- 配置文件：`/root/.nanobot/config.json`
- 工作空间：`/root/.nanobot/workspace`
- 记忆文件：`/root/.nanobot/memory/MEMORY.md`

## 🌐 本地模型配置 (LM Studio)

### 1. LM Studio 设置

1. 启动 LM Studio
2. 加载模型（如：`openai/gpt-oss-20b:2`）
3. 进入 **Settings → Server**
4. 设置 **Host** 为 `0.0.0.0`（重要！）
5. 确认 **Port** 为 `1234`
6. 启动服务器

### 2. 获取宿主机IP

在宿主机上运行：
```bash
# Windows
ipconfig

# macOS/Linux  
ifconfig
# 或
ip addr show
```

记录下宿主机的IP地址（例如：`192.168.169.1`）

### 3. 配置 nanobot

编辑 `/root/.nanobot/config.json`：

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.nanobot/workspace",
      "model": "gpt-oss-20b:2",
      "maxTokens": 8192,
      "temperature": 0.7,
      "maxToolIterations": 20
    }
  },
  "providers": {
    "vllm": {
      "apiKey": "sk-local-key",
      "apiBase": "http://192.168.169.1:1234/v1"
    }
  }
}
```

**关键配置说明：**
- `model`: 使用 LM Studio 中的模型名（去掉 `openai/` 前缀）
- `apiBase`: 宿主机IP + 端口 + `/v1`
- `provider`: 使用 `vllm` 而不是 `openai`

## 🧪 测试连接

### 1. 检查配置状态

```bash
./venv/bin/nanobot status
```

应该显示：
```
🐈 nanobot Status

Config: /root/.nanobot/config.json ✓
Workspace: /root/.nanobot/workspace ✓
Model: gpt-oss-20b:2
vLLM/Local: ✓ http://192.168.169.1:1234/v1
```

### 2. 测试 API 连接

```bash
curl http://192.168.169.1:1234/v1/models
```

应该返回模型列表。

### 3. 测试对话

```bash
# 单次对话
./venv/bin/nanobot agent -m "你好，请简单介绍一下你自己"

# 交互模式
./venv/bin/nanobot agent
```

## 🚀 常用命令

```bash
# 查看状态
./venv/bin/nanobot status

# 单次对话
./venv/bin/nanobot agent -m "你的问题"

# 交互模式
./venv/bin/nanobot agent

# 启动网关（用于 Telegram/WhatsApp）
./venv/bin/nanobot gateway

# 重新初始化
./venv/bin/nanobot onboard
```

## 🔧 故障排除

### 问题1：连接失败
**症状**: `Failed to connect to 127.0.0.1 port 1234`

**解决方案**:
1. 确认 LM Studio 服务器运行中
2. 确认 LM Studio Host 设置为 `0.0.0.0`
3. 检查宿主机IP地址是否正确
4. 确认防火墙设置

### 问题2：模型前缀错误
**症状**: `Error calling LLM: litellm.InternalServerError`

**解决方案**:
1. 使用 `vllm` 提供商而不是 `openai`
2. 模型名去掉 `openai/` 前缀
3. 确认配置文件中没有重复的键

### 问题3：超时问题
**症状**: 命令执行无响应

**解决方案**:
1. 使用 `timeout` 命令限制执行时间
2. 检查网络连接
3. 确认 LM Studio 模型已完全加载

## 📱 可选配置

### Telegram 集成

1. 创建 Telegram Bot：
   - 联系 `@BotFather`
   - 发送 `/newbot`
   - 获取 token

2. 配置文件添加：
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

3. 启动网关：
```bash
./venv/bin/nanobot gateway
```

### WhatsApp 集成

1. 链接设备：
```bash
./venv/bin/nanobot channels login
```

2. 配置文件添加：
```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+1234567890"]
    }
  }
}
```

## 📊 项目信息

- **版本**: v0.1.3.post4
- **Python版本**: >=3.11
- **许可证**: MIT
- **代码量**: ~4,000行
- **GitHub**: https://github.com/HKUDS/nanobot

## 🛣️ 发展路线图

- [x] 语音转录 (Groq Whisper)
- [ ] 多模态支持 (图像、语音、视频)
- [ ] 长期记忆
- [ ] 更好的推理能力
- [ ] 更多集成 (Discord, Slack, email, calendar)
- [ ] 自我改进能力

## 🤝 社区支持

- **Discord**: https://discord.gg/MnCvHqpUGB
- **Feishu**: 见 COMMUNICATION.md
- **WeChat**: 见 COMMUNICATION.md

---

**安装完成后，nanobot 将为你提供强大的 AI 助手功能，支持本地模型运行，保护隐私，响应迅速！** 🐈
