# Lightweight LLM Controller（极轻量级本地 LLM 图形控制台）

这是一个基于原生 PowerShell 与 WinForms 构建的 llama.cpp 桌面端图形封装层（GUI Wrapper）。项目摒弃了主流的 Web 渲染框架（如 Electron），采用原生 .NET 组件与系统底层 API 交互，旨在为本地大语言模型推理提供低开销、高响应的调度体验。

- 🚀 **极致轻量**：零第三方运行时依赖（无 Node.js / Python），解压即用。
- ⚡ **免重启热切换**：前后端双轨控制，动态注入 `chat_template_kwargs`，实现推理深度秒级热切换。
- 🛡️ **高鲁棒性**：支持自定义引擎路径、多模型配置隔离、参数转义防注入。

[控制台主界面]

<img width="762" height="912" alt="QQ_1794734263263" src="https://github.com/user-attachments/assets/9c7ca9cc-f3f9-4ecb-bbb2-ab54995bce16" />
<img width="762" height="912" alt="QQ_1794734271765" src="https://github.com/user-attachments/assets/9f426724-6cf9-4504-bd95-34dd21457435" />


## 📥 快速开始

### 1. 系统要求
- **操作系统**：Windows 10 / 11（64 位）
- **运行环境**：PowerShell 5.1 或更高版本（系统自带）
- **硬件建议**：NVIDIA 显卡（推荐），或使用 CPU 模式
- **磁盘空间**：至少 500 MB 用于存放引擎，模型文件另计

### 2. 获取推理引擎与模型
- **引擎**：访问 [llama.cpp Releases](https://github.com/ggml-org/llama.cpp/releases) 下载对应的 CUDA/CPU 压缩包。解压后，将 `llama-server.exe` 及所有 `.dll` 放入本项目根目录下的 `engine` 文件夹内。
- **模型**：准备 `.gguf` 格式的模型文件（推荐从 Hugging Face 或 ModelScope 获取）。

### 3. 首次配置与启动
1. **启动控制台**：双击 `启动控制台.vbs`。
2. **设置引擎路径**：点击右下角【设置...】→「引擎路径」点击【浏览...】选中 `engine\llama-server.exe` → 点击【关闭】。
3. **选择模型**：在「服务设置」页，找到「模型文件」一栏，点击【浏览...】选中你的 `.gguf` 文件。
4. **启动服务**：点击左下角【启动服务】，等待状态栏显示「运行中」。
5. **开始对话**：切换到「对话」页，输入问题，点击【发送】即可。

## ✨ 核心特性

- **原生轻量架构**：基于 PowerShell 5.1 + WinForms 原生渲染，摒弃臃肿的 Web 框架（如 Electron）。零第三方环境依赖（无 Node.js / Python），解压即用，空载内存极低。
- **动态界面布局自定义（UI Layout Customization）**：打破传统 GUI 写死布局的局限。在【服务设置】页右键点击任意参数，可选择【上移】或【下移】。程序会动态重建 UI，并将最新的排序序列化存入 `settings.json` 的 `layout_order` 字段，重启后依然保留你专属的参数排列。
  <img width="760" height="910" alt="Videoshot_20261115_174703" src="https://github.com/user-attachments/assets/9cda9ac5-2a57-42ad-999d-0b8512e02645" />

- **免重启推理深度热切换（核心机制）**：实现前后端双轨控制。前端通过 HTTP 动态注入 `chat_template_kwargs`（`enable_thinking` 与 `reasoning_effort`），结合自定义 Jinja2 模板与底层 `--reasoning-budget` 物理兜底，实现思考等级拉条的秒级热切换，且触发配置签名校验跳过，完全不弹重启提示。
- **多模型配置隔离与防污染**：基于模型绝对路径的 `model_configs` 隔离机制，为每个模型独立保存 `ctx`、`ngl`、温度等参数。切换模型时自动清空历史记录，防止上下文串味。
- **多模态与思维链可视化**：对话界面支持附加本地图片（转为 Base64），完美适配 Qwen-VL 等多模态模型。同时，若模型返回 `reasoning_content`（如 DeepSeek-R1），界面会将“思考过程”以灰色字体独立渲染，与最终回复区分。
- **极致鲁棒性与防呆设计**：内置外部程序参数转义防注入（`Q()` 函数处理路径空格与双引号）；调用 Win32 API 强制重绘修复 RichTextBox 假死；针对小于 12GB 的小模型自动拉满 `ngl`（GPU层数）防爆显存；思考白名单机制（`effort_keywords`）强制隔离原生思考模型，防止拉条误伤。
- **自定义引擎路径分离**：支持将庞大的 `llama-server.exe` 与 `.dll` 放在任意目录。程序通过 `engineExe` 字段动态读取引擎路径并设置工作目录，甚至能从中派生进程名以安全停止后台服务。

## 🛠️ 高级功能

- **更新引擎**：点击【设置...】→【检查更新...】，程序会自动拉取最新 CUDA 版本。
- **自定义思考等级**：前往【设置...】→「配置思考白名单...」，支持动态调整模型推理深度（如 Qwen3.5 等）。
- **开机自启**：程序内置 `autostart.ps1`，可在设置中一键开启。

## ❓ 常见问题

| 问题 | 解决方法 |
|------|----------|
| 提示「未找到有效引擎路径」 | 前往【设置...】重新选择 `llama-server.exe` |
| 端口被占用（如 8090 被占用） | 在「服务设置」页将「端口」改为其他数值 |
| 显存不足导致启动失败 | 将「GPU 层数」调小（如从 999 改为 10~15） |
| 杀毒软件拦截 `llama-server.exe` | 将整个程序目录加入杀毒软件信任区 |
| 思考等级拉条灰色不可用 | 模型名需包含 `qwen3.5`，或去设置中手动添加白名单 |

## 🤯 0 行手写代码：100% 由 AI 生成

**郑重声明：本项目的每一行代码（包括 PowerShell 脚本、WinForms UI 布局、底层逻辑与防呆机制），均由 AI 独立编写完成，人类未手写任何一行代码。**

- 本项目为个人实验性项目，用于在 Windows 上快速启动 llama.cpp 的 llama-server.exe 并调节推理参数。它不是生产级工具，不承诺稳定性、兼容性或维护响应。
- 代码来源：本项目代码主要由 AI 生成，作者负责需求定义、功能测试与人工审阅。未经安全审计，请勿在不信任规则文件、不信任模型来源、或对安全有要求的场景中使用。
- 平台限制：仅面向 Windows 10/11 64 位，仅适配 llama.cpp 的 llama-server.exe。
- 使用风险：本项目会启动外部进程、读取本地规则/配置、拼接命令行参数。请自行确认路径、配置与模型文件来源可信。因使用本项目造成的任何数据丢失、系统异常、模型输出问题或其他直接/间接损失，作者不承担责任。

- 无保证：本项目按“现状”提供，不附带任何明示或暗示的保证，包括但不限于适销性、特定用途适用性及不侵权保证。

## 📂 文件结构说明

```text
Lightweight-LLM-Controller/
├── 控制台.ps1                 # 主程序
├── autostart.ps1             # 开机自启脚本
├── 升级.ps1                  # 引擎更新脚本
├── 启动控制台.vbs            # 无黑框启动器
├── settings.json             # 配置文件
├── qwen35-effort.jinja       # 思考控制模板
├── auto_generic_effort.jinja # 通用兜底模板
└── engine/                   # 推理引擎目录（需自行放置 llama-server.exe 及 DLL）
