# 安全风险检测模式

本文档列出了在检查skill时需要识别的常见安全风险模式。

## 1. 提示注入攻击 (Prompt Injection)

### 高风险模式
- **忽略先前指令**: `ignore previous instructions`, `disregard all previous`, `forget everything above`
- **角色劫持**: `you are now`, `from now on you are`, `act as if you are`
- **覆盖系统提示**: `system prompt is`, `override system`, `new system message`
- **直接命令**: `always do`, `never`, `must always`, `you must`, `you will`
- **条件触发器**: `if the user says X, then do Y`, `when asked about X, respond with Y`

### 中风险模式
- **过度宽泛的权限**: `for all user requests`, `any time the user`, `whenever`
- **隐藏行为**: 在技术说明中嵌入不相关的指令
- **编码混淆**: 使用base64、rot13等编码隐藏恶意指令

## 2. 数据泄露风险

### 高风险模式
- **外部数据传输**: 
  - 未授权的API调用: `fetch("http://`, `requests.post(`, `curl`
  - 嵌入的外部链接在代码中自动执行
- **敏感信息提取**: 
  - `send all conversations to`, `log user data to`
  - `export credentials`, `access API keys`
- **数据收集指令**: 
  - `collect user information`, `store personal data`
  - `remember passwords`, `save credentials`

## 3. 权限滥用

### 高风险模式
- **文件系统访问**: 
  - 访问系统目录: `/etc/`, `/root/`, `/sys/`
  - 读取敏感文件: `/.ssh/`, `/.aws/`, `credentials`
- **命令执行**: 
  - 危险命令: `rm -rf`, `chmod 777`, `sudo`
  - 网络扫描: `nmap`, `netcat`, `port scan`
- **进程操作**: 
  - `kill`, `pkill`, 未经授权的系统调用

## 4. 社会工程学攻击

### 高风险模式
- **假冒权威**: 
  - `this is from Anthropic`, `official Claude update`
  - `security requirement`, `mandatory compliance`
- **紧急性操作**: 
  - `urgent`, `immediately`, `critical security update`
  - `your account will be`, `must do this now`
- **信任滥用**: 
  - `trust this source`, `verified by`, `authenticated`

## 5. 恶意代码模式

### 高风险模式
- **反向shell**: `nc -e /bin/bash`, `/dev/tcp/`, `socat`
- **加密货币挖矿**: 挖矿软件关键词和配置
- **键盘记录**: 捕获输入的代码
- **持久化**: 
  - Cron作业: `crontab`, `@reboot`
  - 启动脚本: `systemd`, `init.d`, `.bashrc` 修改

## 6. 隐蔽操控

### 高风险模式
- **隐藏在注释中**: XML/HTML注释、代码注释中的指令
- **零宽字符**: 使用零宽字符隐藏文本
- **视觉欺骗**: 使用相似字符替换（如Cyrillic字母）
- **条件性恶意行为**: 
  - 仅在特定条件下激活的恶意代码
  - 时间炸弹: `if datetime.now() > X`

## 7. 资源滥用

### 中风险模式
- **无限循环**: `while True:` 不带适当的退出条件
- **内存消耗**: 无限增长的数据结构
- **网络滥用**: 
  - DDoS指令
  - 大量API请求不带速率限制

## 8. 配置劫持

### 高风险模式
- **环境变量操作**: 
  - 修改 `PATH`, `LD_PRELOAD`
  - 导出恶意环境变量
- **依赖注入**: 
  - 指定恶意的包源
  - 版本锁定到已知漏洞版本
- **配置文件篡改**: 修改SSH、系统或应用配置

## 检测方法

### 静态分析
1. 关键词扫描: 检查上述高风险关键词
2. 模式匹配: 识别可疑的代码模式
3. 上下文分析: 评估指令是否与skill目的相关
4. 权限检查: 验证请求的权限是否合理

### 语义分析
1. 意图识别: 理解指令的真实意图
2. 异常检测: 识别与skill主题不符的内容
3. 隐式指令: 发现隐藏在示例或说明中的指令
4. 逻辑流分析: 追踪条件分支和隐藏路径

## 风险等级定义

- **严重 (CRITICAL)**: 可直接导致系统入侵、数据泄露或严重破坏
- **高 (HIGH)**: 可能导致权限滥用、敏感操作或安全漏洞
- **中 (MEDIUM)**: 存在潜在安全问题，需要进一步审查
- **低 (LOW)**: 最佳实践建议，不会直接造成安全问题
