---
name: thesis-writer
description: Use when writing, editing, or formatting a Chinese undergraduate graduation thesis (.docx) by copying a reference template and replacing only text content. Covers the complete workflow from template analysis to content replacement, diagram generation, screenshot capture, and quality verification.
---

<!-- Public export: deployment-specific identifiers or personal examples were replaced with placeholders on 2026-10-09. -->
# 对比模版写毕业论文 Skill

## 核心原则

**绝对铁律：复制模版 → 只替换文字 → 不动任何格式。**

绝对不要从空白文档创建，不要用 `doc.add_paragraph()` 添加新段落，不要修改样式属性。模版中的每一个段落、表格、图片占位都有精确的格式（字体、字号、行距、缩进、边框），只需替换其 `run.text` 内容即可。

## 第一阶段：模版分析

在写任何内容之前，必须先彻底分析模版文档。

### 1.1 提取模版结构

```python
from docx import Document

template = Document('模版.docx')

# 统计所有段落的样式分布
from collections import Counter
styles = Counter(p.style.name for p in template.paragraphs)
print("段落样式分布:", styles)

# 统计表格和图片数量
print(f"表格数: {len(template.tables)}")
print(f"内联图片数: {len(template.inline_shapes)}")
print(f"总段落数: {len(template.paragraphs)}")
print(f"节数: {len(template.sections)}")
```

### 1.2 建立段落映射表

遍历所有段落，记录每个段落的索引、样式名、文字内容前50字。这是后续替换的基础：

```python
for i, p in enumerate(template.paragraphs):
    if p.text.strip():
        print(f"Para {i} [{p.style.name}]: {p.text.strip()[:80]}")
```

**重点关注：**
- 封面段落（前70个）：样式名如 `外封基本信息`、`内封2基本信息`、`内封统计` 等
- 章节标题：`Heading 1`、`Heading 2`、`Heading 3`
- 正文段落：`论文正文`
- 图表标题：`Caption`
- 参考文献：`参考文献正文`
- 结束语/致谢：`结束语正文`、`致谢正文`

### 1.3 分析章节对应关系

列出模版中所有章节标题及其子标题数量，与你的论文章节做映射：

```
模版 Heading 1: 绪论 / 相关技术概述 / 系统需求分析 / 系统设计 / 系统实现 / 系统测试
模版 Heading 2 数量: X个
模版 Heading 3 数量: X个
Caption 数量: X个
论文正文段落: X个
```

**映射原则：**
- 论文的章节数必须和模版一致（同样的 Heading 1/2/3 数量）
- 如果你的功能模块数和模版不同，通过合并或拆分内容来匹配段落数
- 不能添加多余段落，也不能删除段落（除非删除已确认无用的旧内容）

### 1.4 分析表格结构

```python
for ti, table in enumerate(template.tables):
    print(f"\nTable {ti}: {len(table.rows)}行 x {len(table.columns)}列")
    for ri, row in enumerate(table.rows):
        cells = [cell.text.strip()[:30] for cell in row.cells]
        print(f"  Row {ri}: {cells}")
```

**记录每个表格的用途**（用例表、数据库表、测试用例表）和字段名模式。

## 第二阶段：文字替换

### 2.1 替换脚本模式

```python
import shutil
from docx import Document

# 第一步：复制模版
shutil.copy('模版.docx', '输出.docx')
doc = Document('输出.docx')

# 第二步：构建替换映射
# key = 段落索引, value = 新文字内容
replacements = {
    # 封面
    13: '云计算技术应用',      # 专业
    14: '示例用户',              # 学生
    15: '示例教师',                # 指导教师
    # ... 正文段落
    178: '新的段落内容...',
}

# 第三步：逐段替换（只改 run.text）
for idx, new_text in replacements.items():
    para = doc.paragraphs[idx]
    if para.runs:
        # 保留第一个 run 的格式，清空其余 run
        para.runs[0].text = new_text
        for run in para.runs[1:]:
            run.text = ''
    else:
        # 无 run 的段落，添加一个 run
        para.add_run(new_text)

doc.save('输出.docx')
```

### 2.2 封面替换的特殊处理

#### 下划线字段（如"专业：___云计算___"）

封面中带下划线的字段，模版使用字符样式 `内封2基本信息文字`（underline=True）。替换时必须保持"标签无下划线 + 内容有下划线"的结构：

```python
def replace_cover_field(para, label, content, total_width=20):
    """替换封面下划线字段
    
    Args:
        para: 段落对象
        label: 标签文字（如"专    业：\t"），不带下划线
        content: 内容文字（如"云计算技术应用"），带下划线
        total_width: 下划线总宽度（用空格填充）
    """
    # 清空所有现有 run
    for run in para.runs:
        run.text = ''
    
    # Run 1: 标签（无下划线）
    if para.runs:
        r1 = para.runs[0]
    else:
        r1 = para.add_run()
    r1.text = label
    r1.underline = False
    
    # Run 2: 内容（有下划线，居中填充空格）
    padded = content.center(total_width)
    if len(para.runs) > 1:
        r2 = para.runs[1]
    else:
        r2 = para.add_run()
    r2.text = padded
    r2.underline = True
    
    # 确保下划线格式来自字符样式
    from docx.oxml.ns import qn
    from docx.oxml import OxmlElement
    rPr = r2._element.get_or_add_rPr()
    u_elem = rPr.find(qn('w:u'))
    if u_elem is None:
        u_elem = OxmlElement('w:u')
        rPr.append(u_elem)
    u_elem.set(qn('w:val'), 'single')
```

**关键教训：** 如果直接替换整个段落的 `runs[0].text` 并清空其余 run，会丢失下划线格式，因为下划线是在特定 run 上设置的，而不是段落样式。必须拆分为"标签 run"和"内容 run"分别处理。

**下划线对齐：** 所有封面字段的下划线长度需一致。用 `str.center(total_width)` 统一填充。

#### 统计数字（总页数、表格数、图片数）

```python
# 封面统计占位符替换
for para in doc.paragraphs[:70]:
    for run in para.runs:
        if 'XX页' in run.text:
            run.text = run.text.replace('XX页', '63页')
        if 'XX个' in run.text and '表' in para.text:
            run.text = run.text.replace('XX个', '28个')
        if 'XX个' in run.text and '图' in para.text:
            run.text = run.text.replace('XX个', '49个')
```

**注意：** 页数需要在 WPS 中打开后从状态栏确认实际页数。

### 2.3 页眉替换

```python
for section in doc.sections:
    header = section.header
    for para in header.paragraphs:
        if para.text.strip():
            for run in para.runs:
                run.text = run.text.replace('模版论文标题', '你的论文标题')
```

### 2.4 表格内容替换

```python
table = doc.tables[table_index]
for ri, row in enumerate(table.rows):
    for ci, cell in enumerate(row.cells):
        # 替换单元格文字
        # 注意：一个单元格可能有多个段落！
        for para in cell.paragraphs:
            if para.runs:
                para.runs[0].text = new_content
                for run in para.runs[1:]:
                    run.text = ''
```

#### 用例表字段名匹配

**致命教训：** 模版中的用例表字段名可能和你预想的不同。不要猜测字段名，必须先读取模版中的实际字段名：

```
模版实际字段: 用例编号 | 用例名称 | 参与者 | 用例描述 | 前置事件 | 基本事件流 | 异常事件流 | 后置条件
常见错误字段: 用例编号 | 用例名称 | 参与者 | 前置条件 | 后置条件 | 基本流程 | 扩展流程 | 特殊需求
```

必须使用模版中的实际字段名，而非自己编造。

#### 表格旧内容清理

**致命教训：** 模版表格的单元格中可能有多个段落（`cell.paragraphs`）。替换第一个段落后，必须删除多余的旧段落：

```python
cell = table.rows[5].cells[1]  # 基本事件流单元格
# 替换第一个段落
cell.paragraphs[0].runs[0].text = "新内容"
# 删除多余的旧段落（从后往前删）
for i in range(len(cell.paragraphs) - 1, 0, -1):
    p_element = cell.paragraphs[i]._element
    p_element.getparent().remove(p_element)
```

不清理多余段落会导致新旧内容混在一起（如"1.学生进入AI问答\n点击菜品，点击'我想吃'"这种混合内容）。

### 2.5 Caption（图表标题）替换

**致命教训：** Caption 必须和正文引用一一对应。正文写"如表3-3所示"，则表3-3的 Caption 必须和正文描述匹配。

**检查方法：**

```python
import re
for i, p in enumerate(doc.paragraphs):
    # 找正文中的引用
    if p.style.name == '论文正文':
        m = re.search(r'如(表\d+-\d+)所示', p.text)
        if m:
            table_ref = m.group(1)
            # 找到正文说这个表是什么
            m2 = re.search(r'(\S+用例描述)如表', p.text)
            if m2:
                expected_name = m2.group(1)
                print(f"正文引用: {table_ref} = {expected_name}")
    
    # 找对应的 Caption
    if p.style.name == 'Caption':
        print(f"Caption: {p.text.strip()}")
```

**错误示例：**
- 正文说"公告管理用例描述如表3-3所示"
- Caption 写的是"表3-3 AI配置管理用例描述"
- 这种不匹配会被老师直接指出

### 2.6 参考文献替换

**教训：** 如果模版的参考文献使用了自动编号列表样式（`参考文献正文`），则替换文字时不要在文本开头加 `[1]` 前缀，否则会出现双重编号（自动编号 + 手写编号）。

```python
# ❌ 错误
run.text = "[1] 韦彩仁,张锐杭.基于Vue 3..."

# ✅ 正确
run.text = "韦彩仁,张锐杭.基于Vue 3..."
```

### 2.7 章节小结一致性

**教训：** 每章"本章小结"段落的内容必须和该章的实际内容一致。例如第2章介绍了 MySQL、Spring Boot、Vue、Redis 四个技术，小结就不能提到 Docker（如果没有 Docker 小节的话）。

## 第三阶段：图片替换

### 3.1 替换已有图片

论文中的图片是作为 `inline_shapes` 存在的。替换逻辑：

```python
from docx.shared import Cm, Emu
from docx.opc.constants import RELATIONSHIP_TYPE as RT
import os

def replace_inline_image(doc, shape_index, new_image_path, width_cm=14):
    """替换指定索引的内联图片"""
    shape = doc.inline_shapes[shape_index]
    
    # 获取原始尺寸用于保持宽高比
    blip = shape._element.find('.//' + qn('a:blip'))
    if blip is None:
        return False
    
    # 添加新图片关系
    rId = doc.part.relate_to(
        doc.part.package.parts[doc.part.rels[blip.get(qn('r:embed'))].target_partname],
        RT.IMAGE
    )
    
    # 更简单的方法：直接替换图片数据
    old_rId = blip.get(qn('r:embed'))
    image_part = doc.part.rels[old_rId].target_part
    
    with open(new_image_path, 'rb') as f:
        image_part._blob = f.read()
    
    # 调整尺寸
    from PIL import Image
    img = Image.open(new_image_path)
    aspect = img.height / img.width
    new_width = Cm(width_cm)
    new_height = int(new_width * aspect)
    shape.width = new_width
    shape.height = new_height
    
    return True
```

### 3.1.1 图片尺寸必须与模版一致（铁律）

**每张图片替换后，必须先读取模版中对应图片的尺寸，再设置论文中图片的尺寸。** 不要凭感觉设置尺寸，不要用固定值，必须用脚本读取模版的实际 cx/cy 值。

**完整流程：替换图片 = 替换数据 + 匹配模版尺寸，缺一不可。**

```python
def get_template_image_size(template_doc, caption_keyword):
    """从模版中读取指定 Caption 对应图片的尺寸"""
    ns = {'w': 'http://schemas.openxmlformats.org/wordprocessingml/2006/main',
          'wp': 'http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing',
          'v': 'urn:schemas-microsoft-com:vml'}
    
    for i, para in enumerate(template_doc.paragraphs):
        if 'Caption' in para.style.name and caption_keyword in para.text:
            # 向前搜索图片（最多5段）
            for ci in range(max(0, i-5), i):
                p = template_doc.paragraphs[ci]
                # 方式1：inline drawing（常见）
                drawings = p._element.findall('.//w:drawing', ns)
                if drawings:
                    for d in drawings:
                        for ext in d.findall('.//wp:extent', ns):
                            return int(ext.get('cx')), int(ext.get('cy'))
                # 方式2：嵌入式 OLE 对象（EMF等）
                objects = p._element.findall('.//w:object', ns)
                if objects:
                    for obj in objects:
                        shapes = obj.findall('.//v:shape', ns)
                        for s in shapes:
                            import re
                            style = s.get('style', '')
                            w_m = re.search(r'width:([\d.]+)pt', style)
                            h_m = re.search(r'height:([\d.]+)pt', style)
                            if w_m and h_m:
                                # pt → EMU: 1pt = 12700 EMU
                                return int(float(w_m.group(1)) * 12700), int(float(h_m.group(1)) * 12700)
    return None, None

# 使用示例
template = Document('模版.docx')
tpl_cx, tpl_cy = get_template_image_size(template, '系统E-R')
# tpl_cx, tpl_cy 就是模版中该图片的精确尺寸（EMU单位）

# 替换论文图片时：按模版尺寸的宽度或高度等比缩放
from PIL import Image
img = Image.open('new_image.png')
ratio = img.size[1] / img.size[0]  # PNG 像素高宽比

# 优先匹配模版宽度，高度按PNG比例计算
target_cx = tpl_cx
target_cy = int(tpl_cx * ratio)
# 如果计算出的高度超过模版高度太多，则改为匹配高度
if target_cy > tpl_cy * 1.3:
    target_cy = tpl_cy
    target_cx = int(tpl_cy / ratio)
```

**注意事项：**
- 模版中的 EMF 图片用 `w:object > v:shape` 存储尺寸（单位是 pt），需转换为 EMU
- 模版中的 PNG/JPG 图片用 `wp:extent` 存储尺寸（单位是 EMU）
- 同类型的图片（如所有用例图、所有ER属性图）尺寸应一致，只需读取模版中一张的尺寸即可
- **绝对不要跳过这一步**，否则每次都会因为尺寸不匹配被打回

### 3.2 建立图片位置映射

先遍历找出所有图片在文档中的位置和对应的 Caption：

```python
from lxml import etree
ns = {'w': 'http://schemas.openxmlformats.org/wordprocessingml/2006/main',
      'a': 'http://schemas.openxmlformats.org/drawingml/2006/main',
      'r': 'http://schemas.openxmlformats.org/officeDocument/2006/relationships'}

shape_idx = 0
for i, para in enumerate(doc.paragraphs):
    # 检查段落中是否有内联图片
    drawings = para._element.findall('.//w:drawing', ns)
    if drawings:
        # 找到下一个 Caption 段落
        caption = ""
        for j in range(i+1, min(i+3, len(doc.paragraphs))):
            if doc.paragraphs[j].style.name == 'Caption':
                caption = doc.paragraphs[j].text.strip()
                break
        print(f"Shape {shape_idx} at Para {i}: {caption}")
        shape_idx += len(drawings)
```

### 3.3 图片生成规范

#### matplotlib 全局配置

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

plt.rcParams.update({
    'font.sans-serif': ['Heiti TC', 'STHeiti', 'SimHei'],
    'font.serif': ['Songti SC', 'STSong', 'SimSun'],
    'axes.unicode_minus': False,
    'figure.dpi': 300,
    'savefig.dpi': 300,
    'savefig.bbox': 'tight',
    'savefig.pad_inches': 0.1,
})
```

#### 颜色规范
- 全部黑白灰（black, white, #f0f0f0, #e0e0e0）
- 不使用彩色，符合学术论文规范

#### 用例图绘制

**致命教训：** matplotlib 画的用例图和 Visio/ProcessOn 画的差别很大。要点：

1. Actor（参与者）画成简笔人形 stick figure，放在图的左侧
2. 用例椭圆要足够大，文字要清晰
3. 每张用例图只画一个角色的用例
4. 系统边界用虚线矩形框
5. 连接线从 Actor 到椭圆

```python
def draw_usecase(actor_name, usecases, filename, figsize=(10, 8)):
    fig, ax = plt.subplots(1, 1, figsize=figsize)
    ax.set_xlim(0, 10)
    ax.set_ylim(0, len(usecases) + 2)
    ax.set_aspect('equal')
    ax.axis('off')
    
    # 画 Actor（简笔人形）
    actor_x, actor_y = 1.5, (len(usecases) + 1) / 2
    # 头
    circle = plt.Circle((actor_x, actor_y + 0.6), 0.25, fill=False, color='black', lw=1.5)
    ax.add_patch(circle)
    # 身体
    ax.plot([actor_x, actor_x], [actor_y + 0.35, actor_y - 0.3], 'k-', lw=1.5)
    # 手臂
    ax.plot([actor_x - 0.35, actor_x + 0.35], [actor_y + 0.1, actor_y + 0.1], 'k-', lw=1.5)
    # 腿
    ax.plot([actor_x, actor_x - 0.3], [actor_y - 0.3, actor_y - 0.8], 'k-', lw=1.5)
    ax.plot([actor_x, actor_x + 0.3], [actor_y - 0.3, actor_y - 0.8], 'k-', lw=1.5)
    # 角色名
    ax.text(actor_x, actor_y - 1.1, actor_name, ha='center', fontsize=11)
    
    # 系统边界框
    from matplotlib.patches import FancyBboxPatch
    border = FancyBboxPatch((3.5, 0.3), 5.5, len(usecases) + 0.8,
                            boxstyle="round,pad=0.2", fill=False,
                            edgecolor='black', linestyle='--', lw=1.2)
    ax.add_patch(border)
    ax.text(6.2, len(usecases) + 1.3, '系统', fontsize=12, ha='center')
    
    # 画用例椭圆
    for idx, uc in enumerate(usecases):
        cy = len(usecases) - idx
        ellipse = Ellipse((6.2, cy), 3.2, 0.7, fill=False, edgecolor='black', lw=1.2)
        ax.add_patch(ellipse)
        ax.text(6.2, cy, uc, ha='center', va='center', fontsize=10)
        # 连接线
        ax.annotate('', xy=(4.6, cy), xytext=(actor_x + 0.4, actor_y),
                    arrowprops=dict(arrowstyle='-', color='black', lw=0.8))
    
    plt.savefig(filename, dpi=300, bbox_inches='tight')
    plt.close()
```

#### ER 实体属性图（概念设计）

**致命教训：** 第4章数据库概念设计中的 ER 图，模版用的是**实体-属性图**（一个实体矩形 + 周围一圈属性椭圆），不是关系型 ER 图（多个实体之间的连线）。每个实体单独一张图。

```python
def draw_entity_attributes(entity_name, attributes, filename):
    """绘制单个实体的属性图（中心实体 + 周围属性椭圆）"""
    n = len(attributes)
    fig, ax = plt.subplots(figsize=(10, 8))
    ax.set_xlim(-5, 5)
    ax.set_ylim(-5, 5)
    ax.set_aspect('equal')
    ax.axis('off')
    
    # 中心实体矩形
    rect = FancyBboxPatch((-1.5, -0.5), 3, 1, boxstyle="round,pad=0.1",
                          facecolor='#f0f0f0', edgecolor='black', lw=2)
    ax.add_patch(rect)
    ax.text(0, 0, entity_name, ha='center', va='center', fontsize=14, fontweight='bold')
    
    # 属性椭圆均匀分布在周围
    import math
    radius = 3.5
    for i, attr in enumerate(attributes):
        angle = 2 * math.pi * i / n - math.pi / 2
        x = radius * math.cos(angle)
        y = radius * math.sin(angle)
        
        ellipse = Ellipse((x, y), 2.2, 0.8, fill=True,
                          facecolor='white', edgecolor='black', lw=1.2)
        ax.add_patch(ellipse)
        ax.text(x, y, attr, ha='center', va='center', fontsize=9)
        
        # 连接线到中心
        ax.plot([0, x], [0, y], 'k-', lw=0.8)
    
    plt.savefig(filename, dpi=300, bbox_inches='tight')
    plt.close()
```

#### 流程图绘制

**要点：** 
- 开始/结束: 圆角矩形，灰色填充
- 处理步骤: 矩形，白色填充
- 判断: 菱形
- 箭头: 从上到下流动，判断分支向左右分叉

#### 功能结构图

树形层次结构，根节点为系统名，二级为角色/模块大类，三级为具体功能。

### 3.4 系统截图

使用 Docker 启动系统后，通过 Playwright 无头浏览器截图：

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={'width': 1280, 'height': 800})
    
    # 登录
    page.goto('http://localhost:8888')
    page.fill('input[placeholder*="用户名"]', 'admin')
    page.fill('input[placeholder*="密码"]', 'admin123')
    page.click('button:has-text("登录")')
    page.wait_for_load_state('networkidle')
    
    # 截图
    page.screenshot(path='thesis_images/screenshot_dashboard.jpeg', 
                    type='jpeg', quality=85)
    
    browser.close()
```

**截图注意事项：**
- 所有截图尺寸保持一致（viewport 1280x800）
- 使用 JPEG 格式减小文件体积
- 截图前等待 `networkidle` 确保页面加载完成
- 需要先通过 API 创建测试数据（用户、文章、问题等）

## 第四阶段：质量检查

### 4.1 模版残留检查

替换完成后，必须搜索模版中的特征词确保无遗漏：

```python
old_keywords = ['模版作者名', '模版系统名', '模版特有功能名']  
# 如: ['韩梦婷', '点餐', '菜品', '轮播', '我想吃', '订单管理']

found = False
for i, p in enumerate(doc.paragraphs):
    if p.style.name.startswith('toc'):
        continue  # 目录条目跳过（需在WPS中更新）
    for kw in old_keywords:
        if kw in p.text:
            print(f"❌ Para {i} [{p.style.name}]: 发现'{kw}'")
            found = True

# 同时检查表格
for ti, table in enumerate(doc.tables):
    for ri, row in enumerate(table.rows):
        for ci, cell in enumerate(row.cells):
            for kw in old_keywords:
                if kw in cell.text:
                    print(f"❌ Table {ti} R{ri}C{ci}: 发现'{kw}'")
                    found = True

if not found:
    print("✅ 无模版残留内容")
```

### 4.2 Caption 与正文交叉验证

```python
import re
for i, p in enumerate(doc.paragraphs):
    if p.style.name == '论文正文':
        # 查找"如表X-X所示"或"如图X-X所示"的引用
        refs = re.findall(r'如((?:表|图)\d+-\d+)所示', p.text)
        for ref in refs:
            # 查找对应的 Caption
            for j, cp in enumerate(doc.paragraphs):
                if cp.style.name == 'Caption' and ref in cp.text:
                    # 验证 Caption 名称和正文描述匹配
                    break
```

### 4.3 占位符检查

```python
for p in doc.paragraphs:
    if 'XX' in p.text and not p.style.name.startswith('toc'):
        print(f"❌ 未替换的占位符: {p.text.strip()[:80]}")
```

### 4.4 格式一致性验证

```python
template = Document('模版.docx')
thesis = Document('输出.docx')

# 1. 页面设置
for i, (ts, ds) in enumerate(zip(template.sections, thesis.sections)):
    assert ts.page_width == ds.page_width, f"节{i}纸张宽度不一致"
    assert ts.top_margin == ds.top_margin, f"节{i}上边距不一致"
    assert ts.left_margin == ds.left_margin, f"节{i}左边距不一致"

# 2. 样式数量
from collections import Counter
t_styles = Counter(p.style.name for p in template.paragraphs)
d_styles = Counter(p.style.name for p in thesis.paragraphs)
for s in set(list(t_styles.keys()) + list(d_styles.keys())):
    diff = abs(t_styles.get(s, 0) - d_styles.get(s, 0))
    if diff > 5:
        print(f"⚠️ 样式'{s}'数量差异过大: 模版{t_styles.get(s,0)} vs 论文{d_styles.get(s,0)}")

# 3. 封面段落样式（前70段必须完全一致）
for i in range(min(70, len(template.paragraphs), len(thesis.paragraphs))):
    assert template.paragraphs[i].style.name == thesis.paragraphs[i].style.name, \
        f"封面Para {i}样式不一致: {template.paragraphs[i].style.name} vs {thesis.paragraphs[i].style.name}"
```

### 4.5 统计数字

```python
fig_count = sum(1 for p in doc.paragraphs if p.style.name == 'Caption' and p.text.strip().startswith('图'))
tab_count = sum(1 for p in doc.paragraphs if p.style.name == 'Caption' and p.text.strip().startswith('表'))
ref_count = sum(1 for p in doc.paragraphs if p.style.name == '参考文献正文' and p.text.strip())
print(f"图片: {fig_count}个, 表格: {tab_count}个, 参考文献: {ref_count}条")
```

## 第五阶段：手动收尾（WPS中操作）

以下操作无法通过 python-docx 完成，需要在 WPS 中手动操作：

1. **更新目录** — 右键点击目录 → 更新域 → 更新整个目录
2. **确认页数** — 查看 WPS 右下角状态栏的实际页数，回填封面统计
3. **检查分页** — 确保章节标题不出现在页面底部（孤行）
4. **检查页眉** — 正文页眉是否显示论文标题

## 致命教训清单（按发现顺序）

### 格式类

| # | 教训 | 后果 | 解决方案 |
|---|------|------|----------|
| 1 | 从空白文档创建论文 | 所有格式都不对，老师一眼看出 | 必须复制模版 |
| 2 | 使用 `doc.add_paragraph()` 添加段落 | 段落数不匹配，排版错乱 | 只替换已有段落的 run.text |
| 3 | 封面下划线字段直接替换 text | 下划线消失 | 拆分为标签 run + 内容 run |
| 4 | 下划线长度不统一 | 封面不整齐 | 用 str.center(total_width) 统一填充 |
| 5 | 表格样式不匹配（全网格 vs 三线表） | 和模版明显不同 | 复制模版表格的 tblPr/tcPr XML |
| 6 | 用例表字段名不对 | "前置条件"应为"用例描述" | 先读取模版表格的实际字段名 |
| 7 | 参考文献加 [X] 前缀 | 双重编号 | 不加前缀，依赖自动编号 |
| 8 | 模版续表的分页符 | 产生大片空白 | 删除不需要的分页符 |

### 内容类

| # | 教训 | 后果 | 解决方案 |
|---|------|------|----------|
| 9 | Caption 和正文引用不匹配 | "表3-3 公告管理"但 Caption 写"AI配置管理" | 交叉验证所有引用 |
| 10 | 章节小结和实际内容不一致 | "小结"提到了第2章没有的 Docker | 小结逐条核对本章实际内容 |
| 11 | 表格单元格旧内容未清理 | 新旧内容混合出现 | 替换后删除多余段落 |
| 12 | 技术映射错位（5个技术塞4个槽） | Redis段落出现Docker内容 | 严格按段落位置一对一映射 |
| 13 | 图表数量统计错误 | 封面数字不准确 | 用脚本自动计数 |

### 图片类

| # | 教训 | 后果 | 解决方案 |
|---|------|------|----------|
| 14 | matplotlib 用例图和 Visio 风格差距大 | 老师不认可 | 大椭圆+单角色+比例匹配 |
| 15 | ER图画成关系型而非实体属性型 | 和模版不是同一种图 | 模版是什么类型就画什么类型 |
| 16 | 登录截图和其他截图大小不一致 | 视觉不统一 | 统一 viewport 尺寸 |
| 17 | 截图时系统未启动完全 | 页面空白或报错 | 等待 networkidle + 创建测试数据 |
| 18 | 替换图片后未调整尺寸 | 图片太小或太大，和模版不一致 | 必须按**原图像素比例等比缩放**，用目标高度反算宽度（见下方规则） |
| 19 | draw.io 导出 scale 太低 | 图片在 Word 中不清晰 | 必须用 `--scale 3` 或更高，不要用 `--scale 2` |
| 20 | 流程图 XML 比例太纵向 | 插入 Word 后图太高占整页 | 元素宽度设 250px，垂直间距 70px，菱形更宽扁(250x80)，目标宽高比 1.8:1 |
| 21 | 功能结构图布局和模版不一致 | 横向树形结构和模版的上下结构完全不同 | 必须用**上下结构**：学生功能在上→学生→系统名→管理员→管理员功能在下。功能框用竖排文字（`&lt;br&gt;`逐字换行），所有框统一宽42px高120px，总宽600px均匀分布，导出后设Word宽度13.5cm撑满页面 |
| 22 | 用例图系统边界写标题文字 | 多余标签和模版不一致 | 系统边界用空白矩形 `value=""`，不写"XX子系统" |
| 23 | 开始/结束框用灰色填充 | 和模版不一致（模版全白） | 所有形状统一 `fillColor=#FFFFFF` |
| 24 | 图片尺寸凭感觉设置 | 和模版大小不一致，每次都要返工 | **必须先用脚本读取模版中对应图片的实际尺寸（cx/cy），再按模版宽度等比缩放**，见 §3.1.1 |

## 完整工作流程总结

```
1. 分析模版
   ├── 提取段落结构映射
   ├── 提取表格结构
   ├── 提取图片位置
   └── 记录所有样式名

2. 规划内容映射
   ├── 章节标题一一对应
   ├── 正文段落数量匹配
   ├── 表格数量匹配
   └── 图片数量匹配

3. 生成素材
   ├── matplotlib 画图（用例图、流程图、ER图、功能结构图）
   ├── Docker 启动系统
   ├── Playwright 截图
   └── 图片存入 thesis_images/

4. 执行替换
   ├── 复制模版 → 输出文件
   ├── 替换封面信息（注意下划线）
   ├── 替换页眉
   ├── 替换正文段落 run.text
   ├── 替换表格内容（清理旧段落）
   ├── 替换 Caption 标题
   ├── 替换图片（inline_shapes）
   └── 替换参考文献（不加编号前缀）

5. 质量检查
   ├── 模版残留关键词搜索
   ├── Caption 与正文交叉验证
   ├── 占位符检查
   ├── 格式一致性验证
   ├── 统计数字（图/表/页/参考文献）
   └── 封面信息完整性

6. WPS 手动收尾
   ├── 更新目录
   ├── 确认页数
   └── 检查分页和页眉
```

## draw.io MCP 画图 + CLI 导出 + 自动插入论文（推荐）

### 完整自动化流程

```
draw.io XML 代码 → 保存为 .drawio 文件 → drawio CLI 导出 PNG → python-docx 替换论文图片
```

这是最可靠的论文图表生成方案，全流程自动化，无需手动操作。

### 工具配置

**draw.io MCP**：`npx @drawio/mcp`，提供三个工具：
- `mcp__drawio__open_drawio_xml` — 用 draw.io XML 生成图表（推荐，可预览）
- `mcp__drawio__open_drawio_mermaid` — 用 Mermaid 语法生成图表
- `mcp__drawio__open_drawio_csv` — 用 CSV 格式生成图表

**draw.io 桌面版 CLI**（已安装）：`/opt/homebrew/bin/drawio`
- 用途：将 .drawio 文件导出为 PNG/SVG/PDF
- MCP 工具**没有导出功能**，只能在浏览器中打开编辑器，必须依赖 CLI 导出

### 第一步：生成 draw.io XML 并保存为 .drawio 文件

```python
xml_content = '''<mxGraphModel>
  <root>
    <mxCell id="0"/><mxCell id="1" parent="0"/>
    <!-- 图表内容 -->
  </root>
</mxGraphModel>'''

with open('thesis_images/diagram.drawio', 'w') as f:
    f.write(xml_content)
```

### 第二步：用 CLI 导出为 PNG

```bash
drawio -x -f png -o thesis_images/diagram.png --scale 2 thesis_images/diagram.drawio
```

参数说明：
- `-x` 导出模式
- `-f png` 输出格式（支持 png/jpg/svg/pdf）
- `--scale 2` 2倍缩放，确保清晰度
- `-o` 输出文件路径

### 第三步：用 python-docx 替换论文中的图片

```python
from docx import Document
from lxml import etree

ns = {'w': 'http://schemas.openxmlformats.org/wordprocessingml/2006/main',
      'a': 'http://schemas.openxmlformats.org/drawingml/2006/main',
      'r': 'http://schemas.openxmlformats.org/officeDocument/2006/relationships'}

doc = Document('论文.docx')

# 遍历找到目标图片（按序号）
img_count = 0
for para in doc.paragraphs:
    for run in para.runs:
        drawings = run._element.findall('.//w:drawing', ns)
        for drawing in drawings:
            img_count += 1
            if img_count == TARGET_INDEX:  # 替换目标图片
                blips = drawing.findall('.//a:blip', ns)
                for blip in blips:
                    rId = blip.get('{http://schemas.openxmlformats.org/officeDocument/2006/relationships}embed')
                    if rId:
                        rel = run.part.rels[rId]
                        with open('thesis_images/diagram.png', 'rb') as f:
                            rel.target_part._blob = f.read()

doc.save('论文.docx')
```

### 第四步（关键）：调整图片显示尺寸匹配模版

**绝对铁律：必须按原图像素比例等比缩放，绝不能随意设宽高！**

错误做法：直接设 `cx=13.5cm, cy=7.6cm`（会拉伸变形）
正确做法：读取 PNG 原图像素比例，用目标高度反算宽度（或用目标宽度算高度）

**尺寸控制原则（按高度控制）：**
- 用例图、流程图、系统总体ER图：**目标高度 7.6cm**（和模版一致），宽度按原图比例自动算
- ER 实体属性图：**目标高度 5.0cm**
- 功能结构图：**目标宽度 12cm**（横向图按宽度控制）
- 界面截图/系统截图：保持原尺寸不动

```python
from PIL import Image

def cm_to_emu(cm):
    return int(cm / 2.54 * 914400)

# 按目标高度等比缩放（保持原图比例）
def resize_by_height(drawing, png_path, target_h_cm, ns_wp):
    img = Image.open(png_path)
    ratio = img.size[1] / img.size[0]  # 原图真实高宽比
    target_cy = cm_to_emu(target_h_cm)
    target_cx = int(target_cy / ratio)  # 按比例算宽度
    _apply_size(drawing, target_cx, target_cy, ns_wp)

# 按目标宽度等比缩放（保持原图比例）
def resize_by_width(drawing, png_path, target_w_cm, ns_wp):
    img = Image.open(png_path)
    ratio = img.size[1] / img.size[0]
    target_cx = cm_to_emu(target_w_cm)
    target_cy = int(target_cx * ratio)
    _apply_size(drawing, target_cx, target_cy, ns_wp)

def _apply_size(drawing, cx, cy, ns_wp):
    for ext in drawing.findall('.//wp:extent', ns_wp):
        ext.set('cx', str(cx))
        ext.set('cy', str(cy))
    a_ns = 'http://schemas.openxmlformats.org/drawingml/2006/main'
    for a_ext in drawing.findall('.//{%s}ext' % a_ns):
        if a_ext.get('cx') and int(a_ext.get('cx')) > 100000:
            a_ext.set('cx', str(cx))
            a_ext.set('cy', str(cy))
```

### 批量自动化脚本模板

```python
import subprocess, shutil
from docx import Document

# 1. 定义所有图表的 XML 和对应的论文图片序号
diagrams = {
    1: ('admin_usecase', ADMIN_USECASE_XML),
    2: ('student_usecase', STUDENT_USECASE_XML),
    3: ('flow_audit', FLOW_AUDIT_XML),
    # ... 更多图表
}

# 2. 批量生成 .drawio 并导出 PNG
for idx, (name, xml) in diagrams.items():
    drawio_path = f'thesis_images/{name}.drawio'
    png_path = f'thesis_images/{name}.png'
    with open(drawio_path, 'w') as f:
        f.write(xml)
    subprocess.run(['drawio', '-x', '-f', 'png', '-o', png_path, '--scale', '2', drawio_path], check=True)

# 3. 备份论文并批量替换图片
shutil.copy2('论文.docx', '论文-backup.docx')
doc = Document('论文.docx')
# ... 遍历替换（同上）
doc.save('论文.docx')
```

### draw.io XML 风格规范（经验教训）

#### 通用规则
- **所有形状白色填充**：`fillColor=#FFFFFF`，不要用灰色（`#E6E6E6`）
- **黑色描边**：`strokeColor=#000000`
- **宋体字**：`fontFamily=SimSun`
- **字号 11-12pt**：`fontSize=11` 或 `fontSize=12`

#### 用例图
- 系统边界用**空白矩形**（`rounded=0;fillColor=none;`），**不要写系统名称**（如"管理员子系统"），只画边框
- Actor 用 `shape=umlActor`，放在左侧
- 用例椭圆用 `ellipse;fillColor=#FFFFFF;`
- 连线**无箭头**：`endArrow=none;`
- 每个角色单独一张图

```xml
<!-- 系统边界（无标题空矩形） -->
<mxCell id="2" value="" style="rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#000000;" vertex="1" parent="1">
  <mxGeometry x="200" y="20" width="280" height="560" as="geometry"/>
</mxCell>
<!-- Actor -->
<mxCell id="3" value="管理员" style="shape=umlActor;verticalLabelPosition=bottom;verticalAlign=top;html=1;outlineConnect=0;fontSize=12;fontFamily=SimSun;strokeColor=#000000;fillColor=#FFFFFF;" vertex="1" parent="1">
  <mxGeometry x="60" y="240" width="40" height="60" as="geometry"/>
</mxCell>
<!-- 用例椭圆 -->
<mxCell id="10" value="用户信息管理" style="ellipse;whiteSpace=wrap;html=1;fontSize=11;fontFamily=SimSun;strokeColor=#000000;fillColor=#FFFFFF;" vertex="1" parent="1">
  <mxGeometry x="260" y="40" width="160" height="40" as="geometry"/>
</mxCell>
<!-- 连线（无箭头） -->
<mxCell id="30" style="endArrow=none;strokeColor=#000000;" edge="1" source="3" target="10" parent="1">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
```

#### 流程图
- **所有形状白色填充**（包括开始/结束），不要灰色
- **宽高比 1.8:1**（宽 > 高），元素宽度 250px，垂直间距 70px
- 开始/结束: `rounded=1;fillColor=#FFFFFF;arcSize=50;` 宽 250px 高 45px
- 处理步骤: `rounded=0;fillColor=#FFFFFF;` 宽 250px 高 45px
- 判断菱形: `rhombus;fillColor=#FFFFFF;` 宽 250px 高 80px（更宽扁，不要正方形）
- 箭头连线: `endArrow=block;endFill=1;`
- N 分支从菱形右侧出去，向上回到输入步骤（形成回路）
- Y 分支从菱形下方出去
- **导出用 `--scale 3`**，不要用 2

```xml
<!-- N 回路的画法（三段折线） -->
<!-- 1. 菱形右侧出去 -->
<mxCell style="endArrow=block;endFill=1;exitX=1;exitY=0.5;" edge="1" source="diamond">
  <mxGeometry><mxPoint x="380" y="345" as="targetPoint"/></mxGeometry>
</mxCell>
<!-- 2. 垂直向上 -->
<mxCell style="endArrow=none;" edge="1">
  <mxGeometry><mxPoint x="380" y="345" as="sourcePoint"/><mxPoint x="380" y="222" as="targetPoint"/></mxGeometry>
</mxCell>
<!-- 3. 水平向左回到步骤 -->
<mxCell style="endArrow=block;endFill=1;" edge="1">
  <mxGeometry><mxPoint x="380" y="222" as="sourcePoint"/><mxPoint x="320" y="222" as="targetPoint"/></mxGeometry>
</mxCell>
```

#### 功能结构图（重要，必须匹配模版风格）

**必须用上下结构，不要横向树形！** 模版风格是：
- 上方：学生端功能（一排竖排文字窄矩形）
- 中间：学生 → 系统名称 → 管理员（三个圆角矩形垂直排列）
- 下方：管理员端功能（一排竖排文字窄矩形）

关键参数：
- 画布总宽 600px，所有功能框均匀分布撑满
- 每个功能框：宽 42px，高 120px，统一大小
- 竖排文字用 `&lt;br&gt;` 逐字换行（不是 `horizontal=0` 旋转）
- 间距自动计算：`gap = (600 - n * 42) / (n - 1)`
- Word 中设为 **13.5cm 宽**撑满页面，高度按原图比例

```python
# 竖排文字写法（XML 转义）
def vertical(text):
    return '&amp;lt;br&amp;gt;'.join(text)
# "用户管理" → "用&lt;br&gt;户&lt;br&gt;管&lt;br&gt;理"
```

```xml
<!-- 竖排文字功能框 -->
<mxCell value="用&lt;br&gt;户&lt;br&gt;管&lt;br&gt;理" 
  style="whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#000000;fontSize=12;fontFamily=SimSun;verticalAlign=middle;spacing=0;" 
  vertex="1" parent="1">
  <mxGeometry x="0" y="425" width="42" height="120" as="geometry"/>
</mxCell>
```

#### ER 实体属性图
- 中心实体用矩形 `fillColor=#FFFFFF`
- 属性用椭圆均匀分布在周围（6个属性用六边形布局，8个用八方位布局）
- 主键 ID 用 HTML 下划线：`value="&lt;u&gt;主键ID&lt;/u&gt;"`
- 连线无箭头：`endArrow=none;`

#### 系统总体 ER 图
- 实体用小矩形（70x30），关系用小菱形（45x25）
- 中心实体（如"用户"）放中间，其余实体辐射状分布
- 关系菱形放在两个实体之间的连线上
- 文字标签用 `text` 元素标注关系名（如"发布"、"提问"）

### MCP 工具注意事项

1. **MCP 工具只在会话启动时加载**，断开需新开对话恢复
2. **XML 内容不要太大**，否则 URL 截断导致 "Unexpected end of JSON input"
3. **MCP 不能导出文件**，只能在浏览器中预览，导出必须用 CLI
4. 可以先用 `mcp__drawio__open_drawio_xml` 预览效果，确认无误后再用 CLI 导出

### 模版图片格式

模版（韩梦婷论文）中的图片存储为 **EMF（Enhanced Metafile）** 矢量格式。提取后需通过 LibreOffice 或 ImageMagick 转换为 PNG：
```bash
/Applications/LibreOffice.app/Contents/MacOS/soffice --headless --convert-to png xxx.emf --outdir ./
```

## 样式速查表

| 样式名 | 用途 | 字体 | 字号 | 行距 |
|--------|------|------|------|------|
| 论文正文 | 正文段落 | 宋体 | 12pt(小四) | 1.25倍 |
| Heading 1 | 章标题 | 黑体 | 15pt(小三) | 1.25倍 |
| Heading 2 | 节标题 | 黑体 | 14pt(四号) | 1.25倍 |
| Heading 3 | 小节标题 | 黑体 | 12pt(小四) | 1.25倍 |
| Caption | 图表标题 | 黑体 | 10pt(五号) | 居中 |
| 参考文献正文 | 参考文献条目 | 宋体 | 继承 | 1.25倍 |
| 结束语正文 | 结束语内容 | 宋体 | 12pt | 1.25倍 |
| 致谢正文 | 致谢内容 | 宋体 | 12pt | 1.25倍 |
| 内封2基本信息 | 封面字段 | - | 12pt | 1.5倍 |
| 内封统计 | 封面统计 | - | - | 固定 |
| 外封基本信息 | 外封面字段 | 黑体 | 14pt | 1.5倍 |
