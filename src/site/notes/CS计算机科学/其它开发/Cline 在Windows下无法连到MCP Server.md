---
{"dg-publish":true,"permalink":"/CS计算机科学/其它开发/Cline 在Windows下无法连到MCP Server/","noteIcon":"","created":"2025-05-26T18:04:03.189+08:00","updated":"2025-05-26T18:19:10.948+08:00"}
---


作者：游鱼思

---

症状：

npm error code ENOENT npm error syscall lstat npm error path C:\Users\yuis\AppData\Local\Programs\Microsoft VS Code\\${APPDATA} npm error errno -4058 npm error enoent ENOENT: no such file or directory, lstat 'C:\Users\yuis\AppData\Local\Programs\Microsoft VS Code\\${APPDATA}' npm error enoent This is related to npm not being able to find a file.

Trae、Cursor 在Windows下也可能出现通用问题。

原因：

VS Code 启动 MCP 服务器时所在的子进程环境的 PATH 环境变量可能不完整，导致操作系统无法找到 `npx` 可执行文件，从而引发底层 `spawn npx ENOENT` 错误，最终表现为“未连接”错误。

解决办法：

1.  [Node.js](https://nodejs.org/zh-cn)使用官方 .msi 安装包，升级到最新版。
2. 安装 node.js 时，注意选择 add to path，添加环境变量路径。
3. VS Code 默认终端，选用 PowerShell 或 CMD
4. MCP Server 配置时，不能直接 `npx ...` ，应该改为 `cmd /c npx ...` 。只有先启动cmd，才能获取到环境变量。

Cline MCP Server 配置示例：

```json
{

  "mcpServers": {

    "filesystem": {

      "command": "cmd",

      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-filesystem", "C:/Users/username/Desktop"],

      "disabled": false,

      "autoApprove": [],

      "transportType": "stdio"

    },

    "github.com/GLips/Figma-Context-MCP": {

      "command": "cmd",

      "args": ["/c", "npx", "-y", "figma-developer-mcp", "--figma-api-key=your_key", "--stdio"],

      "disabled": false,

      "autoApprove": []

    }

  }

}
```

参考：

[Connect MCP Servers error"spawn npx enoent" · Issue #1948 · cline/cline](https://github.com/cline/cline/issues/1948)