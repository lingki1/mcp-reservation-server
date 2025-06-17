# Claude Desktop 配置修复指南

## 🚨 问题症状
```
错误: Error: Operation not permitted. The 'get_available_meets' operation on server 'reservation' requires appropriate permissions.
```

## 🔍 问题诊断

### 1. 检查Claude Desktop配置文件位置

**Windows 系统配置文件位置**：
```
%APPDATA%\Claude\claude_desktop_config.json
```

**完整路径通常是**：
```
C:\Users\[用户名]\AppData\Roaming\Claude\claude_desktop_config.json
```

### 2. 验证MCP服务器状态

我们的测试显示MCP服务器本身工作正常：
- ✅ 服务器启动成功
- ✅ 返回13个工具
- ✅ 环境变量配置正确

## 🛠️ 修复步骤

### 步骤1: 找到正确的配置文件

在PowerShell中执行：
```powershell
# 检查配置文件是否存在
$configPath = "$env:APPDATA\Claude\claude_desktop_config.json"
echo "配置文件路径: $configPath"
Test-Path $configPath
```

### 步骤2: 备份现有配置（如果存在）

```powershell
# 备份现有配置
$configPath = "$env:APPDATA\Claude\claude_desktop_config.json"
if (Test-Path $configPath) {
    Copy-Item $configPath "$configPath.backup.$(Get-Date -Format 'yyyyMMdd-HHmmss')"
    echo "已备份现有配置"
} else {
    echo "配置文件不存在，将创建新文件"
}
```

### 步骤3: 创建或更新配置文件

```powershell
# 确保目录存在
$configDir = "$env:APPDATA\Claude"
if (!(Test-Path $configDir)) {
    New-Item -ItemType Directory -Path $configDir -Force
}

# 创建配置文件
$config = @{
    mcpServers = @{
        reservation = @{
            command = "node"
            args = @("C:\Projects\restweapp\mcp-reservation-server\dist\index.js")
            env = @{
                WECHAT_APP_ID = ""
                WECHAT_APP_SECRET = ""
                WECHAT_ENV_ID = "-"
            }
        }
    }
}

$configJson = $config | ConvertTo-Json -Depth 10
$configPath = "$env:APPDATA\Claude\claude_desktop_config.json"
$configJson | Out-File -FilePath $configPath -Encoding UTF8
echo "配置文件已更新: $configPath"
```

### 步骤4: 验证配置文件

```powershell
# 显示配置文件内容
$configPath = "$env:APPDATA\Claude\claude_desktop_config.json"
Get-Content $configPath | ConvertFrom-Json | ConvertTo-Json -Depth 10
```

### 步骤5: 重启Claude Desktop

**重要**: 修改配置后必须：
1. 完全关闭Claude Desktop应用
2. 等待5-10秒
3. 重新启动Claude Desktop

### 步骤6: 验证连接

重启后，在Claude中测试：
```
请查询所有可用的预约窗口
```

## 🔧 高级故障排除

### 检查文件权限

```powershell
# 检查配置文件权限
$configPath = "$env:APPDATA\Claude\claude_desktop_config.json"
Get-Acl $configPath | Format-List
```

### 检查Node.js路径

```powershell
# 验证Node.js是否在PATH中
node --version
Get-Command node
```

### 检查MCP服务器文件

```powershell
# 验证MCP服务器文件存在
$serverPath = "C:\Projects\restweapp\mcp-reservation-server\dist\index.js"
Test-Path $serverPath
```

### 手动测试MCP服务器

```powershell
# 手动启动服务器测试
cd C:\Projects\restweapp\mcp-reservation-server
node dist/index.js
```

## 🎯 常见问题解决

### 问题1: 配置文件路径错误
**症状**: Claude找不到MCP服务器
**解决**: 使用绝对路径，确保反斜杠正确转义

### 问题2: 环境变量未设置
**症状**: 服务器启动但API调用失败
**解决**: 在配置文件的env字段中明确设置所有环境变量

### 问题3: Node.js版本问题
**症状**: 服务器无法启动
**解决**: 确保Node.js版本 >= 18

### 问题4: 缓存问题
**症状**: 修改配置后仍然出错
**解决**: 完全卸载并重新安装Claude Desktop

## 📋 完整配置模板

```json
{
  "mcpServers": {
    "reservation": {
      "command": "node",
      "args": ["C:\\Projects\\restweapp\\mcp-reservation-server\\dist\\index.js"],
      "env": {
        "WECHAT_APP_ID": "",
        "WECHAT_APP_SECRET": "",
        "WECHAT_ENV_ID": ""
      }
    }
  }
}
```

## ✅ 验证成功的标志

当配置正确时，您应该能够：
1. 查询预约窗口（不再出现权限错误）
2. 查询预约记录
3. 修改预约状态
4. 删除预约记录

所有工具都应该能够正常工作，不再出现"Operation not permitted"错误。 
