# Chatterbox TTS 项目个人化修改记录

## 📝 项目概述
- **项目名称**: chatterbox (resemble-ai)
- **项目路径**: `/Users/joe/github/3rdParty/chatterbox`
- **修改时间**: 2025年5月29-30日
- **修改目的**: 将在线TTS项目改造为完全离线的个人化本地系统

## 🔄 Git提交历史

### Commit 1: `cf0b671` - 自定义gradio TTS应用本地使用
**时间**: Fri May 30 01:55:21 2025 +0800  
**文件**: `gradio_tts_app.py` (18行增加, 4行删除)

#### 主要修改内容：

**1. Apple Silicon设备适配**
```python
# 原来：
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"

# 修改为：
DEVICE = "mps" if torch.cuda.is_available() else "cpu"
MAP_LOCATION = torch.device(DEVICE)
```

**2. Torch模型加载补丁**
```python
# 添加全局torch.load补丁，确保模型加载到正确设备
torch_load_original = torch.load
def patched_torch_load(*args, **kwargs):
    if 'map_location' not in kwargs:
        kwargs['map_location'] = MAP_LOCATION
    return torch_load_original(*args, **kwargs)
torch.load = patched_torch_load
```

**3. 模型加载路径本地化**
```python
# 原来：
model = ChatterboxTTS.from_pretrained(DEVICE)  # 在线下载

# 修改为：
model = ChatterboxTTS.from_local("./models", DEVICE)  # 本地加载
```

**4. 网络访问配置**
```python
# 原来：
.launch(share=True)  # 公网分享

# 修改为：
.launch()  # 仅本地访问，保护隐私
```

### Commit 2: `2e65421` - 添加参考语音文件
**时间**: Fri May 30 01:55:21 2025 +0800  
**文件**: 
- `ref_voice/joe.wav` (756,524 bytes)
- `ref_voice/mi.wav` (1,701,164 bytes)

**说明**: 添加个人语音样本库，用于语音克隆

### Commit 3: `8da3e92` - 添加本地模型文件
**时间**: Fri May 30 01:55:21 2025 +0800  
**文件**: `models` (Git子模块)

**说明**: 添加Git子模块指向本地模型仓库 (commit: f96a40510e8b10b046a37108bad33fd991814b9c)

## 📁 本地文件结构

### Models目录 (总计 ~2.1GB)
```
models/
├── .git/               # Git子模块
├── .gitattributes      # Git LFS配置
├── README.md           # 模型说明文档
├── s3gen.pt           # 主语音生成模型 (1.06GB)
├── t3_cfg.pt          # Text-to-Speech配置模型 (1.06GB)
├── ve.pt              # 语音编码器 (5.7MB)
├── conds.pt           # 条件向量数据 (107KB)
└── tokenizer.json     # 文本分词器 (25KB)
```

### 语音样本库
```
ref_voice/
├── joe.wav            # 个人语音样本 (756KB)
└── mi.wav             # 第二个语音样本 (1.7MB)
```

## 🏗️ 技术架构

### 核心优化
1. **设备兼容性**: 支持Apple Silicon (M1/M2) 的MPS加速
2. **离线运行**: 所有模型本地存储，无需联网
3. **隐私保护**: 仅本地访问，不对外分享
4. **个人化**: 支持个人语音克隆

### 模型组件说明
- **s3gen.pt**: 主要的语音生成神经网络模型
- **t3_cfg.pt**: Text-to-Speech的配置和权重文件
- **ve.pt**: Voice Encoder，用于处理参考语音
- **conds.pt**: 条件向量，控制生成参数
- **tokenizer.json**: 文本预处理和分词

## 🚀 使用方法

### 启动应用
```bash
cd /Users/joe/github/3rdParty/chatterbox
python gradio_tts_app.py
```

### 功能特性
- ✅ 输入任意中英文文本
- ✅ 选择joe.wav或mi.wav作为音色参考
- ✅ 调整语音参数（温度、夸张度、随机种子等）
- ✅ 生成高质量的个人化语音文件
- ✅ 完全离线运行，保护隐私

## 📋 Git管理策略

### 当前状态
- **分支**: master
- **远程同步**: 已合并远程更新 (16个新提交)
- **本地领先**: 4个提交 (3个个人修改 + 1个merge commit)
- **建议**: 不要推送到远程仓库 (包含个人文件和配置)

### 更新策略
```bash
# 拉取远程更新（已配置为merge模式）
git pull

# 查看状态
git status
```

## ⚠️ 注意事项

1. **不要推送**: 个人语音文件和模型不应推送到原项目
2. **模型大小**: models目录约2GB，注意磁盘空间
3. **设备要求**: 建议使用Apple Silicon Mac或CUDA GPU
4. **隐私安全**: 语音样本为个人数据，注意保护

## 🔄 更新记录

- **2025-06-21**: 成功合并远程更新，保留所有个人修改
- **2025-05-30**: 完成初始本地化配置
- **2025-05-29**: 开始项目个人化改造

---

**备注**: 此文档记录了对chatterbox项目的完整个人化改造过程，包含所有技术细节和使用说明。