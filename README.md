# ARL-NPoC (xing)

一个集漏洞验证、协议识别、弱口令爆破和漏洞利用于一体的安全测试框架。

> 本项目是 [ARL 资产侦察灯塔系统](https://github.com/TophantTechnology/ARL) 的子模块

## 功能特性

| 功能 | 命令 | 说明 |
|------|------|------|
| POC 扫描 | `xing scan` | 运行漏洞验证插件 |
| 协议识别 | `xing sniffer` | 识别目标端口服务类型 |
| 弱口令爆破 | `xing brute` | 支持 20+ 种服务的密码爆破 |
| 漏洞利用 | `xing exploit` | 执行命令进行漏洞利用 |
| 监听服务 | `xing listener` | DNS/HTTP 反连监听器 |

### 支持的爆破服务

SSH, FTP, MySQL, PostgreSQL, MongoDB, Redis, SQLServer, RDP, SMTP, POP3, IMAP, Tomcat, Jenkins, Gitlab, Grafana, Harbor, Nacos, ActiveMQ, Nexus, APISIX, ClickHouse, Exchange, Shiro 等

## 环境要求

- Python >= 3.10
- [ncrack](https://nmap.org/ncrack/) (可选, 用于部分爆破插件)

## 安装

**使用 uv (推荐)**

```bash
# 克隆仓库
git clone https://github.com/phpmac/ARL-NPoC.git
cd ARL-NPoC

# 安装
uv pip install -e .
```

**作为项目依赖引用**

```toml
# pyproject.toml
dependencies = [
    "xing @ git+https://github.com/phpmac/ARL-NPoC.git",
]
```

**使用 pip**

```bash
pip install -e .
```

## 快速开始

```bash
# 查看帮助
xing -h

# 列出所有插件
xing list -n "*"

# POC 扫描单个目标
xing scan -t http://target.com -n "*"

# POC 扫描多个目标 (从文件读取)
xing scan -t targets.txt -n "*" -c 20

# 协议识别
xing sniffer -t 192.168.1.1:22 -n "*"

# 弱口令爆破
xing brute -t http://target.com -n "*SSH*"

# 使用自定义字典爆破
xing brute -t http://target.com -n "*MySQL*" -U users.txt -P passwords.txt

# 漏洞利用
xing exploit -t http://target.com -n "PluginName" -c "whoami"
```

## 命令参数

### 通用参数

| 参数 | 说明 |
|------|------|
| `-h, --help` | 显示帮助信息 |
| `-V, --version` | 显示版本号 |
| `-q, --quit` | 安静模式 |
| `-L, --log` | 日志等级: debug, info, success, warning, error |

### scan / sniffer / brute 子命令

| 参数 | 说明 |
|------|------|
| `-t, --target` | 目标 URL 或目标文件路径 |
| `-n, --plugin-name` | 插件名称, 支持通配符 (*) |
| `-c` | 并发数量 (默认: 10) |
| `-x, --proxy` | 代理地址 |

### brute 子命令额外参数

| 参数 | 说明 |
|------|------|
| `-U, --username-file` | 自定义用户名字典 |
| `-P, --password-file` | 自定义密码字典 |

### exploit 子命令

| 参数 | 说明 |
|------|------|
| `-c, --cmd` | 需要执行的命令 |
| `-o, --option` | 额外参数 (HTTP GET 格式) |

## 项目结构

```
xing/
├── core/                # 核心框架
│   ├── BasePlugin.py    # 插件基类
│   ├── BaseThread.py    # 并发线程管理
│   ├── PluginRunner.py  # 插件运行器
│   └── BruteRunner.py   # 爆破任务运行器
├── plugins/             # 插件目录
│   ├── poc/             # 漏洞验证插件
│   ├── sniffer/         # 协议识别插件 (28个)
│   ├── brute/           # 爆破插件 (29个)
│   ├── identify/        # 应用识别插件
│   ├── noauth/          # 未授权访问检测
│   └── listener/        # 监听器插件
├── dicts/               # 内置字典
└── utils/               # 工具函数
```

## 开发插件

### POC 插件

```python
from xing.core.BasePlugin import BasePlugin
from xing.core import PluginType

class Plugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.plugin_type = PluginType.POC
        self.vul_name = "CVE-XXXX-XXXX 漏洞名称"
        self.app_name = "目标应用"
        self.scheme = ["http", "https"]

    def verify(self, target):
        # 验证漏洞是否存在
        # 返回 True/字符串表示存在, None/False 表示不存在
        response = requests.get(f"{target}/vulnerable/path")
        if "vulnerable_flag" in response.text:
            return True
        return None
```

### 爆破插件

```python
from xing.core.BasePlugin import BasePlugin
from xing.core import PluginType

class Plugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.plugin_type = PluginType.BRUTE
        self.app_name = "ServiceName"
        self.scheme = ["http", "https"]
        self.username_file = "username_service.txt"
        self.password_file = "password_service.txt"

    def check_app(self, target):
        # 检查目标是否为该服务
        return True

    def login(self, target, user, passwd):
        # 尝试登录, 返回 True 表示成功
        return False
```

## 配置

复制配置文件模板:

```bash
cp xing/config-example.yml xing/config.yml
```

## 许可证

MIT License