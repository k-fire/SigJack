# SigJack

**将 DLL 劫持与 SigFlip 签名完美结合的自动化工具**

一个用于将 PE 文件转换为 Shellcode，并通过 SigFlip 将其注入到具有合法数字签名的可执行文件中，实现 DLL 劫持的 Windows GUI 工具。

大佬们，点点star吧！

![License](https://img.shields.io/badge/license-MIT-blue.svg) ![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg) ![Architecture](https://img.shields.io/badge/arch-x86%20%7C%20x64-green.svg)

## 🎨 界面
<img src="https://raw.githubusercontent.com/k-fire/SigJack/refs/heads/main/SigJack.png" alt="SigJack UI" width="200"/>

---

## ⚠️ 免责声明

**本工具仅供安全研究和教育目的使用。**

- ⚖️ 使用本工具进行**未经授权的攻击活动是违法的**
- 🚫 作者**不对因使用本工具造成的任何损害负责**
- ✅ 使用者需在**合法授权范围内**使用（如：渗透测试授权、安全研究、CTF 竞赛）
- 📋 使用者需**自行承担所有法律责任**
- 🎓 本工具旨在帮助安全研究人员**理解攻击技术**并**开发相应防御措施**
- ⚖️ 使用者需**遵守《中华人民共和国网络安全法》及相关法律法规**

**如果您不同意上述条款，请勿下载、使用或传播本工具。**

---

## ✨ 功能特性

- 🔄 **PE 到 Shellcode 转换**：支持 Donut、PengCode、ZigDonut、PE2SHC 等多种工具
- 🎭 **SigFlip 签名注入**：将 Shellcode 注入到合法签名文件的证书区
- 💉 **DLL 劫持自动化**：生成配套的劫持 DLL，自动提取并执行 Shellcode
- 🎯 **架构自动识别**：拖拽文件后自动检测 x86/x64 架构
- 🏷️ **多种签名模板**：内置 Microsoft、Sangfor 等厂商的合法签名模板
- 🎨 **友好的图形界面**：基于 MFC 的现代化界面
- 🛡️ **权限自动检测**：自动提示需要管理员权限的工具

## 🎯 核心原理

SigJack 通过以下三步实现 **DLL 劫持 + 签名绕过**：

### 1️⃣ PE 转 Shellcode
将 PE 文件转换为位置无关的 Shellcode（.bin 格式）

支持多种转换工具：

- **[Donut](https://github.com/TheWover/donut)**
  生成与位置无关的 x86、x64 或 AMD64+x86 shellcode，该 shellcode 从内存加载 .NET 程序集、PE 文件和其他 Windows 有效负载，并使用参数运行它们。通用型转换器，支持最丰富的配置选项。

- **[PengCode](https://github.com/Mephostophiles/PengCode)**
  专业的 EXE 转 ShellCode 程序，支持包括 Golang 在内的多种编译器生成的可执行文件。

- **[ZigDonut](https://github.com/howmp/zigdonut)**
  用 Zig 实现的精简版 Donut，可将 exe/dll 转换为 shellcode，支持 Rust 静态 TLS。

- **[PE2SHC](https://github.com/hasherezade/pe_to_shellcode)**
  转换 PE 文件，使其可以像普通 shellcode 一样注入。简单快速的转换器。

### 2️⃣ SigFlip 注入
使用 **[SigFlip](https://github.com/med0x2e/SigFlip)** 工具将 Shellcode 注入到合法签名的可执行文件的证书区域：

SigFlip 是一个用于修补 authenticode 签名的 PE 文件（exe、dll、sys 等）的工具，可以在不影响或破坏现有 authenticode 签名的情况下嵌入数据（如 shellcode）。换句话说，您可以通过嵌入数据来更改 PE 文件校验和/哈希值，而不会破坏文件签名、完整性检查或 PE 文件功能。

- ✅ 保留原始文件的合法数字签名
- ✅ 将 Shellcode 隐藏在证书表（Certificate Table）中
- ✅ 不破坏原始文件的签名验证和功能

### 3️⃣ DLL 劫持执行
生成的文件包含：
- **主 EXE**：带有合法签名 + 隐藏 Shellcode 的可执行文件
- **劫持 DLL**：被主 EXE 加载时，从证书区提取并执行 Shellcode

**执行流程**：
```
运行 payload_signed.exe
    ↓
✅ Windows 验证签名通过（显示合法签名）
    ↓
程序加载同目录下的 DLL（DLL 劫持）
    ↓
 DLL 从 EXE 证书区提取 Shellcode
    ↓
在内存中执行 Shellcode
    ↓
🎭 程序表面正常运行，实际已被控制
```

**技术优势**：
- ✅ **绕过签名检查**：EXE 文件保持合法签名，受信任
- ✅ **隐蔽性强**：Shellcode 隐藏在证书区，常规扫描难以检测
- ✅ **DLL 劫持**：利用 Windows DLL 搜索顺序劫持合法 DLL
- ✅ **自动化流程**：一键完成 PE 转换 + 签名注入 + DLL 生成

## 🏷️ 支持的签名模板

### x86 架构模板
| 模板 | 来源 | 文件 |
|------|------|------|
| vsperfcmd.exe | Microsoft | vsperfcmd.exe + vspmsg.dll |
| vsperfmon.exe | Microsoft | vsperfmon.exe + vspmsg.dll |
| curl.exe | Sangfor | curl.exe + libcurl.dll |
| rc.exe | Microsoft | rc.exe + rcdll.dll |
| csc.exe | Microsoft | csc.exe + cscomp.dll |

### x64 架构模板
| 模板 | 来源 | 文件 |
|------|------|------|
| psl.exe | Microsoft | psl.exe + libpsl-5.dll |
| sfe_wscreg.exe | Sangfor | sfe_wscreg.exe + sfe_wscctrl.dll |
| curl.exe | Sangfor | curl.exe + libcurl.dll |
| rc.exe | Microsoft | rc.exe + rcdll.dll |
| csc.exe | Microsoft | csc.exe + cscomp.dll |

## 💻 系统要求

- **操作系统**：Windows 7 及以上（支持 Windows 10/11）
- **架构**：x86 / x64
- **权限**：
  - 大部分功能：普通用户权限
  - PengCode 工具：需要管理员权限

## 📖 使用方法

### 基本流程

1. **选择原始 PE 文件**
   - 拖拽 PE 文件到界面
   - 或手动输入文件路径
   - 程序会自动识别文件架构（x86/x64）

2. **选择转换工具**
   - Donut：通用选择，支持最多配置选项
   - PengCode：专业转换器，支持多种编译器（包括 Golang）
   - ZigDonut：用于 Rust 静态 TLS 程序
   - PE2SHC：快速简单转换

3. **配置选项**（根据工具不同）
   - Donut 支持多种配置：擦除 PE 头、压缩、绕过 AMSI/WLDP/ETW 等
   - ZigDonut 支持擦除 PE 头选项
   - 其他工具使用默认配置

4. **选择签名模板**
   - 根据目标架构（x86/x64）选择合适的签名模板
   - 模板来源包括 Microsoft、Sangfor 等知名厂商
   - 选择与目标环境相匹配的模板以提高隐蔽性

5. **点击生成**
   - 程序执行完整的武器化流程：
     1. 将 PE 转换为 Shellcode (.bin)
     2. 使用 SigFlip 将 Shellcode 注入到模板 EXE 的证书区
     3. 生成劫持 DLL，用于提取和执行 Shellcode
   - 生成两个文件：
     - `原文件名_signed.exe`：带有合法签名 + 隐藏 Shellcode 的 EXE
     - `模板DLL文件`：用于 DLL 劫持的 DLL

### 使用示例

```
输入文件：D:\payload\mimikatz.exe
转换工具：Donut
架构：x64（自动识别）
配置：
  ├─ 擦除 PE 头：✓
  ├─ 压缩输出：✓
  └─ 绕过 AMSI/WLDP/ETW：✓
签名模板：psl.exe (Microsoft)

生成文件：
- D:\payload\mimikatz_signed.exe  ← 带有 Microsoft 合法签名
- D:\payload\libpsl-5.dll          ← 劫持 DLL

部署方式：
将两个文件放在同一目录下，运行 mimikatz_signed.exe
```

## 🛡️ 防御检测

虽然 SigJack 具有较强的隐蔽性，但安全防护人员可以在以下方面进行检测：

### 检测点

1. **行为分析**
   - 监控 DLL 从 EXE 证书区读取数据的异常行为
   - 检测进程注入或内存执行行为
   - 监控低权限 DLL 加载顺序异常

2. **静态分析**
   - 检查证书区大小异常（正常签名 vs 包含 Shellcode）
   - 对比 DLL 导出函数与官方版本的差异
   - 分析 DLL 代码段的可疑操作

3. **EDR/XDR 防护**
   - 监控 DLL 加载路径（当前目录优先级劫持）
   - 检测内存中的可疑代码执行
   - 跟踪敏感 API 调用链（如 VirtualAlloc、CreateThread）

4. **哈希值与签名验证**
   - DLL 文件哈希值与官方版本不匹配
   - 虽然 EXE 签名验证通过，但证书链和时间戳可能存在异常
   - 使用深度签名验证工具检查证书区数据

### 防御建议

- ✅ 启用 **应用程序白名单** 控制可执行文件
- ✅ 使用 **EDR/XDR** 产品监控异常行为
- ✅ 部署 **DLL 劫持防护** 机制
- ✅ 定期进行 **威胁狩猎** 和 **IOC 扫描**
- ✅ 培训员工识别 **社会工程学攻击**
- ✅ 实施 **最小权限原则**

## ❓ 常见问题

### Q: 为什么生成的文件无法运行？

**可能的原因：**

1. **架构选择不正确**
   - 问题：选择的架构（x86/x64）与原始 PE 文件不匹配
   - 解决：程序会自动识别架构，请确认自动识别结果正确，或手动选择正确的架构

2. **转换工具不支持该类型文件**
   - 问题：某些特殊编译的 PE 文件可能不被当前工具支持
   - 解决：尝试切换不同的转换工具
     - Donut：适合大多数标准 PE 文件和 .NET 程序集
     - PengCode：专业转换器，兼容多种编译器（包括 Golang）
     - ZigDonut：适合 Rust 静态 TLS 程序
     - PE2SHC：适合简单的原生 PE 文件

3. **签名模板不兼容**
   - 问题：选择的签名模板可能与目标环境或文件类型不兼容
   - 解决：尝试切换不同的签名模板
     - Microsoft 模板（psl.exe、vsperfcmd.exe、rc.exe）：兼容性较好
     - Sangfor 模板（sfe_wscreg.exe、curl.exe）：针对特定环境

4. **目标环境缺少依赖**
   - 问题：生成的 DLL 依赖于特定的系统库
   - 解决：确保目标系统具有必要的运行时环境

5. **杀毒软件拦截**
   - 问题：杀毒软件或 EDR 检测到行为
   - 解决：在测试环境中暂时禁用安全软件进行验证

**建议的排查步骤：**
```
1. 确认架构匹配（x86 vs x64）
   ↓
2. 尝试不同的转换工具
   ↓
3. 更换签名模板
   ↓
4. 在无杀软环境测试
   ↓
5. 检查目标系统环境
```

## 🙏 致谢

本项目使用了以下开源工具和资源：

- **[Donut](https://github.com/TheWover/donut)** - 生成与位置无关的 shellcode，从内存加载 .NET 程序集、PE 文件和其他 Windows 有效负载
- **[SigFlip](https://github.com/med0x2e/SigFlip)** - 在不破坏 authenticode 签名的情况下修补 PE 文件，嵌入 shellcode
- **[PengCode](https://github.com/Mephostophiles/PengCode)** - 专业的 EXE 转 ShellCode 程序，支持多种编译器（包括 Golang）
- **[ZigDonut](https://github.com/howmp/zigdonut)** - 用 Zig 实现的精简版 Donut，支持 Rust 静态 TLS
- **[PE2SHC](https://github.com/hasherezade/pe_to_shellcode)** - 转换 PE 文件为可注入的 shellcode

感谢所有开源工具的作者和贡献者！

## 📄 许可证

本项目采用 MIT 许可证。详见 LICENSE 文件。

```
Copyright (c) 2025 K.Fire

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 📮 联系方式

- **GitHub**: [https://github.com/k-fire/sigjack](https://github.com/k-fire/sigjack)
- **Issues**: 如有问题或建议，请在 GitHub 提交 Issue

---

**再次提醒：本工具仅供安全研究和教育目的使用。使用本工具进行未经授权的活动是违法的。作者不对因使用本工具造成的任何损害负责。使用者需自行承担所有法律责任。**



