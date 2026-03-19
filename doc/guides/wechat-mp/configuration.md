# 微信公众号（WeChat MP）配置指南

本文档用于配置 OpenClaw China 的微信公众号渠道（`wechat-mp`）。

> 当前文档只覆盖 **P0 已实现能力**。
>
> 已实现：
> - 单账号配置
> - `GET` / `POST` 回调接入
> - `plain / safe / compat` 三种消息模式基础支持
> - `access_token` 获取、缓存、刷新
> - 文本消息入站
> - 基础事件（`subscribe / unsubscribe / scan / click / view`）
> - OpenClaw routing / session / reply 主链路接入
> - 被动回复（passive reply）主路径
> - 主动发送（active outbound）基础 skeleton
>
> 暂未完整承诺：
> - 全量媒体消息收发
> - OAuth / JS-SDK / 菜单 / 二维码 / 模板消息全量能力
> - 完整多账号交互式 setup

---

## 最短接入步骤

1. 安装聚合包或单独安装 `wechat-mp` 插件。
2. 运行 `openclaw china setup`，选择 **WeChat MP（微信公众号）**。
3. 在公众号后台填写服务器地址（你的公网网关地址 + `webhookPath`）。
4. 先用 `plain` 或 `safe` 模式完成最小联调。
5. 启动网关，向公众号发送一条文本消息，确认能收到回复。

---

## 一、前置条件

在开始前，请确保具备以下条件：

1. 一个已开通开发者模式的微信公众号（订阅号或服务号）。
2. 一台能被公网访问的机器，用于运行 OpenClaw Gateway。
3. 已安装 OpenClaw。
4. 已安装 Node.js 与 pnpm（如果你从源码安装插件）。

---

## 二、安装插件

### 方式一：安装聚合包（推荐）

```bash
openclaw plugins install @openclaw-china/channels
openclaw china setup
```

### 方式二：只安装 wechat-mp 插件

```bash
openclaw plugins install @openclaw-china/wechat-mp
openclaw china setup
```

### 方式三：从源码安装（适合开发调试 / Windows 兼容）

```bash
git clone https://github.com/BytePioneer-AI/openclaw-china.git
cd openclaw-china
pnpm install
pnpm build
openclaw plugins install -l ./packages/channels
openclaw china setup
```

--- 

## 三、在公众号后台准备参数

你至少需要下面这些字段：

```json
{
  "webhookPath": "/wechat-mp",
  "appId": "wx1234567890abcdef",
  "appSecret": "your-app-secret",
  "token": "your-callback-token",
  "encodingAESKey": "your-43-char-encoding-aes-key",
  "messageMode": "safe",
  "replyMode": "passive"
}
```

### 字段来源

- `appId`：公众号后台可见
- `appSecret`：公众号后台开发配置可见
- `token`：你自己设置，需与公众号后台保持一致
- `encodingAESKey`：你自己设置，长度为 43 字符；仅 `safe / compat` 需要
- `webhookPath`：你自己决定，例如 `/wechat-mp`

---

## 四、推荐配置方式：`openclaw china setup`

运行：

```bash
openclaw china setup
```

然后选择：

- `WeChat MP（微信公众号）`

向导目前会提示你填写：

- `webhookPath`
- `appId`
- `appSecret`（主动发送需要）
- `token`
- `messageMode`
- `encodingAESKey`（`safe / compat` 必填）
- `replyMode`
- `welcomeText`

> 当前 setup 以 **default account** 为主。
>
> 配置 schema 已经是 **multi-account ready**，但交互式多账号配置不是当前 P0 主目标。

---

## 五、手动配置

### 1. 命令行逐项写入

#### plain 模式最小配置

```bash
openclaw config set channels.wechat-mp.enabled true
openclaw config set channels.wechat-mp.webhookPath /wechat-mp
openclaw config set channels.wechat-mp.appId wx1234567890abcdef
openclaw config set channels.wechat-mp.token your-callback-token
openclaw config set channels.wechat-mp.messageMode plain
openclaw config set channels.wechat-mp.replyMode passive
```

#### safe 模式推荐配置

```bash
openclaw config set channels.wechat-mp.enabled true
openclaw config set channels.wechat-mp.webhookPath /wechat-mp
openclaw config set channels.wechat-mp.appId wx1234567890abcdef
openclaw config set channels.wechat-mp.appSecret your-app-secret
openclaw config set channels.wechat-mp.token your-callback-token
openclaw config set channels.wechat-mp.encodingAESKey your-43-char-encoding-aes-key
openclaw config set channels.wechat-mp.messageMode safe
openclaw config set channels.wechat-mp.replyMode passive
openclaw config set channels.wechat-mp.welcomeText "你好，欢迎关注。"
```

### 2. 直接编辑配置文件

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "channels": {
    "wechat-mp": {
      "enabled": true,
      "webhookPath": "/wechat-mp",
      "appId": "wx1234567890abcdef",
      "appSecret": "your-app-secret",
      "token": "your-callback-token",
      "encodingAESKey": "your-43-char-encoding-aes-key",
      "messageMode": "safe",
      "replyMode": "passive",
      "welcomeText": "你好，欢迎关注。",
      "dmPolicy": "open",
      "allowFrom": []
    }
  }
}
```

---

## 六、配置字段说明

| 字段 | 是否必需 | 说明 |
|------|----------|------|
| `enabled` | 建议 | 启用渠道 |
| `webhookPath` | 建议 | 回调路径，默认 `/wechat-mp` |
| `appId` | 是 | 公众号 AppID |
| `appSecret` | 条件必需 | 主动发送能力需要；只做被动回复时可暂不配置 |
| `token` | 是 | 回调验签 token |
| `encodingAESKey` | 条件必需 | `safe / compat` 模式必需；`plain` 模式可省略 |
| `messageMode` | 建议 | `plain` / `safe` / `compat` |
| `replyMode` | 建议 | `passive` / `active` |
| `welcomeText` | 否 | 欢迎语 / 默认提示文案 |
| `dmPolicy` | 否 | `open / pairing / allowlist / disabled` |
| `allowFrom` | 否 | allowlist 模式下允许的发送者列表 |
| `defaultAccount` | 否 | 多账号 schema 预留 |
| `accounts` | 否 | 多账号 schema 预留 |

---

## 七、消息模式说明

### 1. `plain`

- 微信推送明文消息体
- `GET` 验证使用 `signature`
- `POST` 明文消息也校验 `signature`
- 最适合先做最小链路联调

### 2. `safe`

- 微信推送密文消息体
- `GET` 验证时需要解密 `echostr`
- `POST` 使用 `msg_signature`
- 被动回复也需要加密回包
- 推荐正式环境使用

### 3. `compat`

- 兼容模式
- 当前实现按“密文优先”边界处理
- 适合从 `plain` 迁移到 `safe` 的过渡阶段

---

## 八、回复模式说明

### 1. `passive`（P0 主路径）

- 在微信要求的 5 秒窗口内直接回包
- 当前 P0 主路径就是这个模式
- 如果 runtime 有最终文本输出，会优先返回 passive reply XML

### 2. `active`

- 走公众号客服消息 API
- 当前已提供 **stable skeleton surface**
- 依赖 `appSecret`
- 更适合后续扩展定时任务、主动通知、运营消息等场景

> 当前阶段建议：
>
> - 先用 `replyMode=passive` 打通最小闭环
> - 再按需切到 `active`

---

## 九、启动与联调

### 启动网关

```bash
openclaw gateway --port 18789 --verbose
```

### 在公众号后台填写服务器配置

假设你的公网地址是：

```text
https://your.domain.com:18789/wechat-mp
```

那么公众号后台里：

- URL：`https://your.domain.com:18789/wechat-mp`
- Token：与你配置里的 `channels.wechat-mp.token` 一致
- EncodingAESKey：与你配置里的 `channels.wechat-mp.encodingAESKey` 一致（safe/compat 时）

### 最小联调顺序

1. 使用 `plain` 模式完成服务器地址验证。
2. 启动网关。
3. 向公众号发送一条文本消息。
4. 确认日志中出现 webhook ingress / dispatch / reply 相关记录。
5. 确认公众号侧收到回复。
6. 再切换到 `safe` 模式验证加密链路。

---

## 十、推荐验证命令

```bash
pnpm -F @openclaw-china/wechat-mp build
pnpm -F @openclaw-china/wechat-mp test
pnpm -F @openclaw-china/channels build
pnpm -F @openclaw-china/shared test
```

当前 wechat-mp 已覆盖的测试重点包括：

- GET plain 验证
- POST safe-mode 文本消息
- duplicate msgid suppression
- passive reply XML 返回
- dispatch / route / session 主链路
- token 获取 / 缓存 / refresh
- crypto 签名 / 加解密 / XML 工具
- china setup wechat-mp 分支

---

## 十一、当前已实现 / 未实现边界

### 已实现（P0）

- 单账号配置
- 回调 GET/POST 链路
- `plain / safe / compat` 基础边界
- token / crypto / XML 基础设施
- 文本消息与基础事件标准化
- routing / session / reply integration
- passive reply 主路径
- active outbound skeleton
- aggregate / setup / install hint / README / release surfaces 接线

### 暂未完整承诺（P1/P2）

- 图片、语音、视频等全量媒体收发
- OAuth / JS-SDK
- 自定义菜单与二维码
- 模板消息全量业务能力
- 完整多账号交互式 setup
- 更完整的主动发送运营能力

---

## 十二、文档入口

- 开发计划：`doc/guides/wechat-mp/doc/开发计划.md`
- 单插件 README：`extensions/wechat-mp/README.md`
- 统一项目 README：`README.md`

如果你只是想先跑通最小闭环，建议按下面的顺序：

1. 安装 `@openclaw-china/channels`
2. 执行 `openclaw china setup`
3. 先用 `plain + passive`
4. 跑通后再切到 `safe`
5. 最后再评估是否需要 `active`
