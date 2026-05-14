---
name: csharp-mcp-server-generator
description: 'tools、prompts、適切な設定を備えた完全な C# MCP server project を生成します'
---

# Generate C# MCP Server

次の仕様で、C# の完全な Model Context Protocol (MCP) server を作成してください。

## Requirements

1. **Project Structure**: 適切な directory structure を持つ新しい C# console application を作成する
2. **NuGet Packages**: ModelContextProtocol (prerelease) と Microsoft.Extensions.Hosting を含める
3. **Logging Configuration**: stdio transport の妨げにならないよう、すべての log を stderr に出す設定にする
4. **Server Setup**: 適切な DI 設定を伴う Host builder pattern を使う
5. **Tools**: 適切な attribute と description を持つ有用な tool を少なくとも 1 つ作る
6. **Error Handling**: 適切な error handling と validation を含める

## Implementation Details

### Basic Project Setup
- .NET 8.0 以降を使う
- console application を作成する
- 必要な NuGet package を `--prerelease` flag 付きで追加する
- logging を stderr に設定する

### Server Configuration
- DI と lifecycle management には `Host.CreateApplicationBuilder` を使う
- `AddMcpServer()` を stdio transport 付きで設定する
- tool 自動検出には `WithToolsFromAssembly()` を使う
- server は `RunAsync()` で実行されるようにする

### Tool Implementation
- tool class に `[McpServerToolType]` attribute を使う
- tool method に `[McpServerTool]` attribute を使う
- tool と parameter に `[Description]` attribute を付ける
- 適切な箇所では async operation をサポートする
- 適切な parameter validation を含める

### Code Quality
- C# の naming convention に従う
- XML documentation comment を含める
- nullable reference type を使う
- McpProtocolException による適切な error handling を実装する
- debugging 用に structured logging を使う

## Example Tool Types to Consider
- file operation（read、write、search）
- data processing（transform、validate、analyze）
- external API integration（HTTP request）
- system operation（command 実行、status 確認）
- database operation（query、update）

## Testing Guidance
- server の実行方法を説明する
- MCP client で試すための command 例を示す
- troubleshooting tip を含める

包括的な documentation と error handling を備えた、production-ready な MCP server を生成してください。
