# 文件MD5代码校验
CertUtil -hashfile C:\Users\ymam0\Desktop\quant_goods_126.sql MD5
MD5 的 C:\Users\ymam0\Desktop\quant_goods_126.sql 哈希:
bc015ef23696c5435b232734a9da54cc
CertUtil: -hashfile 命令成功完成。

BC015EF23696C5435B232734A9DA54CC

CertUtil -hashfile C:\Users\ymam0\Desktop\quant_goods_127.sql MD5
MD5 的 C:\Users\ymam0\Desktop\quant_goods_127.sql 哈希:
74c60d4784703c7f378f0e30e350665b
CertUtil: -hashfile 命令成功完成。

在 Windows 11 上生成文件的 MD5 校验值，可以使用 PowerShell 或第三方工具。以下是两种常见的方法：

## 使用 PowerShell

1. **打开 PowerShell**：右键点击开始菜单，选择“Windows PowerShell（管理员）”或“Windows Terminal（管理员）”。

2. **使用 `Get-FileHash` 命令**：输入以下命令来计算文件的 MD5 值：
   ```powershell
   Get-FileHash -Path "文件路径" -Algorithm MD5
   ```
   将 `"文件路径"` 替换为你要计算 MD5 值的文件的实际路径。例如：
   ```powershell
   Get-FileHash -Path "C:\Users\YourUsername\Documents\example.txt" -Algorithm MD5
   ```

## 使用第三方工具

如果你更喜欢图形用户界面工具，可以使用以下常见的工具：

1. **WinMD5Free**：这是一个简单的工具，可以快速计算和验证文件的 MD5 值。
   - 下载并安装 [WinMD5Free](https://www.winmd5.com/)。
   - 打开工具，点击“浏览”选择你的文件，然后点击“计算”获取 MD5 值。

2. **HashCalc**：另一个流行的工具，可以计算 MD5、SHA-1、SHA-256 等多种哈希值。
   - 下载并安装 [HashCalc](http://www.slavasoft.com/hashcalc/)。
   - 打开工具，选择你的文件，选择 MD5，然后点击“计算”。

选择你喜欢的方法，按照步骤操作即可计算文件的 MD5 值。
