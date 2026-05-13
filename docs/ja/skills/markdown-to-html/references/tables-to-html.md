# テーブルをHTMLに変換する

## テーブルの作成

### Markdown

```markdown

| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |
```

### 解析されたHTML

```html
<table>
 <thead>
  <tr>
   <th>First Header</th>
   <th>Second Header</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>Content Cell</td>
   <td>Content Cell</td>
  </tr>
  <tr>
   <td>Content Cell</td>
   <td>Content Cell</td>
  </tr>
 </tbody>
</table>
```

### Markdown

```markdown
| Command | Description |
| --- | --- |
| git status | 新規または変更されたファイルをすべてリストアップ |
| git diff | ステージされていないファイルの差分を表示 |
```

### 解析されたHTML

```html
<table>
 <thead>
  <tr>
   <th>Command</th>
   <th>Description</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>git status</td>
   <td>新規または変更されたファイルをすべてリストアップ</td>
  </tr>
  <tr>
   <td>git diff</td>
   <td>ステージされていないファイルの差分を表示</td>
  </tr>
 </tbody>
</table>
```

## テーブル内のコンテンツの書式設定

### Markdown

```markdown
| Command | Description |
| --- | --- |
| `git status` | 新しいまたは*変更された*ファイルをすべてリストアップ |
| `git diff` | ステージされて**いない**ファイルの差分を表示 |
```

### 解析されたHTML

```html
<table>
 <thead>
  <tr>
   <th>Command</th>
   <th>Description</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td><code>git status</code></td>
   <td>新しいまたは<em>変更された</em>ファイルをすべてリストアップ</td>
  </tr>
  <tr>
   <td><code>git diff</code></td>
   <td>ステージされて<strong>いない</strong>ファイルの差分を表示</td>
  </tr>
 </tbody>
</table>
```

### Markdown

```markdown
| 左寄せ       | 中央寄せ       | 右寄せ         |
| :---         |     :---:      |          ---:  |
| git status   | git status     | git status     |
| git diff     | git diff       | git diff       |
```

### 解析されたHTML

```html
<table>
  <thead>
   <tr>
    <th align="left">左寄せ</th>
    <th align="center">中央寄せ</th>
    <th align="right">右寄せ</th>
   </tr>
  </thead>
  <tbody>
   <tr>
    <td align="left">git status</td>
    <td align="center">git status</td>
    <td align="right">git status</td>
   </tr>
   <tr>
    <td align="left">git diff</td>
    <td align="center">git diff</td>
    <td align="right">git diff</td>
   </tr>
  </tbody>
</table>
```

### Markdown

```markdown
| 名前       | 文字       |
| ---        | ---        |
| Backtick   | `          |
| Pipe       | \|         |
```

### 解析されたHTML

```html
<table>
 <thead>
  <tr>
   <th>名前</th>
   <th>文字</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>Backtick</td>
   <td>`</td>
  </tr>
  <tr>
   <td>Pipe</td>
   <td>|</td>
  </tr>
 </tbody>
</table>
```
