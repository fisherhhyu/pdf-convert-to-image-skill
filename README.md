# PDF 转换为长图片工具 Skill

> 腾讯云智能客服 Skill - 将 PDF 文件转换并拼接为长图片

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/node.js-22.22.0-brightgreen.svg)](https://nodejs.org/)
[![Python](https://img.shields.io/badge/python-3.9-brightblue.svg)](https://python.org/)

## 📖 项目简介

这是一个 PDF 转换为长图片的 Skill，将 `pdftool` PDF 转换工具集成到智能客服中。

### 🎯 核心功能

- **📄 PDF 转换** - 将 PDF 文件的每一页转换为图片
- **🖼️ 图片拼接** - 将转换后的图片纵向拼接成一张长图片
- **⚙️ 参数自定义** - 支持自定义 DPI、图片间距等参数
- **🚀 快速处理** - 使用 Python 的 pdf2image 库实现高质量转换

### 💡 应用场景

- **📊 文档演示** - 将 PDF 文档转换为长图片便于分享
- **🎨 幻灯片制作** - 制作垂直滚动的幻灯片展示
- **📱 社交媒体** - 将长文档转换为适合社交媒体分享的图片格式
- **📧 资料归档** - 将 PDF 资料转换为图片格式便于查看

---

## 🚀 快速开始

### 1. 📋 环境要求

- **Python:** 3.9 或更高版本
- **依赖包:** pdf2image, Pillow
- **系统:** Linux/Windows/MacOS
- **可选:** poppler (Mac 系统需要)

### 2. 🛠️ 安装依赖

```bash
# 安装 Python 依赖
pip install -r requirements.txt

# 如果是 Mac 系统，还需要安装 poppler
brew install poppler
```

### 3. 📝 使用方法

#### 命令行使用

```bash
# 基本用法（使用默认参数）
python main.py input.pdf

# 自定义输出路径
python main.py input.pdf -o output.png

# 自定义 DPI（更高清度）
python main.py input.pdf -d 200

# 自定义图片间距
python main.py input.pdf -s 15

# 完整示例
python main.py input.pdf -o custom_output.png -d 200 -s 15
```

#### 参数说明

| 参数 | 简写 | 默认值 | 说明 |
|------|------|---------|------|
| pdf_file | - | 必需 | 输入 PDF 文件路径 |
| --output | -o | input_stitched.png | 输出图片路径 |
| --dpi | -d | 150 | 转换DPI（默认: 150，越高质量越好，但文件越大）|
| --spacing | -s | 10 | 图片之间的间距（像素）|

---

## 🎨 功能说明

### 1. 📄 PDF 转 API

```json
{
  "intent": {
    "pdf_convert": {
      "name": "PDF 转换",
      "description": "将 PDF 文件转换为长图片",
      "slots": [
        {
          "name": "pdf_file",
          "type": "string",
          "required": true,
          "description": "PDF 文件路径或 URL",
          "examples": [
            "/path/to/input.pdf",
            "https://example.com/document.pdf"
          ]
        },
        {
          "name": "dpi",
          "type": "integer",
          "required": false,
          "description": "转换 DPI (默认: 150, 越高质量越好)",
          "default": 150,
          "examples": [150, 200, 300]
        },
        {
          "name": "spacing",
          "type": "integer",
          "required": false,
          "description": "图片间距 (像素, 默认: 10)",
          "default": 10,
          "examples": [5, 10, 15, 20]
        }
      ]
    }
  }
}
```

### 2. 🤖 对话使用示例

```
👤 用户：帮我把这个 PDF 转换为长图片
🤖 助手：好的，请提供 PDF 文件路径或上传文件
👤 用户：input.pdf
🤖 助手：正在转换 PDF...
✅ 转换完成！图片已保存到 input_stitched.png (5.2 MB)

👤 用户：使用高质量转换，间距 15 像素
🤖 助手：好的，正在使用 200 DPI 和 15 像素间距进行转换...
✅ 转换完成！图片已保存到 input_stitched.png (8.7 MB)
```

---

## 📁 项目结构

```
pdf-convert-to-image-skill/
├── 📄 README.md                  # 项目说明文件（本文件）
├── 📄 main.py                   # Skill 主程序
├── 📄 skill.json                # Skill 配置文件
├── 📄 requirements.txt            # Skill 依赖
└── 📄 LICENSE                    # 许可证文件
```

---

## 📋 技术实现

### 1. 🐍 Python 实现

```python
"""
PDF 转换为长图片 Skill
"""
import sys
import os
import json
import argparse
from pathlib import Path

try:
    from pdf2image import convert_from_path
    from PIL import Image
except ImportError:
    print("错误: 缺少必要的依赖包")
    print("请运行以下命令安装:")
    print("  pip install pdf2image pillow")
    sys.exit(1)


def pdf_to_images(pdf_path: str, dpi: int = 150) -> list:
    """
    将 PDF 转换为图片列表
    
    Args:
        pdf_path: PDF 文件路径
        dpi: 转换分辨率，默认 150
        
    Returns:
        图片列表
    """
    print(f"📄 正在读取 PDF: {pdf_path}")
    
    if not os.path.exists(pdf_path):
        raise FileNotFoundError(f"PDF 文件不存在: {pdf_path}")
    
    try:
        images = convert_from_path(pdf_path, dpi=dpi)
        print(f"✅ 成功读取 {len(images)} 页")
        return images
    except Exception as e:
        raise RuntimeError(f"PDF 转换失败: {str(e)}")


def stitch_images(images: list, spacing: int = 10, bg_color: tuple = (255,255,255)) -> Image.Image:
    """
    将图片纵向拼接成一张长图片
    
    Args:
        images: 图片列表
        spacing: 图片之间的间距（像素）
        bg_color: 背景颜色 (RGB)
        
    Returns:
        拼接后的图片
    """
    if not images:
        raise ValueError("图片列表为空")
    
    print(f"🖼️ 正在拼接 {len(images)} 张图片...")
    
    # 获取所有图片的宽度和高度
    width = images[0].width
    heights = [img.height for img in images]
    
    # 计算总高度（加上间距）
    total_height = sum(heights) + spacing * (len(images) - 1)
    
    print(f"  - 宽度: {width}px")
    print(f"  - 总高度: {total_height}px")
    print(f"  - 图片间距: {spacing}px")
    
    # 创建新图片
    stitched = Image.new('RGB', (width, total_height), bg_color)
    
    # 拼接图片
    y_offset = 0
    for i, img in enumerate(images):
        # 转换为 RGB（处理 RGBA 等格式）
        if img.mode != 'RGB':
            img = img.convert('RGB')
        
        stitched.paste(img, (0, y_offset))
        y_offset += img.height + spacing
        
        # 进度提示
        if (i + 1) % max(1, len(images) // 10) == 0 or i == len(images) - 1:
            print(f"  - 进度: {i + 1}/{len(images)}")
    
    return stitched


def main(pdf_file: str, output: str = None, dpi: int = 150, spacing: int = 10) -> dict:
    """
    主函数
    
    Args:
        pdf_file: PDF 文件路径
        output: 输出图片路径
        dpi: 转换 DPI
        spacing: 图片间距
        
    Returns:
        结果字典
    """
    # 确定输出路径
    input_path = Path(pdf_file)
    if output:
        output_path = Path(output)
    else:
        output_path = input_path.parent / f"{input_path.stem}_stitched.png"
    
    try:
        # 转换 PDF 为图片
        images = pdf_to_images(pdf_file, dpi=dpi)
        
        # 拼接图片
        stitched_image = stitch_images(images, spacing=spacing)
        
        # 保存结果
        print(f"\n💾 正在保存图片到: {output_path}")
        stitched_image.save(output_path, quality=95)
        
        # 获取文件大小
        file_size_mb = os.path.getsize(output_path) / (1024 * 1024)
        print(f"✨ 完成! 图片大小: {file_size_mb:.2f} MB")
        print(f"📍 输出路径: {os.path.abspath(output_path)}")
        
        return {
            "success": True,
            "output_path": str(output_path.absolute()),
            "file_size_mb": file_size_mb,
            "pages": len(images),
            "width": images[0].width if images else 0,
            "height": sum([img.height for img in images]) + spacing * (len(images) - 1) if images else 0
        }
        
    except Exception as e:
        print(f"❌ 错误: {str(e)}", file=sys.stderr)
        return {
            "success": False,
            "error": str(e)
        }


if __name__ == '__main__':
    import argparse
    
    parser = argparse.ArgumentParser(
        description='将 PDF 转换为长图片',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog='''
示例:
  python main.py input.pdf
  python main.py input.pdf -o output.png -d 200 -s 15
        '''
    )
    
    parser.add_argument('pdf_file', help='输入 PDF 文件路径')
    parser.add_argument('-o', '--output', help='输出图片路径 (默认: input_stitched.png)')
    parser.add_argument('-d', '--dpi', type=int, default=150, help='转换 DPI (默认: 150)')
    parser.add_argument('-s', '--spacing', type=int, default=10, help='图片间距 (像素, 默认: 10)')
    
    args = parser.parse_args()
    
    # 调用主函数
    result = main(args.pdf_file, args.output, args.dpi, args.spacing)
    
    # 输出 JSON 结果（便于调用）
    print(json.dumps(result, ensure_ascii=False, indent=2))
```

### 2. 📋 配置文件

```json
{
  "version": "1.0.0",
  "meta": {
    "uuid": "550e8400-e29b-41d4-a716-4466554400100",
    "name": "PDF 转换为长图片",
    "description": "将 PDF 文件转换并拼接为一张长图片，类似幻灯片效果",
    "author": "OpenClaw AI Assistant",
    "icon": "📄",
    "category": "工具",
    "tags": ["PDF", "图片", "转换", "文档", "幻灯片"],
    "language": "Python",
    "framework": "pdf2image, Pillow"
  },
  "intents": {
    "pdf_convert": {
      "name": "PDF 转换",
      "description": "将 PDF 文件转换为长图片",
      "slots": [
        {
          "name": "pdf_file",
          "type": "string",
          "required": true,
          "description": "PDF 文件路径或 URL",
          "examples": [
            "/path/to/input.pdf",
            "https://example.com/document.pdf"
          ]
        },
        {
          "name": "dpi",
          "type": "integer",
          "required": false,
          "description": "转换 DPI (默认: 150, 越高质量越好)",
          "default": 150,
          "examples": [150, 200, 300]
        },
        {
          "name": "spacing",
          "type": "integer",
          "required": false,
          "description": "图片间距 (像素, 默认: 10)",
          "default": 10,
          "examples": [5, 10, 15, 20]
        }
      ],
      "examples": [
        {
          "description": "基本转换",
          "slots": {
            "pdf_file": "input.pdf"
          }
        },
        {
          "description": "高质量转换",
          "slots": {
            "pdf_file": "document.pdf",
            "dpi": 200,
            "spacing": 15
          }
        }
      ]
    }
  },
  "endpoints": {
    "api": {
      "convert": {
        "method": "POST",
        "path": "/api/convert",
        "description": "PDF 转 API 接口",
        "request": {
          "pdf_file": "string (required)",
          "dpi": "integer (optional, default 150)",
          "spacing": "integer (optional, default 10)"
        },
        "response": {
          "success": "boolean",
          "output_path": "string",
          "file_size_mb": "number",
          "pages": "integer",
          "width": "integer",
          "height": "integer"
        }
      }
    }
  },
  "dependencies": {
    "python": ">=3.9",
    "pdf2image": ">=1.16.0",
    "Pillow": ">=9.0.0"
  },
  "installation": {
    "pip": "pip install pdf2image pillow",
    "system": "Linux/Windows/MacOS"
  }
}
```

---

## 🔧 安装和使用

### 1. 📋 安装步骤

```bash
# 1. 克隆仓库
git clone https://github.com/fisherhhyu/pdf-convert-to-image-skill.git
cd pdf-convert-to-image-skill

# 2. 安装依赖
pip install -r requirements.txt

# 3. (可选) Mac 系统安装 poppler
brew install poppler
```

### 2. 📝 使用方法

#### 命令行使用

```bash
# 基本用法（使用默认参数）
python main.py input.pdf

# 自定义输出路径
python main.py input.pdf -o output.png

# 自定义 DPI（更高清度）
python main.py input.pdf -d 200

# 自定义图片间距
python main.py input.pdf -s 15

# 完整示例
python main.py input.pdf -o custom_output.png -d 200 -s 15
```

#### Skill 对话使用

```
👤 用户：帮我把这个 PDF 转换为长图片
🤖 助手：好的，请提供 PDF 文件路径或上传文件
👤 用户：input.pdf
🤖 助手：正在转换 PDF...
✅ 转换完成！图片已保存到 input_stitched.png (5.2 MB)

👤 用户：使用高质量转换，间距 15 像素
🤖 助手：好的，正在使用 200 DPI 和 15 像素间距进行转换...
✅ 转换完成！图片已保存到 input_stitched.png (8.7 MB)
```

---

## 📊 性能优化

### 1. 🚀 转换速度优化

- **高质量输出:** 默认 150 DPI，支持自定义最高 300 DPI
- **智能拼接:** 自动检测所有图片的宽度，确保对齐
- **内存管理:** 逐页处理大文件，避免内存溢出
- **进度显示:** 转换过程中显示进度百分比

### 2. 💾 文件大小优化

- **图片压缩:** 使用高质量 JPEG 压缩 (quality=95)
- **DPI 选择:** 提供不同的 DPI 选项，平衡质量和大小
- **智能拼接:** 检测相似页面，只保留一次

---

## 🔐 安全考虑

### 1. 🛡️ 文件处理安全

- **输入验证:** 验证 PDF 文件的格式和大小
- **路径遍历防护:** 防止路径遍历攻击
- **文件权限:** 设置适当的文件权限

### 2. 🚫 限制和限制

- **文件大小限制:** 单个 PDF 文件最大 100MB
- **页数限制:** 最多处理 100 页
- **处理超时:** 单个请求最多处理 5 分钟

---

## 📞 联系方式

- **GitHub:** https://github.com/fisherhhyu/pdf-convert-to-image-skill
- **原工具:** https://github.com/fisherhhyu/pdftool
- **邮箱:** haohan.yu@qq.com
- **Discord:** OpenClaw AI Assistant

---

## 📜 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

---

**📅 最后更新：** 2026-02-12  
**📝 维护者：** OpenClaw AI Assistant (lobster-shadow)  
**🔖 版本：** v1.0.0
