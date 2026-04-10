# WechatClawBot（MoviePilot V2 插件）

WechatClawBot 是一个基于 ClawBot/iLink 协议的 MoviePilot V2 微信插件，用于把 MoviePilot 的消息通知和命令交互接入个人微信。

## 插件能做什么

1. 微信扫码登录
- 在插件详情页自动准备登录二维码。
- 支持命令触发生成二维码。
- 扫码成功后自动保存 token 并启动后台轮询。

2. 系统通知转发到微信
- 监听 MoviePilot 的 NoticeMessage 事件并转发。
- 支持按通知类型过滤（高级配置 notify_types）。
- 支持文本、图片、图文、长文本分段发送。

3. 微信命令控制
- 支持插件命令：/wechatclawbot_qrcode
- 支持管理员权限控制（admins 配置）。
- 非插件内置命令会转交 MoviePilot 官方微信链路处理，尽量保持默认行为一致。

4. 命令/列表结果回包增强
- 处理普通消息直接回包。
- 处理媒体候选列表回包（如搜索结果列表）。
- 处理种子候选列表回包（如资源选择列表）。

5. 稳定性与可运维能力
- 轮询、状态查询、发送都带重试逻辑。
- 连续异常会自动判定 token 失效并清理。
- 提供状态、二维码、连接测试、日志查询/清理 API。

## 适用版本

- MoviePilot V2
- 插件目录位于 plugins.v2/wechatclawbot
- 插件清单定义在 package.v2.json（标记 v2: true）

## 运行依赖

- MoviePilot 运行环境可访问 ClawBot/iLink 服务（默认 https://ilinkai.weixin.qq.com）
- 发送图片/图文依赖 pycryptodome（Crypto）
- 二维码图片渲染会尝试外部二维码服务（api.qrserver.com、quickchart.io）

## 安装方式

### 方式一：作为第三方插件仓库接入（推荐）

1. 在 MoviePilot 的插件市场中添加本仓库。
2. 安装 WechatClawBot消息推送。
3. 安装后在插件配置页进行参数配置并启用。

### 方式二：手动部署

1. 将仓库中的 plugins.v2/wechatclawbot 放到 MoviePilot 对应插件目录。
2. 确保 package.v2.json 中包含 WechatClawBot 条目。
3. 重启 MoviePilot 后在插件列表中启用。

## 快速开始

1. 启用插件
- 打开插件配置，打开 enabled 开关并保存。

2. 登录微信
- 进入插件详情页，等待二维码加载。
- 使用微信扫码登录。
- 登录成功后会自动进入连接状态。

3. 验证连接
- 在插件页查看状态是否为已连接。
- 或调用测试接口 /test_connection。

4. 开始使用
- 系统通知会按规则转发到微信。
- 在支持命令的入口发送 /wechatclawbot_qrcode 可重新获取登录二维码或给其他人登录。

## 配置项说明

| 配置项 | 默认值 | 说明 |
|---|---|---|
| enabled | false | 是否启用插件 |
| base_url | https://ilinkai.weixin.qq.com | ClawBot/iLink 服务地址 |
| force_generate_qrcode | false | 保存配置时强制刷新二维码（一次性，执行后自动复位） |
| admins | 空 | 管理员用户 ID，逗号分隔；为空表示不限制（所有用户可执行插件命令） |
| command_enabled | true | 是否允许微信命令控制 |
| notify_enabled | true | 是否启用系统通知转发 |
| notify_types | [] | 通知类型白名单（高级配置，空表示不过滤） |
| poll_timeout | 25 | 长轮询超时秒数 |
| reconnect_delay | 3 | 重连间隔秒数（预留配置） |

说明：
- notify_types 当前属于高级配置项（默认表单未暴露输入框），可通过配置数据手动维护。
- 默认通知目标为最近 24 小时有交互的微信用户；若通知显式指定 userid/targets，则按指定目标发送。

## 命令用法

- /wechatclawbot_qrcode
  - 作用：生成登录二维码并返回登录提示。
  - 常见参数：force/new/refresh（强制刷新二维码）。

## 插件 API（便于排障与联调）

以下为插件暴露的主要接口（插件路径前缀由 MoviePilot 自动注入）：

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | /qrcode | 获取登录二维码信息 |
| GET | /qrcode/image | 获取二维码图片（匿名可访问） |
| GET | /status | 获取登录状态 |
| POST | /logout | 退出登录并清理凭据 |
| GET | /test_connection | 测试连接 |
| GET | /logs | 获取插件日志 |
| POST | /logs/clear | 清空插件日志 |

## 常见问题

1. 二维码不显示或过期
- 二维码有效期较短，等待几秒后刷新插件详情页。
- 可执行 /wechatclawbot_qrcode force 强制刷新。
- 检查 base_url 是否可达。

2. 收不到通知
- 确认 notify_enabled 已开启。
- 若配置了 notify_types，确认通知类型在白名单内。
- 未显式指定用户时，仅会发给最近 24 小时活跃用户。

3. 命令无法执行
- 确认 command_enabled 已开启。
- 若设置了 admins，确认当前 user_id 在管理员列表中。

4. 登录后很快失效
- 插件对连续异常会自动清理失效 token，这是保护机制。
- 建议检查 ClawBot 服务可用性和网络连通性后重新扫码。

## 开发说明

- 核心入口：plugins.v2/wechatclawbot/__init__.py
- iLink 客户端：plugins.v2/wechatclawbot/ilink/client.py
- 插件清单：package.v2.json

## 免责声明

- 本插件依赖第三方 ClawBot/iLink 能力，请遵守相关平台协议与使用规范。
- 请仅在你授权的账号和环境中使用。