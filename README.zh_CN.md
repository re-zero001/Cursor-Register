<p align="center">
  <span>
   <a href="https://github.com/JiuZ-Chn/CursorRegister/blob/main/README.md">English</a>  | 
   <a href="https://github.com/JiuZ-Chn/CursorRegister/blob/main/README.zh_CN.md">简体中文</a>
  </span>
<p>

# Cursor 注册机

自动注册Cursor账号并保存邮箱、密码和令牌(token)

大陆的网络环境可能对本项目性能有较大影响，如果注册成功率较低，请考虑使用代理或者使用其他网络。此外本项目提供Github Action以供网络条件不便的用户试用。

## 功能

- 注册Cursor账号并保存账号、密码、令牌(token)到本地
- 注册Cursor账号并上传令牌(token)到Chat-API
- 管理Chat-API中额度不足的Cursor账号（支持删除/禁用）
- 上述功能均支持在Github Action中运行

## 本地运行

### 安装依赖 **(务必使用 `Python >= 3.10`)**

```
pip install -r requirements.txt
```

### 注册账户，并将账号信息保存到csv文件

```
python cursor_register.py register.number=3
```
- `register.number`：您要注册的账户号

### 注册机完整配置 `config/config.yaml`

`config.yaml` 描述了注册机的所有参数。如果需要在注册机中接入IMAP或者Chat- API等服务，请在`config.yaml`中进行配置。

#### 基本注册配置
```
register:
  number: 1
  max_workers: 1
```
- `register.number`: 需要注册的账号数量
- `register.max_workers`: 线程数

#### 临时邮箱配置

默认从临时邮箱接收验证码。

```
register:
  temp_email_server:
    name: Minuteinboxcom
```
- `register.temp_email_server.name`: 使用的临时邮箱服务器

#### IMAP 电子邮件服务器配置

使用IMAP接受邮箱验证码。

一种使用IMAP接口的实践是手动注册Outlook账号后，将Outlook的邮件转发至自己的邮箱，然后利用其IMAP接口处理验证码。注册机支持删除账号后重新注册以获取额度。

```
register:
  email_server:
    name: imap_email_server
    use_custom_address: true
    custom_email_address:
    - email1@outlook.com
    - email2@outlook.com

imap_email_server:
  imap_server: imap.qq.com
  imap_port: 993
  username: username
  password: password
```
- `register.email_server.name`: 使用 IMAP 接收邮件时应为 `imap_email_server`
- `register.email_server.use_custom_address`: 使用 IMAP 接收邮件时应为 `true`
- `register.email_server.custom_email_address`: 邮件地址列表
- `register.imap_email_server`: IMAP 服务器配置

#### 将 Token 上传到 [Chat-API](https://github.com/ai365vip/chat-api)

将 `oneapi` 部分的 `enabled` 设置为 `true`，并替换 `url`、`token`、`channel_url` 的值。

```
oneapi:
  enabled: true
  url: http://localhost:3000
  token: your_oneapi_token
  channel_url: http://localhost:3010
```
- `oneapi.enabled`：将数据上传到 Chat-API 时应为 `true`
- `oneapi.url`：Chat-API 地址
- `oneapi.token`：Chat-API 访问令牌。详见 [OneAPI API](https://github.com/songquanpeng/one-api/blob/main/docs/API.md)
- `oneapi.channel_url`: Cursor-API 反代服务地址，需自行搭建Cursor-API反代服务 [Cursor-To-OpenAI](https://github.com/JiuZ-Chn/Cursor-To-OpenAI)

### 在 [Chat-API](https://github.com/ai365vip/chat-api) 中管理低余额 Cursor 频道

- [ChatAPI] 如果运行过程中出现429报错，需提高GLOBAL_API_RATE_LIMIT值，详见[ChatAPI环境变量](https://github.com/ai365vip/chat-api?tab=readme-ov-file#%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)

```
python tokenManager/oneapi_cursor_cleaner.py --oneapi_url {oneapi_url} --oneapi_token {oneapi_token} --disable_low_balance_accounts {disable_low_balance_accounts} --delete_low_balance_accounts {delete_low_balance_accounts}
```
- `oneapi_url`: Chat-API 地址
- `oneapi_token`: Chat-API 访问令牌。详见 [OneAPI API](https://github.com/songquanpeng/one-api/blob/main/docs/API.md)
- `disable_low_balance_accounts`: `True` 或 `False`，禁用Chat-API中的低额度账号
- `delete_low_balance_accounts`: `True` 或 `False`，删除Chat-API中的低额度账号

## 在 Github Action 中运行 Register

GitHub Action适用于不便在本地搭建环境或本地环境不佳的用户以供试用。

Github Action Pipeline **`Cursor Register`** 提供以下参数：
- `number`：需要注册的账号数量
- `max_workers`：线程池的并行度。建议在 Github Action 环境中使用 `1`
- `email_server`：支持 `TempEmail`、`IMAP`。如果使用`IMAP`，需要设置对应的secret
- `Ingest account tokens to OneAPI`: 选中此项，以开启Chat-API服务。如果使用，需要设置对应的secret
- `Upload account infos to artifact`: 选中慈祥，则数据也会被上传到工作流程构件(GitHub Artifacts)，如果不选则跳过该步骤

在仓库中添加机密(secrets)，请参考 [Github 安全指南 - 为存储库创建机密](https://docs.github.com/zh/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)

### 注册帐户，并从 Github Artifact 下载账户信息和 token

适用于手动导入账户令牌(token)或Chat-API没有公网ip的用户。账号注册完成后需手动从工作流程构件(GitHub Artifacts)中下载账号信息。
**务必在下载完成后删除网页中的工作流程构件(GitHub Artifacts)以避免数据泄漏**

使用以下参数运行 **`Cursor Register`**
- `Upload account infos to artifact`: 应为 `☑`

### 注册帐户，并将帐户 cookie 令牌上传到 [Chat-API](https://github.com/ai365vip/chat-api)

需检查已在仓库中添加以下机密(secret)：
- `CURSOR_ONEAPI_URL`: 对应参数 `oneapi_url`
- `CURSOR_ONEAPI_TOKEN`: 对应参数 `oneapi_token`
- `CURSOR_CHANNEL_URL`: 对应参数 `oneapi_channel_url`

使用以下参数运行 **`Cursor Register`**
- `Ingest account tokens to OneAPI`:：应为 `☑`

### 注册帐户，其中使用 IMAP 服务器接收电子邮件

需检查已在仓库中添加以下机密(secret)：
- `CURSOR_IMAP_SERVER`：IMAP 服务器
- `CURSOR_IMAP_PORT`：IMAP 端口
- `CURSOR_IMAP_USERNAME`：IMAP 用户名
- `CURSOR_IMAP_PASSWORD`：IMAP 密码
- `CURSOR_CUSTOM_EMAIL_ADDRESS`：Email地址，以逗号分隔

使用以下参数运行 **`Cursor Register`**
- `email_server`：选择为 `IMAP`

### 管理[Chat-API](https://github.com/ai365vip/chat-api)中额度不足的Cursor账号 

需检查已在仓库中添加以下机密(secret)：
- `CURSOR_ONEAPI_URL`: 对应参数 `oneapi_url`
- `CURSOR_ONEAPI_TOKEN`: 对应参数 `oneapi_token`

使用以下参数运行 **`OneAPI Cursor Cleaner`**
- `Disable Low Balance Accounts`: 是否禁用额度较低的渠道
- `Delete Low Balance Accounts`: 是否删除额度较低的渠道

## 计划
- 一个自动维护One-API中额度的Github Action pipeline

## 致谢
- [Fork source](https://github.com/JiuZ-Chn/Cursor-Register)
- 感谢[cursor-api](https://github.com/Old-Camel/cursor-api/)中的注册机代码思路
