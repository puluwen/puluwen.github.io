---
layout: post
title: "python读取Excel常用包，可插入删除一行一列"
date: 2019-05-06
description: "介绍python读取Excel常用的包，包含插入如何一行一列"
tag: 工具

---

# 总览 
- `xlrd`：用于读Excel文件
- `xlwt`：写Excel包，文件后缀名为.xls，最多只能存`65536`行数据（熟悉的朋友，可能会知道这个数字是2的16次方）
- `xlsxwriter`：也是写Excel包，文件后缀名.xlsx，最大支持1048576（2的20次方）行数据，16384（2的14次方）列数据
- `openpyxl`：既能读也能写，非常厉害，还能插入删除一行一列，后缀名是.xlsx

# xlrd使用

```python
# -*- coding: utf-8 -*-
import xlrd
xlsfile = r"C:\Users\Administrator\Desktop\test\Account.xls"# 打开指定路径中的xls文件
book = xlrd.open_workbook(xlsfile)#得到Excel文件的book对象，实例化对象
sheet0 = book.sheet_by_index(0) # 通过sheet索引获得sheet对象
print("1、",sheet0)
sheet_name = book.sheet_names()[0]# 获得指定索引的sheet表名字
print("2、",sheet_name)
sheet1 = book.sheet_by_name(sheet_name)# 通过sheet名字来获取，当然如果知道sheet名字就可以直接指定
nrows = sheet0.nrows    # 获取行总数
print("3、",nrows)
#循环打印每一行的内容
for i in range(nrows):
    print(sheet1.row_values(i))
ncols = sheet0.ncols    #获取列总数
print("4、",ncols)
row_data = sheet0.row_values(0)     # 获得第1行的数据列表
print(row_data)
col_data = sheet0.col_values(0)     # 获得第1列的数据列表
print("5、",col_data)
# 通过坐标读取表格中的数据
cell_value1 = sheet0.cell_value(0, 0)
print("6、",cell_value1)
cell_value2 = sheet0.cell_value(0, 1)
print("7、",cell_value2)
```

# xlwt使用
```python
# -*- coding: utf-8 -*-
#导入xlwt模块
import xlwt
# 创建一个Workbook对象，这就相当于创建了一个Excel文件
book = xlwt.Workbook(encoding='utf-8', style_compression=0)
'''
Workbook类初始化时有encoding和style_compression参数
encoding:设置字符编码，一般要这样设置：w = Workbook(encoding='utf-8')，就可以在excel中输出中文了。
默认是ascii。当然要记得在文件头部添加：
#!/usr/bin/env python
# -*- coding: utf-8 -*-
style_compression:表示是否压缩，不常用。
'''
#创建一个sheet对象，一个sheet对象对应Excel文件中的一张表格。
# 在电脑桌面右键新建一个Excel文件，其中就包含sheet1，sheet2，sheet3三张表
sheet = book.add_sheet('test', cell_overwrite_ok=True)
# 其中的test是这张表的名字,cell_overwrite_ok，表示是否可以覆盖单元格，其实是Worksheet实例化的一个参数，默认值是False
# 向表test中添加数据
sheet.write(0, 0, 'EnglishName')  # 其中的'0-行, 0-列'指定表中的单元，'EnglishName'是向该单元写入的内容
sheet.write(1, 0, 'Marcovaldo')
txt1 = '中文名字'
sheet.write(0, 1, txt1.decode('utf-8'))  # 此处需要将中文字符串解码成unicode码，否则会报错
txt2 = '马可瓦多'
sheet.write(1, 1, txt2.decode('utf-8'))
 
# 最后，将以上操作保存到指定的Excel文件中
book.save(r'e:\test1.xls')  # 在字符串前加r，声明为raw字符串，这样就不会处理其中的转义了。否则，可能会报错
``` 

# xlsxwriter使用
```python
#coding:utf-8
import xlsxwriter
 
workbook=xlsxwriter.Workbook('demo1.xlsx')#创建一个excel文件
worksheet=workbook.add_worksheet(u'sheet1')#在文件中创建一个名为TEST的sheet,不加名字默认为sheet1
 
worksheet.set_column('A:A',20)#设置第一列宽度为20像素
bold=workbook.add_format({'bold':True})#设置一个加粗的格式对象
 
worksheet.write('A1','HELLO')#在A1单元格写上HELLO
worksheet.write('A2','WORLD',bold)#在A2上写上WORLD,并且设置为加粗
worksheet.write('B2',U'中文测试',bold)#在B2上写上中文加粗
 
worksheet.write(2,0,32)#使用行列的方式写上数字32,35,5
worksheet.write(3,0,35.5)#使用行列的时候第一行起始为0,所以2,0代表着第三行的第一列,等价于A4
worksheet.write(4,0,'=SUM(A3:A4)')#写上excel公式

workbook.close()
```

# 如何向Excel插入一行或一列
功能非常强大，[文档主页点这里](https://openpyxl.readthedocs.io/en/stable/)

- 插入列用`insert_cols`
- 插入行用`insert_rows'

读并加一列示例：
```python
import openpyxl

wb = openpyxl.load_workbook('0.xlsx')
ws = wb.worksheets[0]
# 在第3列之前插入数据，这里序号是从1开始的
ws.insert_cols(3)
# 插入数据
for index, row in enumerate(ws.rows):#按行读取
    if index == 0:
        row[2].value = '新字段'
    else:
        row[2].value = index
wb.save('0_new.xlsx')
```

写示例：
```python
import openpyxl
wb = openpyxl.Workbook()#创建一个表
sheet = wb.active#找到活动sheet页，
sheet.title = 'New Sheet'
sheet['C3'] = 'hello world'#这里读取是一样的，按cell读
for i in range(10):
    sheet["A%d" % (i+1)].value = i + 1

sheet["E1"].value = "=SUM(A:A)"#还可以写公式
wb.save('新的excel.xlsx')

```