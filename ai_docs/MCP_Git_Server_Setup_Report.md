# Git MCP 伺服器設置技術報告

## 專案資訊
- **日期**: 2025年11月16日
- **倉庫**: gitgraph_test (https://github.com/lee359/gitgraph_test)
- **目標**: 為 GitHub 倉庫配置 Model Context Protocol (MCP) Git 伺服器

## 問題摘要

在為本地 Git 倉庫配置 MCP 伺服器時遇到了兩個主要問題：
1. 初始配置使用 `uvx` 命令導致 "Cannot read properties of undefined (reading 'cmd')" 錯誤
2. 修改為 Python 命令後出現 "Server disconnected" 錯誤

## 解決方案

### 問題 1: uvx 命令錯誤

**錯誤訊息**:
```
Cannot read properties of undefined (reading 'cmd')
```

**原因分析**:
- Claude Desktop 配置中使用了 `uvx` 命令
- 系統中未安裝 `uv` 或 `uvx` 工具
- Claude Desktop 無法識別該命令

**初始配置** (錯誤):
```json
{
  "gitgraph-test": {
    "command": "uvx",
    "args": [
      "mcp-server-git",
      "--repository",
      "c:\\Users\\user\\gitgraph_test"
    ]
  }
}
```

**解決方法**:
改用系統已安裝的 Python 直接執行模組：

```json
{
  "gitgraph-test": {
    "command": "python",
    "args": [
      "-m",
      "mcp_server_git",
      "--repository",
      "c:\\Users\\user\\gitgraph_test"
    ]
  }
}
```

### 問題 2: Server Disconnected

**錯誤訊息**:
```
Server disconnected
```

**原因分析**:
- `mcp-server-git` Python 套件未安裝
- 執行 `python -m mcp_server_git` 時找不到模組

**診斷步驟**:
```powershell
# 測試命令是否可執行
python -m mcp_server_git --repository c:\Users\user\gitgraph_test

# 輸出錯誤
C:\Users\user\AppData\Local\Programs\Python\Python313\python.exe: No module named mcp_server_git
```

**解決方法**:
安裝 `mcp-server-git` 套件及其依賴：

```powershell
pip install mcp-server-git
```

**安裝的套件** (主要依賴):
- mcp-server-git (2025.9.25)
- mcp (1.21.1)
- gitpython (3.1.45)
- pydantic (2.12.4)
- click (8.3.1)
- 其他依賴套件 (共 39 個)

## 最終配置

### 完整的 Claude Desktop 配置文件

**路徑**: `C:\Users\user\AppData\Roaming\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "node",
      "args": [
        "C:\\Program Files\\nodejs\\node_modules\\@modelcontextprotocol\\server-filesystem\\dist\\index.js",
        "C:\\Users\\user\\runs\\detect"
      ]
    },
    "YOLOv8 Detection Server": {
      "command": "C:\\Users\\user\\MCPproject-YOLOv8\\venv\\Scripts\\python.exe",
      "args": [
        "C:\\Users\\user\\MCPproject-YOLOv8\\mcpclient.py"
      ],
      "env": {
        "PYTHONPATH": "C:\\Users\\user\\MCPproject-YOLOv8"
      }
    },
    "gitgraph-test": {
      "command": "python",
      "args": [
        "-m",
        "mcp_server_git",
        "--repository",
        "c:\\Users\\user\\gitgraph_test"
      ]
    }
  }
}
```

## 驗證步驟

### 1. 驗證套件安裝
```powershell
pip list | Select-String "mcp-server-git"
# 應顯示: mcp-server-git
```

### 2. 驗證命令可執行
```powershell
python -m mcp_server_git --help
# 應顯示幫助訊息
```

### 3. 驗證 Git 倉庫路徑
```powershell
Test-Path "c:\Users\user\gitgraph_test\.git"
# 應返回: True
```

### 4. 測試 MCP 伺服器
```powershell
python -m mcp_server_git --repository c:\Users\user\gitgraph_test
# 伺服器應正常啟動
```

## Git MCP 伺服器功能

安裝成功後，Git MCP 伺服器提供以下功能：

1. **讀取和搜尋 Git 倉庫**
2. **查看提交歷史**
3. **檢視文件內容**
4. **查看分支和標籤**
5. **查看差異和狀態**

## 技術細節

### MCP 伺服器類型
- **類型**: 本地伺服器
- **費用**: 完全免費（無需 API 金鑰或雲端服務）
- **運行方式**: 直接在本地電腦上運行，讀取本地 Git 倉庫

### 系統需求
- Python 3.7+
- Git 已安裝並配置
- 有效的本地 Git 倉庫

### 套件依賴關係
```
mcp-server-git
├── mcp (>=1.0.0)
├── gitpython (>=3.1.43)
├── pydantic (>=2.0.0)
├── click (>=8.1.7)
└── [其他依賴套件...]
```

## 故障排除建議

### 如果再次遇到問題

1. **檢查日誌**
   - 在 Claude Desktop 錯誤對話框中點擊 "Open Logs Folder"
   - 查看最新的日誌文件

2. **驗證安裝**
   ```powershell
   pip show mcp-server-git
   ```

3. **重新安裝套件**
   ```powershell
   pip uninstall mcp-server-git
   pip install mcp-server-git
   ```

4. **檢查 Python 路徑**
   ```powershell
   where python
   ```

5. **測試直接執行**
   ```powershell
   python -m mcp_server_git --repository <倉庫路徑>
   ```

## 重啟步驟

配置修改後必須重啟 Claude Desktop：

1. 完全關閉 Claude Desktop（包括系統托盤）
2. 重新啟動應用程式
3. 等待 MCP 伺服器連接

## 結論

通過將配置從 `uvx` 改為 `python -m` 執行方式，並安裝必要的 Python 套件，成功解決了 Git MCP 伺服器的連接問題。此配置為完全本地化、免費的解決方案，無需任何外部 API 或付費服務。

## 參考資源

- [Model Context Protocol 官方文檔](https://modelcontextprotocol.io/)
- [MCP Git Server GitHub](https://github.com/modelcontextprotocol/servers)
- [mcp-server-git PyPI](https://pypi.org/project/mcp-server-git/)

---

**報告生成時間**: 2025-11-16  
**技術支援**: Claude AI Assistant
