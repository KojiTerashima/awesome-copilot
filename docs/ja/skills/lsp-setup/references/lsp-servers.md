# Copilot CLI用の既知のLSPサーバー

`lsp-setup`スキルの参照データです。各セクションにはOSごとのインストールコマンドとすぐに使える設定スニペットが含まれています。

> **設定スニペットの形式**：以下の各スニペットは、トップレベルの`lspServers`キーの下に値として挿入するオブジェクトを示しています。完全な設定ファイルは `{ "lspServers": { <ここにスニペット> } }` のようになります。複数の言語を追加する場合は、それぞれのスニペットを`lspServers`の兄弟キーとしてマージしてください。

---

## TypeScript / JavaScript

**サーバー**: [typescript-language-server](https://github.com/typescript-language-server/typescript-language-server)

### インストール

| OS      | コマンド                                               |
|---------|-------------------------------------------------------|
| 任意     | `npm install -g typescript typescript-language-server` |

### 設定スニペット

```json
{
  "typescript": {
    "command": "typescript-language-server",
    "args": ["--stdio"],
    "fileExtensions": {
      ".ts": "typescript",
      ".tsx": "typescriptreact",
      ".js": "javascript",
      ".jsx": "javascriptreact"
    }
  }
}
```

---

## Java

**サーバー**: [Eclipse JDT Language Server (jdtls)](https://github.com/eclipse-jdtls/eclipse.jdt.ls)

`JAVA_HOME`または`$PATH`に**Java 21以上**が必要です。

### インストール

| OS      | コマンド                           |
|---------|-----------------------------------|
| macOS   | `brew install jdtls`              |
| Linux   | ディストリビューションのリポジトリで`jdtls`または`eclipse.jdt.ls`を確認、または https://download.eclipse.org/jdtls/milestones/ からダウンロード |
| Windows | https://download.eclipse.org/jdtls/milestones/ からダウンロードし、`bin/`を`PATH`に追加 |

macOSのHomebrewでは、バイナリは`jdtls`として`$PATH`にインストールされます。

### 設定スニペット

```json
{
  "java": {
    "command": "jdtls",
    "args": [],
    "fileExtensions": {
      ".java": "java"
    }
  }
}
```

> **注意**：`jdtls`のラッパースクリプトは`--stdio`モードを内部で処理します。手動インストールの場合はランチャージャーjarを直接起動する必要があるかもしれません。詳細は[jdtls README](https://github.com/eclipse-jdtls/eclipse.jdt.ls#running-from-command-line-with-wrapper-script)を参照してください。

---

## Python

**サーバー**: [pyright](https://github.com/microsoft/pyright)

### インストール

| OS      | コマンド                    |
|---------|----------------------------|
| 任意     | `npm install -g pyright`   |
| 任意     | `pip install pyright`      |

### 設定スニペット

```json
{
  "python": {
    "command": "pyright-langserver",
    "args": ["--stdio"],
    "fileExtensions": {
      ".py": "python"
    }
  }
}
```

---

## Go

**サーバー**: [gopls](https://github.com/golang/tools/tree/master/gopls)

### インストール

| OS      | コマンド                                    |
|---------|--------------------------------------------|
| 任意     | `go install golang.org/x/tools/gopls@latest` |
| macOS   | `brew install gopls`                       |

### 設定スニペット

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "fileExtensions": {
      ".go": "go"
    }
  }
}
```

---

## Rust

**サーバー**: [rust-analyzer](https://github.com/rust-lang/rust-analyzer)

### インストール

| OS      | コマンド                        |
|---------|--------------------------------|
| 任意     | `rustup component add rust-analyzer` |
| macOS   | `brew install rust-analyzer`   |
| Linux   | ディストリビューションのパッケージまたは`rustup` |
| Windows | `rustup component add rust-analyzer` またはGitHubリリースからダウンロード |

### 設定スニペット

```json
{
  "rust": {
    "command": "rust-analyzer",
    "args": [],
    "fileExtensions": {
      ".rs": "rust"
    }
  }
}
```

---

## C / C++

**サーバー**: [clangd](https://clangd.llvm.org/)

### インストール

| OS      | コマンド                                |
|---------|----------------------------------------|
| macOS   | `brew install llvm`（clangd含む）またはXcodeコマンドラインツール |
| Linux   | `apt install clangd` / `dnf install clang-tools-extra` |
| Windows | https://releases.llvm.org/ からLLVMをダウンロード |

### 設定スニペット

```json
{
  "cpp": {
    "command": "clangd",
    "args": ["--background-index"],
    "fileExtensions": {
      ".c": "c",
      ".h": "c",
      ".cpp": "cpp",
      ".cxx": "cpp",
      ".cc": "cpp",
      ".hpp": "cpp",
      ".hxx": "cpp"
    }
  }
}
```

---

## C# (.NET)

**サーバー**: [Roslyn Language Server](https://github.com/dotnet/roslyn) (`dotnet dnx`経由)

### インストール

| OS      | コマンド                                                        |
|---------|----------------------------------------------------------------|
| 任意     | [.NET SDK](https://dot.net/download)のインストールが必要        |

### 設定スニペット

```json
{
  "csharp": {
    "command": "dotnet",
    "args": ["dnx", "roslyn-language-server", "--yes", "--prerelease", "--", "--stdio", "--autoLoadProjects"],
    "fileExtensions": {
      ".cs": "csharp"
    }
  }
}
```

---

## Ruby

**サーバー**: [solargraph](https://github.com/castwide/solargraph)

### インストール

| OS      | コマンド                   |
|---------|---------------------------|
| 任意     | `gem install solargraph`  |

### 設定スニペット

```json
{
  "ruby": {
    "command": "solargraph",
    "args": ["stdio"],
    "fileExtensions": {
      ".rb": "ruby",
      ".rake": "ruby",
      ".gemspec": "ruby"
    }
  }
}
```

---

## PHP

**サーバー**: [intelephense](https://github.com/bmewburn/vscode-intelephense)

### インストール

| OS      | コマンド                                    |
|---------|--------------------------------------------|
| 任意     | `npm install -g intelephense`              |

### 設定スニペット

```json
{
  "php": {
    "command": "intelephense",
    "args": ["--stdio"],
    "fileExtensions": {
      ".php": "php"
    }
  }
}
```

---

## Kotlin

**サーバー**: [kotlin-language-server](https://github.com/fwcd/kotlin-language-server)

### インストール

| OS      | コマンド                                           |
|---------|---------------------------------------------------|
| macOS   | `brew install kotlin-language-server`             |
| 任意     | GitHubリリースからダウンロードし`PATH`に追加       |

### 設定スニペット

```json
{
  "kotlin": {
    "command": "kotlin-language-server",
    "args": [],
    "fileExtensions": {
      ".kt": "kotlin",
      ".kts": "kotlin"
    }
  }
}
```

---

## Swift

**サーバー**: [sourcekit-lsp](https://github.com/swiftlang/sourcekit-lsp)（Swiftツールチェーンに同梱）

### インストール

| OS      | コマンド                                                        |
|---------|----------------------------------------------------------------|
| macOS   | Xcodeに含まれ、バイナリは`xcrun sourcekit-lsp`で利用可能      |
| Linux   | Swiftツールチェーンに含まれ、https://swift.org からインストール可能 |

### 設定スニペット

```json
{
  "swift": {
    "command": "sourcekit-lsp",
    "args": [],
    "fileExtensions": {
      ".swift": "swift"
    }
  }
}
```

> macOSではフルパス `/usr/bin/sourcekit-lsp` を使うか、`command`を`xcrun`にして`args: ["sourcekit-lsp"]`と設定する必要がある場合があります。

---

## Lua

**サーバー**: [lua-language-server](https://github.com/LuaLS/lua-language-server)

### インストール

| OS      | コマンド                              |
|---------|--------------------------------------|
| macOS   | `brew install lua-language-server`   |
| Linux   | GitHubリリースからダウンロード        |
| Windows | GitHubリリースからダウンロード        |

### 設定スニペット

```json
{
  "lua": {
    "command": "lua-language-server",
    "args": [],
    "fileExtensions": {
      ".lua": "lua"
    }
  }
}
```

---

## YAML

**サーバー**: [yaml-language-server](https://github.com/redhat-developer/yaml-language-server)

### インストール

| OS      | コマンド                                      |
|---------|----------------------------------------------|
| 任意     | `npm install -g yaml-language-server`        |

### 設定スニペット

```json
{
  "yaml": {
    "command": "yaml-language-server",
    "args": ["--stdio"],
    "fileExtensions": {
      ".yaml": "yaml",
      ".yml": "yaml"
    }
  }
}
```

---

## Bash / Shell

**サーバー**: [bash-language-server](https://github.com/bash-lsp/bash-language-server)

### インストール

| OS      | コマンド                                       |
|---------|-----------------------------------------------|
| 任意     | `npm install -g bash-language-server`         |

### 設定スニペット

```json
{
  "bash": {
    "command": "bash-language-server",
    "args": ["start"],
    "fileExtensions": {
      ".sh": "shellscript",
      ".bash": "shellscript",
      ".zsh": "shellscript"
    }
  }
}
```
