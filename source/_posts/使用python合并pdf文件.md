---
title: 使用python合并pdf文件
date: 2016-09-05 14:48:14
tags: "学习笔记"
---
当需要合并pdf文件的时候才发现很多pdf软件好像并没有提供这个功能，虽然Acrobat是有的…
干脆搜索了下，果然python有支持pdf编辑的库PyPDF2
<!-- more -->
安装PyPDF2:
```bash
pip install PyPDF2
```
简单的示例：
```python
import os
from PyPDF2 import PdfFileMerger

def merger_pdf():
    merger = PdfFileMerger()
    pdf_dir = "D:\\pdf\\"
    file_list = os.listdir(pdf_dir)
    for file_name in file_list:
        print pdf_dir + file_name
        input_pdf = open(pdf_dir+file_name, "rb")
        merger.append(input_pdf)
    output = open(pdf_dir+"output.pdf", "wb")
    merger.write(output)

if __name__ == '__main__':
    merger_pdf()
```
使用中遇到一个error：PyPDF2.utils.PdfReadError: Unexpected destination ‘/\_\_WKANCHOR\_2’
谷歌了下这个问题还是挺常见的，由于我输入的pdf是由wkhtmltopdf生成的…奇葩的Error…
解决办法：
https://github.com/mstamy2/PyPDF2/issues/193
最后使用注释的办法绕过了该问题，虽然不太友好╮(╯▽╰)╭…
