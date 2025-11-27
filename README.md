# Euterpe - AI音乐生成系统

Euterpe是一个基于人工智能的音乐生成系统，使用户能够通过文本描述创作独特的音乐作品。该系统使用Facebook的MusicGen模型，支持多种音乐风格和情感表达。

## 项目结构

```
Euterpe/
  - Code/                 # 项目代码
    - backend/            # 后端服务
      - music_generator.py  # 音乐生成核心模块
      - main.py             # FastAPI服务器
      - config.py.example   # 配置文件示例
      - requirements.txt    # 依赖项
      - models/             # 模型存储目录
      - generated_music/    # 生成的音乐存储目录
    - frontend/           # 前端界面
      - index.html          # 主页
      - chatbot.html        # AI聊天界面
      - chat.js             # 聊天功能脚本
      - style.css           # 样式表
```

## 技术栈

- **后端**:
  - FastAPI: 高性能的Python API框架
  - Hugging Face Transformers: 用于加载和运行MusicGen模型
  - PyTorch: 深度学习框架

- **前端**:
  - HTML5/CSS3/JavaScript
  - Bootstrap 5: 响应式UI框架
  - Font Awesome: 图标库

## 安装与启动

### 先决条件

- Python 3.8+ 
- Anaconda 或 Miniconda
- NVIDIA GPU (可选，推荐用于加速)

### 1. 后端设置

1. 进入后端目录:
   ```bash
   cd Code/backend
   ```

2. 使用Conda创建环境:

   ```bash
   # 创建并激活环境
   conda create -y -n euterpe python=3.10
   conda activate euterpe

   # GPU用户: 安装CUDA版PyTorch
   pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu121

   # 安装其他依赖
   pip install fastapi uvicorn openai python-multipart
   pip install transformers scipy
   pip install accelerate
   pip install bitsandbytes psutil
   ```

<!-- 3. 配置API密钥:
   ```bash
   # 复制示例配置文件
   cp config.py.example config.py
   
   # 编辑config.py并添加您的API密钥
   # OPENAI_API_KEY = "your-openai-api-key"
   # HUGGINGFACE_TOKEN = "your-huggingface-token"
   ``` -->

3. 启动后端服务:
   ```bash
   # 确保conda环境已激活
   conda activate euterpe
   
   # 启动服务器
   uvicorn main:app --reload --host 0.0.0.0 --port 8000
   ```

### 2. 前端设置

1. 使用任意Web服务器提供前端文件:

   ```bash
   # 进入前端目录
   cd Code/frontend
   
   # Python内置HTTP服务器
   python -m http.server 5500
   
   # 或者使用Node.js (如果已安装)
   # npx http-server -p 5500
   ```

2. 在浏览器中访问:
   ```
   http://localhost:5500
   ```

## 使用方法

1. 在主页上点击"开始创作音乐"按钮
2. 在聊天框中输入音乐描述，例如:
   - "创作一段欢快的钢琴曲，带有轻快的节奏和明亮的旋律"
   - "生成一段适合冥想的环境音乐，包含舒缓的自然声音"
3. 点击发送按钮，等待系统生成音乐
4. 使用音频控件播放生成的音乐
5. 您可以在历史记录中找到之前生成的所有音乐

## API文档

后端提供了Swagger UI文档，可通过以下地址访问:
```
http://localhost:8000/docs
```

## 性能注意事项

- MusicGen是一个大型模型，需要较大的GPU内存才能高效运行
- 在CPU上运行时，首次加载模型可能需要几分钟时间
- 生成音乐可能需要10-30秒，取决于硬件性能

## GPU加速指南

系统已针对GPU加速进行了优化，如果您有兼容的GPU，可以获得显著的性能提升。

### GPU要求

- CUDA兼容的NVIDIA GPU (推荐8GB+内存)
- 已安装CUDA和cuDNN
- PyTorch的CUDA版本

### GPU优化特性

1. **自动检测与配置**
   - 系统会自动检测可用的GPU
   - 根据GPU能力选择最佳精度(FP16/BF16/FP32)
   - 动态调整生成参数以适应可用内存

2. **内存优化**
   - 支持4位量化(依赖bitsandbytes库)
   - 内存不足时自动降级到小模型
   - 智能管理CUDA缓存

3. **模型选项**
   - `facebook/musicgen-large` (默认，需要≥10GB GPU内存)
   - `facebook/musicgen-medium` (中等质量，需要≥5GB GPU内存)
   - `facebook/musicgen-small` (较低质量，需要≥3GB GPU内存)

### 检查GPU状态

系统提供了一个API端点来检查GPU状态：

```
GET http://localhost:8000/model/status
```

此端点返回详细信息，包括:
- GPU名称和可用性
- 内存使用情况
- 模型大小和参数数量

### 无GPU环境

如果没有兼容的GPU，系统会自动降级到CPU模式。请注意：
- 加载时间可能长达10-15分钟
- 生成单个音频可能需要数分钟
- 内存使用量较大(≥16GB RAM推荐)

## 故障排除

### 常见问题

1. **"需要accelerate库"错误**

   如果您在启动时看到此错误:
   ```
   ValueError: Using a `device_map`, `tp_plan`, `torch.device` context manager or setting `torch.set_default_device(device)` requires `accelerate`. You can install it with `pip install accelerate`
   ```

   解决方案:
   ```bash
   pip install accelerate
   ```

2. **内存不足错误**

   如果您在生成音乐时看到内存错误:

   解决方案:
   - 尝试使用较小的模型（修改`music_generator.py`中的`model_name_hf`参数为`"facebook/musicgen-small"`）
   - 减少生成令牌数量（修改`max_new_tokens`参数为较小值，如128）

3. **前端无法连接到后端**

   确保:
   - 后端服务正在运行（http://localhost:8000）
   - 前端正确配置了后端URL（查看`chat.js`中的`API_BASE_URL`变量）
   - 浏览器未阻止跨域请求

4. **音频播放问题**

   如果生成的音频无法播放:
   - 确保`generated_music`目录存在并有写入权限
   - 检查浏览器控制台是否有404错误
   - 尝试直接在后端URL访问音频文件（例如：http://localhost:8000/generated_music/xxx.wav）

## 许可证

本项目采用MIT许可证。
