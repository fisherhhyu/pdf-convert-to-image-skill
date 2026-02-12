# PDF 转换为长图片工具 Skill

> PDF 转换为长图片工具 Skill - 将 PDF 文件转换并拼接为长图片

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/node.js-22.22.0-brightgreen.svg)](https://nodejs.org/)
[![Python](https://img.shields.io/badge/python-3.9-brightblue.svg)](https://python.org/)

## 📖 项目简介

这是一个 PDF 转换为长图片的工具 Skill，基于 `pdftool` PDF 转换工具开发。

### 🎯 核心功能

- **📄 PDF 转换** - 将 PDF 文件的每一页转换为图片
- **🖼️ 图片拼接** - 将转换后的图片纵向拼接成一张长图片（类似幻灯片）
- **⚙️ 参数自定义** - 支持自定义 DPI、图片间距等参数
- **🚀 快速处理** - 使用 Python 的 pdf2image 库实现高质量转换

### 💡 应用场景

- **📊 文档演示** - 将 PDF 文档转换为长图片便于分享
- **🎨 幻灯片制作** - 制作垂直滚动的幻灯片展示
- **📱 社交媒体** - 将长文档转换为适合社交媒体分享的图片格式
- **📧 嵄料归档** - 将 PDF 嵄料转换为图片格式便于查看

---

## 🚀 快速开始

### 1. 📋 环境要求

- **Python:** 3.9 或更高版本
- **依赖包：** pdf2image、Pillow
- **系统：** Linux/Windows/MacOS
- **可选：** poppler（Mac 系统需要）

### 2. 🛠️ 安装依赖

```bash
# 安装 Python 依赖
pip install pdf2image pillow

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
python main.py input.pdf -o output.png -d 200 -s 15
```

#### 参数说明

| 参数 | 简写 | 默认值 | 说明 |
|------|------|---------|------|
| pdf_file | - | 必需 | 输入 PDF 文件路径 |
| --output | -o | input_stitched.png | 输出图片路径 (默认: input_stitched.png) |
| --dpi | -d | 150 | 转换DPI (默认: 150，越高质量越好，但文件越大） |
| --spacing | -s | 10 | 图片之间的间距 (像素) (默认: 10) |

---

## 🎨 Skill 配置

### Skill 元数据

```json
{
  "name": "PDF 转换为长图片",
  "description": "将 PDF 文件转换并拼接为一张长图片，类似幻灯片效果",
  "version": "1.0.0",
  "author": "OpenClaw AI Assistant",
  "icon": "📄",
  "category": "工具",
  "tags": ["PDF", "图片", "转换", "文档", "幻灯片"],
  "language": "Python",
  "framework": "pdf2image, Pillow"
}
```

### Skill 入口配置

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
          "description": "PDF 文件路径或 URL"
        },
        {
          "name": "dpi",
          "type": "integer",
          "required": false,
          "description": "转换 DPI (默认: 150)",
          "default": 150
        },
        {
          "name": "spacing",
          "type": "integer",
          "required": false,
          "description": "图片间距 (默认: 10)",
          "default": 10
        }
      ]
    }
  }
}
```

---

## 📦 技术实现

### 1. 🐍 Python 实现

```python
"""
PDF 转换为长图片 Skill
"""
import sys
import os
import json
from pathlib import Path
from typing import Dict, Any

# 当前文件目录
CURRENT_DIR = Path(__file__).parent

try:
    from pdf2image import convert_from_path
    from PIL import Image
except ImportError:
    print("❌ 错误: 缺少必要的依赖包")
    print("请运行以下命令安装:")
    print("  pip install pdf2image pillow")
    sys.exit(1)


class PDFConvertSkill:
    """PDF 转换为长图片 Skill"""
    
    def __init__(self):
        self.name = "PDF 转换为长图片"
        self.version = "1.0.0"
        self.description = "将 PDF 文件转换并拼接为一张长图片，类似幻灯片效果"
    
    def convert_pdf(self, pdf_path, output_path=None, dpi=150, spacing=10):
        """
        转换 PDF 为长图片
        
        Args:
            pdf_path: PDF 文件路径
            output_path: 输出图片路径（可选）
            dpi: 转换 DPI，默认 150
            spacing: 图片间距（像素），默认 10
            
        Returns:
            结果字典
        """
        print(f"📄 正在转换 PDF: {pdf_path}")
        
        if not os.path.exists(pdf_path):
            return {
                "success": False,
                "error": f"PDF 文件不存在: {pdf_path}"
            }
        
        try:
            # 使用 pdf2image 库
            images = convert_from_path(pdf_path, dpi=dpi)
            stitched_image = self.stitch_images(images, spacing=spacing)
            
            # 确定输出路径
            if output_path:
                output_path = Path(output_path)
            else:
                input_path = Path(pdf_path)
                output_path = input_path.parent / f"{input_path.stem}_stitched.png"
            
            # 保存图片
            print(f"💾 正在保存图片到: {output_path}")
            stitched_image.save(output_path, quality=95)
            
            # 获取文件大小
            file_size_mb = os.path.getsize(output_path) / (1024 * 1024)
            file_size_str = f"{file_size_mb:.2f} MB" if file_size_mb >= 1 else f"{file_size_mb * 1024:.0f} KB"
            
            return {
                "success": True,
                "output_path": str(output_path.absolute()),
                "file_size_mb": file_size_mb,
                "file_size_str": file_size_str,
                "pages": len(images),
                "width": images[0].width if images else 0,
                "height": sum([img.height for img in images]) + spacing * (len(images) - 1) if images else 0,
                "dpi": dpi,
                "spacing": spacing
            }
            
        except Exception as e:
            print(f"❌ 转换失败: {e}")
            return {
                "success": False,
                "error": str(e)
            }
    
    def convert_pdf_from_url(self, pdf_url, output_path=None, dpi=150, spacing=10):
        """
        从 URL 下载并转换 PDF
        
        Args:
            pdf_url: PDF 文件 URL
            output_path: 输出图片路径（可选）
            dpi: 转换 DPI，默认 150
            spacing: 图片间距（像素），默认 10
            
        Returns:
            结果字典
        """
        print(f"📥 正在下载 PDF: {pdf_url}")
        
        try:
            import requests
            
            # 下载 PDF
            response = requests.get(pdf_url, stream=True, timeout=30)
            response.raise_for_status()
            
            # 保存临时 PDF 文件
            temp_pdf = CURRENT_DIR / "temp.pdf"
            with open(temp_pdf, 'wb') as f:
                for chunk in response.iter_content(chunk_size=8192):
                    if chunk:
                        f.write(chunk)
            
            print(f"✅ PDF 下载完成: {temp_pdf}")
            
            # 转换 PDF
            result = self.convert_pdf(str(temp_pdf), output_path, dpi, spacing)
            
            # 删除临时文件
            os.remove(temp_pdf)
            print(f"🗑️ 临时文件已删除: {temp_pdf}")
            
            return result
            
        except Exception as e:
            print(f"❌ 下载或转换失败: {e}")
            return {
                "success": False,
                "error": str(e)
            }
    
    def batch_convert(self, pdf_dir, output_dir=None, dpi=150, spacing=10):
        """
        批量转换目录中的 PDF 文件
        
        Args:
            pdf_dir: PDF 文件目录
            output_dir: 输出图片目录（可选）
            dpi: 转换 DPI，默认 150
            spacing: 图片间距（像素），默认 10
            
        Returns:
            结果列表
        """
        print(f"📁 正在批量转换目录: {pdf_dir}")
        
        pdf_dir = Path(pdf_dir)
        if not pdf_dir.exists():
            return {
                "success": False,
                "error": f"目录不存在: {pdf_dir}"
            }
        
        # 查找所有 PDF 文件
        pdf_files = list(pdf_dir.glob("*.pdf"))
        
        if not pdf_files:
            return {
                "success": False,
                "error": f"目录中没有找到 PDF 文件: {pdf_dir}"
            }
        
        print(f"📊 找到 {len(pdf_files)} 个 PDF 文件")
        
        # 确定输出目录
        if output_dir:
            output_dir = Path(output_dir)
        else:
            output_dir = pdf_dir / "converted"
        
        output_dir.mkdir(exist_ok=True)
        
        # 批量转换
        results = []
        success_count = 0
        fail_count = 0
        
        for i, pdf_file in enumerate(pdf_files, 1):
            print(f"\n[{i}/{len(pdf_files)}] 处理: {pdf_file.name}")
            
            output_file = output_dir / f"{pdf_file.stem}_stitched.png"
            result = self.convert_pdf(str(pdf_file), str(output_file), dpi, spacing)
            
            results.append({
                "file": pdf_file.name,
                "result": result
            })
            
            if result.get('success'):
                success_count += 1
            else:
                fail_count += 1
        
        print(f"\n✅ 批量转换完成!")
        print(f"   成功: {success_count}")
        print(f"   失败: {fail_count}")
        
        return {
            "success": True,
            "total": len(pdf_files),
            "success_count": success_count,
            "fail_count": fail_count,
            "results": results
        }
    
    def stitch_images(self, images, spacing=10, bg_color=(255,255,255)):
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
    
    def get_skill_info(self):
        """
        获取 Skill 信息
        
        Returns:
            Skill 信息字典
        """
        return {
            "name": self.name,
            "version": self.version,
            "description": self.description,
            "author": "OpenClaw AI Assistant",
            "icon": "📄",
            "category": "工具",
            "tags": ["PDF", "图片", "转换", "文档", "幻灯片"],
            "language": "Python",
            "framework": "pdf2image, Pillow",
            "features": [
                "PDF 转换为图片",
                "图片纵向拼接",
                "自定义 DPI",
                "自定义图片间距",
                "批量转换",
                "URL 下载转换"
            ]
        }


def main():
    """主函数"""
    parser = argparse.ArgumentParser(
        description='PDF 转换为长图片 Skill',
        formatter_class=argparse.RawDescriptionHelpFormatter
    )
    
    # 单文件转换参数
    parser.add_argument('pdf_file', nargs='?', help='PDF 文件路径')
    parser.add_argument('-o', '--output', help='输出图片路径 (默认: input_stitched.png)')
    parser.add_argument('-d', '--dpi', type=int, default=150, help='转换 DPI (默认: 150)')
    parser.add_argument('-s', '--spacing', type=int, default=10, help='图片间距 (像素, 默认: 10)')
    
    # URL 转换参数
    parser.add_argument('-u', '--url', help='PDF 文件 URL')
    
    # 批量转换参数
    parser.add_argument('-b', '--batch', action='store_true', help='批量转换模式')
    parser.add_argument('--pdf-dir', help='PDF 文件目录 (批量转换模式)')
    parser.add_argument('--output-dir', help='输出图片目录 (批量转换模式)')
    
    # Skill 信息参数
    parser.add_argument('--skill-info', action='store_true', help='显示 Skill 信息')
    
    # 帮助参数
    parser.add_argument('-h', '--help', action='store_true', help='显示帮助信息')
    
    args = parser.parse_args()
    
    # 创建 Skill 实例
    skill = PDFConvertSkill()
    
    # 显示 Skill 信息
    if args.skill_info:
        info = skill.get_skill_info()
        print(json.dumps(info, ensure_ascii=False, indent=2))
        return
    
    # 显示帮助
    if args.help or (not args.pdf_file and not args.url):
        print(f"""
📄 PDF 转换为长图片 Skill v{skill.version}

{skill.description}

🚀 快速使用:
    # 单文件转换
    python main.py input.pdf
    
    # 自定义输出
    python main.py input.pdf -o output.png -d 200 -s 15
    
    # URL 下载转换
    python main.py -u https://example.com/document.pdf
    
    # 批量转换
    python main.py -b --pdf-dir ./pdfs --output-dir ./output

📋 选项:
    pdf_file             PDF 文件路径
    -o, --output         输出图片路径 (默认: input_stitched.png)
    -d, --dpi             转换 DPI (默认: 150)
    -s, --spacing         图片间距 (像素, 默认: 10)
    -u, --url             PDF 文件 URL
    -b, --batch           批量转换模式
    --pdf-dir             PDF 文件目录 (批量转换模式)
    --output-dir           输出目录 (批量转换模式)
    --skill-info           显示 Skill 信息
    -h, --help            显示此帮助信息

💡 使用示例:
    # 基本转换
    python main.py document.pdf
    
    # 高质量转换
    python main.py document.pdf -d 200 -s 15
    
    # URL 下载转换
    python main.py -u https://example.com/doc.pdf -o output.png
    
    # 批量转换
    python main.py -b --pdf-dir ./pdfs --output-dir ./output

📖 原工具: https://github.com/fisherhhyu/pdftool
🤖 Skill 作者: OpenClaw AI Assistant (lobster-shadow)
        """)
        return
    
    # 单文件转换
    if args.pdf_file and not args.batch:
        result = skill.convert_pdf(args.pdf_file, args.output, args.dpi, args.spacing)
        print(json.dumps(result, ensure_ascii=False, indent=2))
        return
    
    # URL 下载转换
    if args.url:
        result = skill.convert_pdf_from_url(args.url, args.output, args.dpi, args.spacing)
        print(json.dumps(result, ensure_ascii=False, indent=2))
        return
    
    # 批量转换
    if args.batch and args.pdf_dir:
        result = skill.batch_convert(args.pdf_dir, args.output_dir, args.dpi, args.spacing)
        print(json.dumps(result, ensure_ascii=False, indent=2))
        return
    
    print("❌ 请指定操作参数，使用 -h 查看帮助信息")


if __name__ == '__main__':
    main()
```

---

## 🛠️ Skill 集成

### 1. 集成到智能客服平台

这个 Skill 可以集成到任何支持自定义函数的智能客服平台：

- 腾讯云智能客服
- 百度智能客服
- 阿里云智能客服
- 字节跳动智能客服
- 其他支持自定义 Skill 的平台

### 2. 配置方式

#### 方式 A：通过平台界面配置
1. 登录智能客服平台
2. 选择"自定义技能"或"函数计算"
3. 配置以下信息：
   - **技能名称：** PDF 转换为长图片
   - **技能描述：** 将 PDF 文件转换并拼接为一张长图片，类似幻灯片效果
   - **入口文件：** main.py
   - **Python 版本：** 3.9+
   - **依赖包：** pdf2image, Pillow

#### 方式 B：通过配置文件配置
```json
{
  "skill_name": "PDF 转换为长图片",
  "skill_description": "将 PDF 文件转换并拼接为一张长图片",
  "python_version": "3.9+",
  "dependencies": "pdf2image, Pillow",
  "entry_point": "main",
  "max_file_size": "100MB",
  "timeout": "300"
}
```

---

## 📊 技术栈

```
🔧 核心技术：
├── Python 3.9+
├── pdf2image >= 1.16.0
├── Pillow >= 9.0.0
└── argparse (标准库)

🌐 Web 技术：
├── REST API
└── JSON 数据交换

📱 客户端：
├── 智能客服平台
├── 微信小程序
├── 企业微信
└── H5 网页
```

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
**🔖 版本：** v1.0.1
