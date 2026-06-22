# 自部署 LLM Endpoint 接入说明

本分支放宽了 OpenLess 对自部署 OpenAI-compatible LLM 的 Endpoint 校验，并补齐了常见本地/内网/公网部署地址的自动归一化。

## 背景

原逻辑要求公网 IP / 普通域名必须使用 `https`，所以这类自部署地址会被设置页判定为：

```text
Endpoint 格式不合法
```

例如：

```text
http://36.147.35.14:30080
http://36.147.35.14:30080/v1/chat/completions
```

很多自部署 vLLM、OneAPI、Nginx NodePort、内网网关在测试阶段只开放 HTTP，因此需要允许用户显式配置 HTTP endpoint。

## 推荐填写

如果你的 LLM 服务真实接口是：

```text
http://36.147.35.14:30080/v1/chat/completions
```

OpenLess 设置页里可以填写：

```text
Endpoint: 36.147.35.14:30080
Model: deepseek_v4
API Key: 网关需要鉴权就填真实 key；不需要就留空
```

也可以填写：

```text
Endpoint: http://36.147.35.14:30080
Endpoint: http://36.147.35.14:30080/v1
Endpoint: http://36.147.35.14:30080/v1/chat/completions
```

代码会自动归一化：

```text
36.147.35.14:30080
→ http://36.147.35.14:30080/v1
→ http://36.147.35.14:30080/v1/chat/completions
```

模型列表会请求：

```text
http://36.147.35.14:30080/v1/models
```

## 安全边界

本分支允许公网 HTTP endpoint，但仍然拒绝：

- 云元数据地址，如 `169.254.169.254`、`metadata.google.internal`
- CGNAT，如 `100.64.0.0/10`
- link-local
- unspecified / broadcast
- 非 `http` / `https` scheme

## 快速验证

```bash
curl http://36.147.35.14:30080/v1/models
```

如果能返回模型列表，再在 OpenLess 设置页里点击“验证”或“拉取模型”。
