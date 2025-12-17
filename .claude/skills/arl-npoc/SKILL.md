---
name: arl-npoc
description: ARL-NPoC 是一个集漏洞验证、协议识别、弱口令爆破和漏洞利用于一体的安全测试框架。用于编写和运行安全扫描插件、服务识别和密码爆破任务。
---

# ARL-NPoC 安全测试框架

## 概述

ARL-NPoC (xing) 是 ARL 资产侦察灯塔系统的子模块, 提供漏洞验证和任务运行能力。

## 核心功能

- **POC 扫描** (`scan`): 运行漏洞验证插件
- **协议识别** (`sniffer`): 识别目标端口运行的协议/服务
- **弱口令爆破** (`brute`): 支持多种服务的密码爆破
- **漏洞利用** (`exploit`): 执行命令利用漏洞
- **监听服务** (`listener`): DNS/HTTP 监听器

## 项目结构

```
xing/
├── core/                # 核心框架
│   ├── BasePlugin.py    # 插件基类
│   ├── BaseThread.py    # 线程管理
│   ├── PluginRunner.py  # 插件运行器
│   └── BruteRunner.py   # 爆破运行器
├── plugins/             # 插件目录
│   ├── poc/             # 漏洞验证插件
│   ├── sniffer/         # 协议识别插件
│   ├── brute/           # 爆破插件
│   ├── identify/        # 应用识别插件
│   ├── noauth/          # 未授权访问检测
│   └── listener/        # 监听器插件
├── dicts/               # 字典文件(用户名/密码)
└── utils/               # 工具函数
```

## 命令使用

```bash
# 查看帮助
xing -h

# 列出插件
xing list -n "*"

# POC 扫描
xing scan -t <target_or_file> -n <plugin_name> -c <concurrency>

# 协议识别
xing sniffer -t <target_or_file> -n <plugin_name>

# 弱口令爆破
xing brute -t <target_or_file> -n <plugin_name> -U <user_file> -P <pass_file>

# 漏洞利用
xing exploit -t <target> -n <plugin_name> -c <command>
```

## 插件类型

| 类型 | 说明 | 方法 |
|------|------|------|
| PluginType.POC | 漏洞验证 | `verify(target)` |
| PluginType.SNIFFER | 协议识别 | `sniffer(host, port)` |
| PluginType.BRUTE | 弱口令爆破 | `login(target, user, passwd)` + `check_app(target)` |
| PluginType.LISTENER | 监听服务 | `listen(host, port)` |

## 开发插件

### POC 插件示例

```python
from xing.core.BasePlugin import BasePlugin
from xing.core import PluginType

class Plugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.plugin_type = PluginType.POC
        self.vul_name = "漏洞名称"
        self.app_name = "应用名称"
        self.scheme = ["http", "https"]

    def verify(self, target):
        # 验证漏洞是否存在
        # 返回 True/字符串 表示存在, 返回 None/False 表示不存在
        pass
```

### 爆破插件示例

```python
from xing.core.BasePlugin import BasePlugin
from xing.core import PluginType

class Plugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.plugin_type = PluginType.BRUTE
        self.app_name = "应用名称"
        self.scheme = ["http", "https"]
        self.username_file = "username_xxx.txt"
        self.password_file = "password_xxx.txt"

    def check_app(self, target):
        # 检查是否是目标应用
        return True/False

    def login(self, target, user, passwd):
        # 尝试登录
        # 返回 True 表示成功
        pass
```

### 协议识别插件示例

```python
from xing.core.BasePlugin import BasePlugin
from xing.core import PluginType, SchemeType

class Plugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.plugin_type = PluginType.SNIFFER
        self.target_scheme = SchemeType.SSH
        self.default_port = [22]

    def sniffer(self, host, port):
        # 识别协议
        # 返回协议名称字符串 或 None
        pass
```

## 配置文件

复制 `xing/config-example.yml` 为 `xing/config.yml` 进行配置。

## 依赖

- Python >= 3.10
- ncrack (外部依赖): https://nmap.org/ncrack/

## 安装

```bash
# 使用 uv
uv pip install -e .

# 或作为依赖引用
# xing @ git+https://github.com/phpmac/ARL-NPoC.git
```

## 注意事项

- 目标参数 `-t` 可以是单个 URL 或包含多个目标的文件路径
- 插件名称 `-n` 支持通配符匹配, 如 `*SSH*`
- 使用前请确保已阅读并同意免责声明

