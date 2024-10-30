+++
title = 'MSI TRANSFORMS技术分析'

description = "虽然不是很深刻但至少开始写了"

tags = [ "技术" ]

date = 2024-10-30T18:31:14+08:00

+++

来源于真实环境中遇到的某组织

该技术比较冷门，Google只有Microsoft官方对其的两个介绍页面。它可以修改正常安装包的组件，实现木马的落地。

官方定义：

TRANSFORMS 属性是安装程序在安装包时应用的转换的列表。 安装程序按转换在属性中列出的顺序应用转换。 转换可以按文件名或按完整路径指定。 若要指定多个转换，请使用分号 (;) 分隔每个文件名或完整路径。 例如，若要对包应用三个转换，请将 TRANSFORMS 设置为文件名列表或完整路径列表。

```
TRANSFORMS=transform1.mst;transform2.mst;transform3.mst
TRANSFORMS=\\server\share\path\transform1.mst;\\server2\share2\path2\transform2.mst;\\server3\share3\path3\transform3.mst
```

可以通过在文件名前面加上冒号 (:) 来指示转换文件嵌入 .msi 文件的存储中，而不是一个独立文件。 例如，以下示例指示 transform1.mst 和 transform2.mst 嵌入在 .msi 文件中，而 transform3.mst 是独立文件。

```
TRANSFORMS=:transform1.mst;:transform2.mst;transform3.mst
```

安装程序在包的每次安装、播发、按需安装或维护安装时都需要 TRANSFORMS 中列出的转换。 TransformsSecure策略、TRANSFORMS 属性和 TRANSFORMS 字符串的第一个字符告知安装程序如何处理独立转换文件的源复原能力。 Windows Installer 将 TransformsAtSource 策略 或 TRANSFORMSATSOURCE设置得与 TransformsSecure 策略和 TRANSFORMSSECURE 相同。 请注意，嵌入 .msi 文件的转换不会缓存，并且始终从包中获取。



*看不懂，问了问GPT-4o*



`TRANSFORMS` 是 Microsoft Installer (MSI) 安装包中的一个参数，用于指定一个或多个变换文件（Transform files），这些变换文件可以在安装过程中修改 MSI 安装包的默认设置和行为。

#### MSI `TRANSFORMS` 参数

- **定义**:
  - `TRANSFORMS` 是一个参数，指定一个或多个变换文件（通常是 `.mst` 文件）。
  - 这些变换文件定义了对原始 MSI 安装包进行的修改或定制。
- **用途**:
  - 可以用来配置软件的默认设置，添加或移除功能模块，修改安装路径等。
  - 适用于企业部署场景，帮助管理员根据公司需要定制软件安装。

#### 示例

假设你有一个名为 `example.msi` 的安装包，以及一个变换文件 `custom.mst`，你可以在命令行中使用 `TRANSFORMS` 参数来应用变换文件：

```
msiexec /i example.msi TRANSFORMS=custom.mst
```

#### 多个变换文件

你也可以同时应用多个变换文件，用分号 `;` 分隔：

```
msiexec /i example.msi TRANSFORMS=custom1.mst;custom2.mst
```

#### 实际案例

例如，一个软件包在不同的部门需要不同的配置设置，使用变换文件可以创建多个配置方案而无需修改原始的 MSI 文件。

- **原始安装包**： `software.msi`
- **财务部门定制**： `finance.mst`
- **IT 部门定制**： `it.mst`

#### 安装财务部门定制版本：

```
msiexec /i software.msi TRANSFORMS=finance.mst
```

#### 安装 IT 部门定制版本：

```
msiexec /i software.msi TRANSFORMS=it.mst
```

#### 如何创建变换文件

变换文件通常是通过专门的 MSI 编辑工具创建的，例如：

- **Orca**：Microsoft 提供的一个免费的 MSI 数据库编辑器，可以用来创建和编辑变换文件。
- **InstEd**：第三方工具，功能强大且易于使用。

#### 总结

`TRANSFORMS` 参数在 MSI 安装过程中起到了关键作用，通过指定一个或多个变换文件，管理员可以灵活地定制软件的安装设置和行为。这对于大规模部署和不同部门的个性化需求特别有用，既能减少工作量，又能确保安装的一致性和准确性。



我的理解是攻击者在msi安装文件中嵌入了自己的恶意代码
