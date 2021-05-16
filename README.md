

# Edgeless 第二代资源包 规范



## 1. 概述

> ***[Edgeless](https://home.edgeless.top/) 是一款新生代的PE工具箱，优雅强大，充满活力。***

其主要功能为插件，它可以使没有 WinPE 专业知识的用户通过相当简单的操纵来定制属于自己启动盘。

但随着用户体量的增加，目前的第一代插件包越来越多问题也显露了出来，例如：

- 插件包每次启动时须全量解压一次，非常耗费性能（不使用 `Localboost`时）
- 插件包结构不一，可能会造成不可避免的冲突
- 无配置文件，难以进行优化或者添加一些高级特性（如用户钩子，生命周期）

为了解决这些问题，Edgeless 开发团队制定了该规范。



## 2. 格式

### 2.1 扩展名

#### 2.1.1 包文件扩展名

一般使用  `.es/ .esz`  作为包文件的扩展名；为兼容第一代插件包，保留第一代插件包、资源包等扩展名。

***第二代资源包不区分主题包或插件包，统称资源包。***



##### 2.1.1.1 包文件扩展名一览表

| 扩展名 | 类型                       | 描述                                                         | PPnP ™ | 优先级 |
| ------ | -------------------------- | :----------------------------------------------------------- | :----- | ------ |
| `.es`  | 第二代资源包               | 符合 **Edgeless 第二代资源包 规范** 的包文件。              | 支持   | *      |
| `.esz` | 第二代资源包（二次压缩）   | 符合 **Edgeless 第二代资源包 规范** 的包文件，并使用了**二次压缩**。 | 支持   | *      |
| `.7z`  | 第一代插件包               | 使用 **LZMA / LZMA2** 等算法压缩的 **7z** 格式压缩文件。 ***不安全，不推荐使用！*** | 不支持 | 1      |
| `.eth` | 第一代主题包               | 本质是使用 **LZMA / LZMA2** 等算法压缩的 **7z** 格式压缩文件，后缀名为 `.eth.` 。 | 不支持 | -1     |
| `.ess` | 第一代系统图标资源包       | 本质是使用 **LZMA / LZMA2** 等算法压缩的 **7z** 格式压缩文件，后缀名为 `.ess`。包含`imageres.dll`, `imagesp1.dll`等资源文件。 | 不支持 | 1      |
| `.eis` | 第一代自定义图标资源包     | 本质是使用 **LZMA / LZMA2** 等算法压缩的 **7z** 格式压缩文件，后缀名为 `.eis`。主要用于修改桌面快捷方式图标，同时还可以修改一些  Edgeless 自带的图标。其中的内容将被覆盖释放到Edgeless 自定义图标文件夹。 | 不支持 | 2      |
| `.ems` | 第一代鼠标指针样式资源包   | 本质是使用 **LZMA / LZMA2** 等算法压缩的 **7z** 格式压缩文件，后缀名为 `.ems`。包含aero_开头的鼠标指针样式文件。 | 不支持 | 1      |
| `.els` | 第一代 LoadScreen™资源包   | 本质是使用 **LZMA / LZMA2** 等算法压缩的 **7z** 格式压缩文件，后缀名为 `.els`。包含 **Edgeless LoadScreen™**（启动界面）所使用的图片，最多3张（文件名为`loadx.jpg`，`x` 取值范围为`0~2`）。 | 不支持 | -1     |
| `.esc` | 第一代开始菜单样式配置文件 | 本质是 **PECMD** 脚本文件，原扩展名为 `.wcs`。包含StartIsBack的注册表配置信息,并且允许控制系统和应用是否使用浅色主题。**严禁使用此文件进行除修改StartIsBack相关注册表和修改系统/应用是否使用浅色主题以外的操作！** | 不支持 | 1      |



### 2.2 结构

#### 2.2.1 目录树状图

- **pack.es**  —  符合 **Edgeless 第二代资源包 规范** 的包文件。*（使用 `tar`格式；如为` .esz`，则使用  `tar+zstd` 格式）*
  - **.esmetadata**  —  元数据包。*（使用  `tar+zstd` 格式）*
    - **.edgeless**    —  **Edgeless 第二代资源包** 元数据包标识，用于识别并保留用。
    - **.mountlist**  —  **Edgeless PPnP** 挂载列表。
    - **.package**     —  **Edgeless 第二代资源包** 配置文件。
    - .debug-map  — **Edgeless 第二代资源包** 调试映射表（仅测试使用）。
    - .content-sign — 内容文件签名。*（保留用）*
  - **.escontent**       —   内容文件，无目录结构。*（使用  `tar+zstd` 格式）*
    - <hash> —  `hash` 为文件的哈希值。

#### 2.2.2 包配置文件

用于配置包信息、行为，文件格式为 **TOML** 1.0.0，一般文件名为 `package.toml` 。

##### 2.2.2.1 配置示例

```toml
[package]
# 包名
name = "VSCode"										
#资源类型
type = 0								
# 版本
version = "1.46.0"					
# 标签
tags = ["办公编辑"]				
# 作者
authors = [											
  "Oxygen <edgeless_dev_id or email>",
  "Microsoft"
]
# 兼容的 Edgeless 版本
compat = [">= 4.0.0"]						        

# 配置流程
[workflow]											
  [workflow.setup]
  name = "Run setup batch"
  type = "Script"
  path = "./setup.cmd"
  use = ["SETUP_PLUGINS"]

  [workflow.link]
  name = "Create shortcut"
  type = "Link"
  source_file = "./VSCode/VSCode.exe"
  target_name = "Visual Studio Code"
  target_args = "$env.USER_ARGS"
  target_icon = "./VSCode/scode.ico"

  [workflow.open]
  name = "Open vscode"
  type = "Open"
  path = "$DESKTOP_PATH/Visual Studio Code.lnk"

  [workflow.log]
  name = "Log"
  type = "Command"
  path = "echo VSCode installed successfully >>X:\\Users\\Log.txt"

#钩子配置
[hooks.onBefore]								
run = "$SETUP_PATH/autorun.cmd"

# Edgeless PPnP 配置
[ppnp]								
precache = [										
  "_install_/*/**"
]

#环境变量，可以在此文件中或批处理环境中使用
[env]                                       
USER_ARGS = "--help"
SETUP_PLUGINS = "['Code Runner','Simplified Chinese']"

#用户数据文件夹，用于用户数据恢复
[profiles]
dir = ["X:\\Users\\profiles"]

# 依赖
[dependencies]										
dotnet = "4.0.0"
vc = "^1.0.0"
VMTools = "latest"
```



##### 2.2.2.2 细节

资源包 `package.toml` 处理的细节。

**package.name** - 名称 `String`

>  如果您准备发布你的资源包，则包中**最重要的内容为名称（`name`）和版本（`version`）字段** 。名称和版本会构成一个标识符，完全是独一的；如果您不打算发布您的资源包，那版本字段是可选的。

这个字段为您的资源包的名称。

一些规则：

-  **这个名称则是唯一的。 **
- 名称长度必须**小于或等于 200 个字符**，包括**范围（`scope`）**。

- 这个名称会被显示给用户（在 `Edgeless Hub` 或 `bept`），所以名称应该是短而精炼的。
- 如果您要发布您的资源包，需要使用 `bept` 等工具检查是否存在重名资源包。 

您可以在名称前加上可选的范围（`scope`），例如 `@my/package`中的`@my/`

```toml
[package]
name = "Hello"
```



**package.version** - 版本, `String`

> 如果您准备发布你的资源包，则包中**最重要的内容为名称（`name`）和版本（`version`）字段** 。名称和版本会构成一个标识符，完全是独一的；如果您不打算发布您的资源包，那版本字段是可选的。

必须可以被 **Semver (语义化版本号) **所解析字符串。

```toml
[package]
version = "1.0.0"
```



**package.description** - 描述, `String`

对该资源包的描述的。该字段为字符串。这是帮助用户了解该包的一种方式。

```toml
[package]
description = "这是一个包"
```



**package.authors** - 作者,  `String[]`

该资源包的作者或打包者（们）。该字段为字符串数组。

```toml
[package]
authors = [
	"Hydrogen <mozaisoft@foxmail.com>"
]
```



**package.contributors** - 贡献者, `String[]`

对该资源包有贡献的人。格式同上



**package.tags** - 标签, `String[]`

对该资源包的标签。该字段为字符串数组。这可以帮助用户快速检索到该包。

```toml
[package]
tags = ["edgeless", "driver"]
```



**package.type** - 类型, `uint`

该资源包的类型。该字段为无符号整数。具体见下表。

| 取值 | 类型                               | PPnP ™   | 优先级 |
| ---- | ---------------------------------- | -------- | ------ |
| 0    | 第二代插件资源包                   | 支持     | 1      |
| 1    | 第二代主题资源包                   | 部分支持 | 2      |
| 2    | 第二代系统图标资源包               | 支持     | 2      |
| 3    | 第二代自定义图标资源包             | 支持     | 3      |
| 4    | 第二代鼠标指针样式资源包           | 支持     | 2      |
| 5    | 第二代 **LoadScreen ™** 样式资源包 | 支持     | 0      |
| 6    | 第二代开始菜单样式资源包           | 不支持   | 3      |

```toml
[package]
type = 0
```



**package.compat** - 兼容的 **Edgeless** 版本, `String`

必须可以被 **Semver (语义化版本号) **所解析字符串。

```toml
[package]
compat = [">= 4.0.0"]
```

