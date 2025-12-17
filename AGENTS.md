# AGENTS.md

ARL-NPoC (xing) 是一个 Python 安全测试框架, 用于漏洞验证、协议识别和弱口令爆破。

## 环境设置

```bash
# 安装依赖 (推荐使用 uv)
uv pip install -e .

# 或使用 pip
pip install -e .

# 验证安装
xing --version
```

## 项目结构

```
xing/
├── core/           # 核心框架 (BasePlugin, PluginRunner, BruteRunner)
├── plugins/        # 插件目录
│   ├── poc/        # 漏洞验证插件
│   ├── sniffer/    # 协议识别插件
│   ├── brute/      # 爆破插件
│   ├── identify/   # 应用识别插件
│   ├── noauth/     # 未授权访问检测
│   └── listener/   # 监听器插件
├── dicts/          # 用户名/密码字典文件
├── utils/          # 工具函数
└── main.py         # 入口点
```

## 代码风格

- Python 3.10+, 使用类型注解
- 类名使用 PascalCase, 函数/变量使用 snake_case
- 插件文件名使用 PascalCase, 如 `SSHBrute.py`
- 注释使用中文, 标点使用英文
- 每个插件类必须命名为 `Plugin`
- 继承 `BasePlugin` 基类

## 插件开发规范

### 插件类型

| 类型 | PluginType | 必须实现的方法 |
|------|------------|----------------|
| POC | `PluginType.POC` | `verify(target)` |
| 协议识别 | `PluginType.SNIFFER` | `sniffer(host, port)` |
| 爆破 | `PluginType.BRUTE` | `check_app(target)`, `login(target, user, passwd)` |
| 监听 | `PluginType.LISTENER` | `listen(host, port)` |

### POC 插件模板

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
        # 返回 True/字符串 表示漏洞存在
        # 返回 None/False 表示不存在
        pass
```

### 爆破插件模板

```python
from xing.core.BasePlugin import BasePlugin
from xing.core import PluginType

class Plugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.plugin_type = PluginType.BRUTE
        self.app_name = "ServiceName"
        self.scheme = ["http", "https"]
        self.username_file = "username_xxx.txt"  # 对应 dicts/ 下的文件
        self.password_file = "password_xxx.txt"

    def check_app(self, target):
        # 检查目标是否为该服务, 返回 bool
        pass

    def login(self, target, user, passwd):
        # 尝试登录, 返回 True 表示成功
        pass
```

## 测试命令

```bash
# 列出所有插件
xing list -n "*"

# 测试单个插件
xing scan -t http://example.com -n "PluginName"

# 调试模式
xing -L debug scan -t http://example.com -n "*"
```

## 添加新字典

1. 用户名字典: `xing/dicts/username_<service>.txt`
2. 密码字典: `xing/dicts/password_<service>.txt`
3. 每行一个条目, UTF-8 编码
4. 密码支持 `%user%` 占位符 (会替换为用户名)

## 重要文件

| 文件 | 说明 |
|------|------|
| `xing/core/BasePlugin.py` | 插件基类, 所有插件必须继承 |
| `xing/core/const.py` | 常量定义 (PluginType, SchemeType) |
| `xing/core/PluginRunner.py` | 插件调度器 |
| `xing/utils/loader.py` | 插件加载器 |
| `xing/conf.py` | 配置管理 |

## 注意事项

- 不要修改 `xing/core/` 下的核心文件, 除非修复 bug
- 新增插件放到对应的 `xing/plugins/<type>/` 目录
- 插件文件名即为插件名 (不含 .py 后缀)
- 确保插件类名为 `Plugin`, 否则无法加载
- HTTP 请求使用 `requests` 库, 支持通过 `Conf.PROXY_URL` 设置代理

