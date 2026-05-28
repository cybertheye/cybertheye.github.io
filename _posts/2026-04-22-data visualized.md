---
title: 🌳 数据可视化
layout: post
author: cyven
tags: python pandas plot
categories: CS CS::Tech
---


## 代码逻辑梳理

### 整体架构
脚本 `generate_price_charts_all_points.py` 是一个完整的Excel数据可视化流水线，主要流程如下：
```
读取Excel → 解析数据结构 → 处理中文字体 → 生成折线图 → 创建HTML报告
```

### 主要函数详解

#### 1. `setup_chinese_font()`
**功能**：解决matplotlib中文显示问题
**技术点**：
- 检测系统可用中文字体（macOS常见字体列表）
- 通过`plt.rcParams['font.sans-serif']`设置字体族
- 回退机制：尝试多个字体直到成功
- 设置`axes.unicode_minus=False`解决负号显示

#### 2. `parse_sheet_data(df_raw)`
**功能**：解析Excel工作表，提取结构化数据
**核心逻辑**：
```python
# 关键步骤
1. 识别供应商列（非"Unnamed"列）
2. 配对供应商价格列（优惠前 + 优惠后）
3. 推断缺失的机号（从上个非NaN值继承）
4. 格式化产品ID为"机号 功率"
5. 构建价格字典 {supplier: [prices]}
```

**技术难点处理**：
- **机号推断**：当机号为NaN时，使用上一行的机号值
- **列配对**：识别"供应商名"列和对应的"Unnamed"折扣列
- **数据清洗**：处理NaN值，转换为float类型

#### 3. `create_line_chart(product_ids, prices_dict, suppliers, title, output_path)`
**功能**：生成折线图，显示所有数据点
**核心改进**：
```python
# 关键改进：显示所有价格点
if len(segment) >= 2:
    # 连续点：连线+标记
    plt.plot(..., linewidth=2, marker='o')
else:
    # 孤立点：仅标记（linewidth=0）
    plt.plot(..., linewidth=0, marker='o')
```

**X轴排序解决方案**：
```python
# 1. 创建位置映射
product_to_xpos = {pid: i for i, pid in enumerate(product_ids)}

# 2. 使用数值位置绘图
x_positions = [product_to_xpos[product_ids[j]] for j in segment]

# 3. 显式设置刻度标签
plt.xticks(range(len(product_ids)), product_ids, rotation=60, ha='right')
```

#### 4. `generate_html_report(sheet_names, output_dir)`
**功能**：生成包含所有图表的HTML汇总报告
**技术点**：
- 动态生成HTML内容
- 使用CSS进行响应式布局
- 自动包含时间戳

#### 5. `main()`
**功能**：主控流程
**执行顺序**：
1. 设置中文字体
2. 读取Excel文件
3. 遍历所有工作表
4. 为每个工作表生成两张图表（优惠前/后）
5. 生成HTML报告

### 主要技术点

#### 1. **pandas数据处理**
- `pd.read_excel()`: 读取Excel文件
- `df.columns`: 获取列名
- `df.iloc[]`: 按位置索引行数据
- `pd.isna()/pd.notna()`: 检测NaN值

#### 2. **matplotlib可视化**
- `plt.figure()`: 创建图表
- `plt.plot()`: 绘制折线（关键参数：`marker='o'`, `linewidth=2/0`）
- `plt.xticks()`: 控制X轴刻度
- `plt.xlim()`: 设置X轴范围
- `plt.legend()`: 添加图例
- `plt.savefig()`: 保存PNG

#### 3. **数据映射技术**
```python
# 解决分类数据排序问题
product_to_xpos = {pid: i for i, pid in enumerate(product_ids)}
```
将字符串映射为数值位置，保持原始顺序。

#### 4. **连续数据段检测**
```python
# 分组连续索引
segments = []
current_segment = []
for idx in valid_indices:
    if not current_segment:
        current_segment.append(idx)
    elif idx == current_segment[-1] + 1:
        current_segment.append(idx)
    else:
        segments.append(current_segment)
        current_segment = [idx]
```

### 技术难点与解决方案

#### 难点1：X轴分类数据排序混乱
**问题**：Matplotlib将字符串作为分类数据，默认按字母顺序或首次出现顺序排列
**解决方案**：
- 创建`{产品ID: 位置索引}`映射字典
- 使用数值位置绘图
- 显式设置刻度标签`plt.xticks(positions, labels)`

#### 难点2：孤立数据点显示
**问题**：原代码要求至少2个点才显示，导致孤立点被过滤
**解决方案**：
- 移除`min_points`限制
- 区分处理：2+点绘制连线，1点只绘制标记
- 设置`linewidth=0`使孤立点无连线

#### 难点3：中文字体兼容性
**问题**：不同系统字体不同，中文可能显示为方框
**解决方案**：
- 预定义常见中文字体列表
- 尝试多个字体直到成功
- 设置回退字体机制

#### 难点4：数据缺失推断
**问题**：Excel中机号只出现在分组第一行，后续行为空
**解决方案**：
```python
last_machine = None
for row in data:
    if pd.notna(machine):
        last_machine = machine
    elif last_machine is not None:
        machine = last_machine  # 继承上一行的机号
```

#### 难点5：供应商列识别
**问题**：Excel中供应商列与"Unnamed"折扣列交替出现
**解决方案**：
```python
while i < len(cols):
    if 'Unnamed' not in str(col):
        supplier = col
        if i+1 < len(cols) and 'Unnamed' in str(cols[i+1]):
            supplier_cols[supplier] = (i, i+1)  # 配对列
            i += 2
```

### 代码亮点

1. **健壮的错误处理**：字体检测、数据转换都有try-catch保护
2. **清晰的统计输出**：显示处理了多少供应商、线段、孤立点
3. **模块化设计**：每个函数职责单一，易于测试和维护
4. **用户友好**：生成HTML报告，可直接在浏览器中查看所有图表
5. **配置灵活**：图表尺寸、颜色方案、标签旋转等参数可调

### 可扩展性

如需进一步定制：
1. **颜色方案**：修改`plt.cm.tab20c`为其他色彩映射
2. **图表尺寸**：调整`figsize=(14, 8)`参数
3. **输出格式**：修改`plt.savefig(dpi=300)`的DPI值
4. **数据过滤**：在`parse_sheet_data()`中添加自定义过滤逻辑

这个脚本成功解决了复杂Excel数据可视化中的多个技术挑战，特别是分类数据排序、中文显示和孤立点处理等常见问题。
