---
title: DeepSeek本地部署
sticky: true
cover: images/cover/ai1.jpg
categories:
  - [AI]
audio:
  - https://music.163.com/song?id=441612583&uct2=U2FsdGVkX1/2wR2+hDCkl/GHEI1HC2XDs4N9tUYp5io=
  - https://music.163.com/song?id=2027067479&uct2=U2FsdGVkX1+QWzzNFzHXwwi04+otTrc9VuGiHHiOfHc=
  - https://music.163.com/song?id=1942946560&uct2=U2FsdGVkX1+7SUSUZY/IXv+guqGZihFCP3AeXUzErj0=
---

## 前言

（碎碎念有点长，可以左侧导航到主题内容）

GPT 出来爆火了一轮，心灵导师、代码生成器、文案创业神器等等名头，讨论重心在于人工智能要替代人类了，但事实上很多人靠帮开 GPT 账号和所谓的 GPT 攻略挣得盆满钵满，替不替代都是后话了。

到了百度、豆包、通义千问都不温不火，反而 kimi 在程序员中间使用率高点（也可能是我个人偏好），我在 GPT 限制的时候会更喜欢用 kimi 来协助写代码。

有些会说 ai 生成代码会降低自己的编码技术，其实我觉得那些简单的代码让它们生成也没关系，很多时候还是要靠自己找到问题和屡清逻辑，发现问题、找到问题、给出思路或者结合思路也是人的核心能力，现在让公司的 CTO 完整写个功能算法也得靠查，我认为解决问题的能力在于--**尽可能找出更多解决问题的方法，然后更高效地解决问题**，毕竟科技发展了这么多年，总不能说因为电、网络等等技术降低了人的独立生存能力，就不去使用吧。

但另一方面需要考虑的是，确实现在面试还得考一些代码和算法，平常这些写少了手生了也得花费更多的时间和精力再熟悉起来，虽然大家都知道八股和算法问起来作用不大，但人还是太多了，筛选条件自然也就多了，没办法，想跳槽啥的还是得复习。

说到文章主题--Deep seek，不同厂商的模型之间都存在一些细微的差异，很难说它在模型上比 gpt 好，但是突破性进展就是每个人都可以培育自己的 ai 了，从实用性上来讲，也可以做一个比较“聪明”的个人知识库出来，但如果要继续升级的话，肯定还是需要更好的芯片，英伟达还是不至于会因为这个的出现而破产的^\_^。

至于人工智能对人劳动力的替代性，珍妮纺纱机当初也讨论过，但结论似乎也只是讲科技进步的副作用，人多么渺小，无论是政治、经济或是自然的细微变动都容易被轻轻抹杀，也就是大家讲时代的一粒沙，给具体的人一座山，但科技又比其他好一点，会带来一些新机会，比如卖攻略哈哈，当然不止这些，但人不能太消极，来这个世界几十年，应该尝试体验更多事物，做一个“勇敢、正直、有阅读量”的人ヾ(°v°)ﾉﾞ！

## 模型类型解析

### 按模型规模

| 模型类型     | 参数量   | 典型应用场景           | 显存需求       | 内存需求 |
| :----------- | :------- | :--------------------- | :------------- | :------- |
| DeepSeek-7B  | 70 亿    | 本地测试/基础 NLP 任务 | 最低 10GB VRAM | 16GB+    |
| DeepSeek-13B | 130 亿   | 企业级对话/文档分析    | 最低 24GB VRAM | 32GB+    |
| DeepSeek-MoE | 混合专家 | 多任务并行处理         | 多卡部署       | 64GB+    |

### 按功能类型

- **Base 版**：基础语言模型，适合微调开发
- **Chat 版**：对话优化版本，支持多轮交互
- **Coder 版**：代码生成专用版本（需 16GB+显存）

## 硬件要求

### 最低配置

- CPU：Intel Xeon Silver 4210+ 或等效 AMD EPYC
- GPU：NVIDIA RTX 3090（24GB VRAM）
- 内存：32GB DDR4 ECC
- 存储：NVMe SSD 200GB+可用空间

### 推荐配置

- GPU：NVIDIA A10/A100（多卡并行）
- 显存：每卡 24GB+（13B 模型建议双卡）
- 内存：64GB+ DDR4 ECC
- 存储：RAID0 NVMe 阵列 1TB+

> ⚠️ Windows 特有注意：需开启 Large Page Support，BIOS 中禁用 Secure Boot

## 部署流程（以 DeepSeek-7B-Chat 为例）

### 环境准备

```
# 安装CUDA 11.8（必须匹配PyTorch版本）
choco install cuda --version=11.8.0

# 创建Python 3.10虚拟环境
conda create -n deepseek python=3.10
conda activate deepseek

# 安装PyTorch（Windows特定版本）
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# 安装依赖库（注意版本锁定）
pip install transformers==4.33.0 accelerate sentencepiece einops
```

### 模型下载

```
# 使用官方提供的下载工具（避免断线）
Import-Module BitsTransfer
Start-BitsTransfer -Source "https://models.deepseek.com/7B-Chat-v2/windows" -Destination ".\models"

# 校验文件完整性（示例）
Get-FileHash .\models\pytorch_model.bin -Algorithm SHA256
# 应匹配官方提供的 2a8b4c...（具体值需查最新文档）
```

### 配置文件调整

修改 `model_config.yaml`：

```
device_map: "auto"
torch_dtype: "auto"
# Windows特定设置
use_flash_attention: false  # Win暂不支持Flash Attention
disk_cache_path: "D:\\deepseek_cache"  # 使用NTFS格式路径
```

### 启动推理服务

```
# 加载优化后的Windows版代码
from deepseek_winserver import ChatServer

server = ChatServer(
    model_path="models/7B-Chat-v2",
    gpu_mem_alloc=0.8  # 显存分配比例
)
server.start(port=8080, api_key="your_secret_key")
```

## 常见问题解决方案

### CUDA 内存不足

```
# 启用8bit量化（降低显存占用）
model = AutoModelForCausalLM.from_pretrained(
    model_path,
    load_in_8bit=True,
    device_map='auto'
)

## 配置页面文件（Windows特有）
wmic pagefileset create name="C:\\pagefile.sys",initialSize=32768,maximumSize=32768
```

### DLL 加载错误

- 安装 VC++ 2022 可再发行组件包
- 更新 NVIDIA 驱动至 545+
- 设置环境变量：

```
$env:PYTORCH_CUDA_ALLOC_CONF = "max_split_size_mb:128"
```

### 中文乱码问题

```
# 在代码首部添加：
import sys
import io
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
```

## 性能优化技巧

### 显存优化方案

```
# 梯度检查点技术（13B+模型推荐）
model.gradient_checkpointing_enable()

# Windows专属优化注册表（需管理员权限）
reg add HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management /v PagedPoolSize /t REG_DWORD /d 0xFFFFFFFF
```

### 多卡部署配置

```
# 修改accelerate配置文件
compute_environment: LOCAL_MACHINE
distributed_type: MULTI_GPU
fp16: true
num_processes: 2
device_ids: all
```

## 安全注意事项

### 访问控制

```
# 配置Windows防火墙
New-NetFirewallRule -DisplayName "DeepSeek" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow -Profile Any
```

### 模型加密

```
# 使用Windows DPAPI加密模型
from cryptography.fernet import Fernet
import win32crypt

key = win32crypt.CryptProtectData(b"deepseek_key", None, None, None, None, 0x01)
cipher_suite = Fernet(key)
encrypted_model = cipher_suite.encrypt(model_bytes)
```

## 版本升级指南

1. 保留当前环境的完整快照
2. 使用官方提供的`migration_tool.exe`进行版本迁移
3. 执行完整性检查：

```
.\DeepSeekValidator.exe --model 7B-Chat-v2 --check all
```

> 最后更新：2024 年 1 月（部署前还可以查看官方 Windows 部署专页获取最新信息）

---

## 补充说明

- 对于 13B 及以上模型，建议使用 Windows Server 2022 数据中心版
- 推荐使用 WSL2 作为备用方案，当遇到驱动兼容性问题时
- 定期检查 NVIDIA Windows 驱动更新（建议每月至少一次）
