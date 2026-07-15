# cyp

cyanteam package manager -- 一个轻量的跨平台包管理器。

所有包的安装脚本和配置均为明文（TOML + shell），你可以直接阅读每一个文件，确认它做了什么之后再安装。不需要信任任何人，代码就在那里。

## 它能做什么

cyp 管理软件包的安装、卸载、更新和依赖关系。包格式是普通的 zip 文件，里面包含 TOML 配置和资源文件，没有任何二进制魔法。

- 小包直接从服务器下载，大包从 GitHub 仓库获取
- 支持多架构（macOS ARM/Intel、Linux x64、Windows x64）
- 包可以依赖其他 cyp 包，删除时可以选择级联清理
- 本地安装记录明文存储在 `~/.cyp/` 下

## 系统支持

| 系统 | 架构 | 状态 |
|------|------|------|
| macOS | ARM64 (Apple Silicon) | 支持 |
| macOS | x86_64 (Intel) | 支持 |
| Linux | x86_64 | 支持 |
| Windows | x86_64 | 支持 |

需要 Node.js 18+。

## 安装

macOS / Linux:

```
curl -fsSL https://cyp.cyteam.cc.cd/install.sh | bash
```

Windows PowerShell:

```
Invoke-WebRequest -Uri https://cyp.cyteam.cc.cd/install.ps1 -OutFile install.ps1; .\install.ps1
```

或者用 npm:

```
npm install -g cyp
```

安装完成后把 `~/.cyp/bin` 加入 PATH:

```
# bash
echo 'export PATH="$HOME/.cyp/bin:$PATH"' >> ~/.bashrc

# zsh
echo 'export PATH="$HOME/.cyp/bin:$PATH"' >> ~/.zshrc
```

## 使用

### 安装包

可以用包名或完整的 package_id 安装:

```
$ cyp install chrome
正在从远程仓库查找 "chrome" ...
找到包: Chrome 浏览器，正在下载...
✔ 安装完成: chrome v150.0.7871.124
  运行 cyp run chrome 启动
```

如果名称有重复，需要用完整的 package_id:

```
$ cyp install com.google.chrome
```

也可以从本地文件安装:

```
$ cyp install ./myapp-1.0.0.cyp
$ cyp install ./my-project/          # 本地目录（含 _CYPPACKAGE.TOML）
$ cyp install https://example.com/pkg.cyp
```

### 删除包

同样支持按名称或 package_id:

```
$ cyp remove chrome
正在准备删除 chrome (com.google.chrome) v150.0.7871.124 ...
确认删除 chrome v150.0.7871.124? (y/N) y
✔ 已删除安装目录
✔ 已从注册表移除: chrome (com.google.chrome)
```

### 搜索包

```
$ cyp search chrome
搜索结果:
  包ID               名称     版本            描述
  com.google.chrome  chrome   150.0.7871.124  Google Chrome 网络浏览器

共找到 1 个包
```

### 列出已安装的包

```
$ cyp list
已安装的包:
  com.google.chrome   chrome   150.0.7871.124   Google Chrome 网络浏览器
  com.cyp.helloworld  hello    1.0.0            cyp hello world
共 2 个包
```

### 更新包

```
$ cyp update chrome
```

更新 cyp 自身:

```
$ cyp update --self
```

### 运行包命令

```
$ cyp run chrome
$ cyp run hello
```

### 打包

把一个目录打包成 `.cyp` 文件:

```
$ cyp pack ./my-project/
✔ 打包完成: my-project-1.0.0.cyp (12.3 KB)
```

### 解包

```
$ cyp unpack ./my-project-1.0.0.cyp
✔ 解包完成: ./my-project/
```

### 创建新包

交互式创建包的目录结构:

```
$ cyp create myapp
包名称 [myapp]: myapp
包中文名: 我的应用
版本号 [1.0.0]: 
package_id [com.example.myapp]: com.cyteam.myapp
语言 [node]: 
✔ 已创建包目录: ./myapp/
```

### 查看包信息

```
$ cyp parse ./myapp-1.0.0.cyp
名称:     myapp
中文名:   我的应用
版本:     1.0.0
package_id: com.cyteam.myapp
语言:     node
架构:     any
```

## 包格式

一个 cyp 包就是 zip 文件，扩展名 `.cyp`，里面至少包含:

```
myapp/
  _CYPPACKAGE.TOML      # 包信息
  _CYPREQUIRE.TOML      # 依赖声明（可选）
  _CYPASSETS/            # 包资源文件
    main.js              # 入口文件等
```

### _CYPPACKAGE.TOML

```toml
name = "myapp"
name_cn = "我的应用"
version = "1.0.0"
package_id = "com.example.myapp"
language = "node"
architecture = "any"
description = "一个示例应用"

[config_command]
common = "node _CYPASSETS/main.js"
macosarm64 = "./_CYPASSETS/myapp"
linux64 = "./_CYPASSETS/myapp"
win64 = "_CYPASSETS\\myapp.exe"
```

### _CYPREQUIRE.TOML

```toml
# 系统依赖
[[dependency]]
name = "node"
type = "runtime"
check_command = "node -v"

# cyp 包依赖
[[dependency]]
name = "chrome"
type = "cyp"
cyp_package = "com.google.chrome"
check_command = "cyp list | grep chrome"
```

安装时，cyp 会自动安装 `type = "cyp"` 声明的依赖包。删除时可以选择一并清理。

## 大包和 GitHub 存储

超过 15MB 的包存储在 GitHub 仓库中（如 `cyanteam/cyp-pkg-com-google-chrome`），元数据仍然在 cyp 服务器上。下载时客户端会自动重定向到 GitHub Release。

每个架构有独立的包文件:

```
chrome-150.0.7871.124-macosarm64.cyp
chrome-150.0.7871.124-macosintel.cyp
chrome-150.0.7871.124-linux64.cyp
chrome-150.0.7871.124-win64.cyp
```

客户端安装时会自动选择当前平台对应的文件。

## 为什么要用 cyp

- **明文可审计** -- 所有包的配置和脚本是 TOML 和 shell，不是二进制格式。安装之前你可以读一遍它要做什么。
- **简单** -- 包就是 zip，配置就是 TOML，没有复杂的依赖解析图或 lockfile。
- **跨平台** -- 同一个包格式覆盖 macOS / Linux / Windows。
- **多架构** -- 软件包支持按平台分发不同的二进制文件。
- **依赖管理** -- 包可以依赖其他 cyp 包，安装和删除时自动处理。

## 命令一览

| 命令 | 别名 | 说明 |
|------|------|------|
| `install` | `i`, `inst` | 安装包 |
| `remove` | `rm` | 删除包 |
| `list` | `ls` | 列出已安装的包 |
| `search` | `s` | 搜索远程仓库 |
| `update` | `up` | 更新包或自身 |
| `pack` | `p` | 打包目录为 .cyp |
| `unpack` | `u` | 解包 .cyp 到目录 |
| `create` | `c` | 交互式创建新包 |
| `parse` | `info` | 解析包信息 |
| `run` | `r` | 直接运行包命令 |
| `login` | | 登录 cyp 账号 |

## 相关链接

- 网站: https://cyp.cyteam.cc.cd
- 管理面板: https://cyp.cyteam.cc.cd/admin

## 联系

邮箱: qtof@qq.com
