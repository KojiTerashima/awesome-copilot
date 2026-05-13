# Marked

## クイック変換方法

`SKILL.md` の `### Quick Conversion Methods` の内容を展開。

### 方法1: CLI（単一ファイルに推奨）

```bash
# ファイルをHTMLに変換
marked -i input.md -o output.html

# 文字列を直接変換
marked -s "# Hello World"

# 出力: <h1>Hello World</h1>
```

### 方法2: Node.jsスクリプト

```javascript
import { marked } from 'marked';
import { readFileSync, writeFileSync } from 'fs';

const markdown = readFileSync('input.md', 'utf-8');
const html = marked.parse(markdown);
writeFileSync('output.html', html);
```

### 方法3: ブラウザでの使用

```html
<script src="https://cdn.jsdelivr.net/npm/marked/lib/marked.umd.js"></script>
<script>
  const html = marked.parse('# Markdown Content');
  document.getElementById('output').innerHTML = html;
</script>
```

---

## ステップバイステップのワークフロー

`SKILL.md` の `### Step-by-Step Workflows` の内容を展開。

### ワークフロー1: 単一ファイル変換

1. markedがインストールされていることを確認：`npm install -g marked`
2. 変換を実行：`marked -i README.md -o README.html`
3. 出力ファイルが作成されたことを確認

### ワークフロー2: バッチ変換（複数ファイル）

`convert-all.js` スクリプトを作成：

```javascript
import { marked } from 'marked';
import { readFileSync, writeFileSync, readdirSync } from 'fs';
import { join, basename } from 'path';

const inputDir = './docs';
const outputDir = './html';

readdirSync(inputDir)
  .filter(file => file.endsWith('.md'))
  .forEach(file => {
    const markdown = readFileSync(join(inputDir, file), 'utf-8');
    const html = marked.parse(markdown);
    const outputFile = basename(file, '.md') + '.html';
    writeFileSync(join(outputDir, outputFile), html);
    console.log(`変換完了: ${file} → ${outputFile}`);
  });
```

実行コマンド：`node convert-all.js`

### ワークフロー3: カスタムオプション付き変換

```javascript
import { marked } from 'marked';

// オプション設定
marked.setOptions({
  gfm: true,           // GitHub Flavored Markdownを有効化
  breaks: true,        // \nを<br>に変換
  pedantic: false,     // markdown.plの厳密な準拠を無効化
});

const html = marked.parse(markdownContent);
```

### ワークフロー4: 完全なHTMLドキュメント

変換した内容をフルHTMLテンプレートでラップ：

```javascript
import { marked } from 'marked';
import { readFileSync, writeFileSync } from 'fs';

const markdown = readFileSync('input.md', 'utf-8');
const content = marked.parse(markdown);

const html = `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; max-width: 800px; margin: 0 auto; padding: 2rem; }
    pre { background: #f4f4f4; padding: 1rem; overflow-x: auto; }
    code { background: #f4f4f4; padding: 0.2rem 0.4rem; border-radius: 3px; }
  </style>
</head>
<body>
${content}
</body>
</html>`;

writeFileSync('output.html', html);
```
