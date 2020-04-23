---
title: Excel 合并多个文件
date: 2020-02-24 21:00:00
tags: 'Excel'
categories:
  - ['使用说明', 'Excel']
permalink: excel-merge-file
---

> 通过使用 VB 快速的合并多个 Excel 工作簿成为一个工作簿

- 新建一个工作薄, 将其命名为你合并后的名字
- 打开此工作薄
- 在其下任一个工作表标签上点击右键, 选择"查看代码"
- 在打开的VBA编辑窗口中粘贴以下代码
- 将需要合并的文件放置到同一目录下
- 工具 - 宏 - 宏, 选 "合并文件", 然后 "执行"

注意: **脚本会自动去掉同一标题**

```vb
Sub 合并文件()
Dim MyPath$, MyName$, sh As Worksheet, sht As Worksheet, m&
Set sh = ActiveSheet
MyPath = ThisWorkbook.Path & "\"
MyName = Dir(MyPath & "*.xls")
Application.ScreenUpdating = False
Cells.ClearContents
Do While MyName <> ""
If MyName <> ThisWorkbook.Name Then
With GetObject(MyPath & MyName)
For Each sht In .Sheets
If IsSheetEmpty = IsEmpty(sht.UsedRange) Then
m = m + 1
If m = 1 Then
sht.[a1].CurrentRegion.Copy sh.[a1]
Else
sht.[a1].CurrentRegion.Offset(1).Copy sh.[a65536].End(xlUp).Offset(1)
End If
End If
Next
.Close False
End With
End If
MyName = Dir
Loop
Application.ScreenUpdating = True
End Sub
```
