# 数学式をHTMLに書く

## インライン式の書き方

### Markdown

```markdown
この文はインラインで数式を表示するために `$` 区切りを使っています: $\sqrt{3x-1}+(1+x)^2$
```

### 解析されたHTML

```html
<p>この文はインラインで数式を表示するために <code>$</code> 区切りを使っています:
 <math-renderer><math xmlns="http://www.w3.org/1998/Math/MathML">
  <msqrt>
   <mn>3</mn>
   <mi>x</mi>
   <mo>−</mo>
   <mn>1</mn>
  </msqrt>
  <mo>+</mo>
  <mo>(</mo>
  <mn>1</mn>
  <mo>+</mo>
  <mi>x</mi>
  <msup>
   <mo>)</mo>
   <mn>2</mn>
  </msup>
</math>
</math-renderer>
</p>
```

### Markdown

```markdown
この文はインラインで数式を表示するために $\` と \`$ 区切りを使っています: $`\sqrt{3x-1}+(1+x)^2`$
```

### 解析されたHTML

```html
<p>この文はインラインで数式を表示するために
 <math-renderer>
  <math xmlns="http://www.w3.org/1998/Math/MathML">
   <mo>‘</mo>
   <mi>a</mi>
   <mi>n</mi>
   <mi>d</mi>
   <mo>‘</mo>
  </math>
 </math-renderer> 区切りを使っています:
 <math-renderer>
  <math xmlns="http://www.w3.org/1998/Math/MathML">
   <msqrt>
    <mn>3</mn>
    <mi>x</mi>
    <mo>−</mo>
    <mn>1</mn>
   </msqrt>
   <mo>+</mo>
   <mo stretchy="false">(</mo>
   <mn>1</mn>
   <mo>+</mo>
   <mi>x</mi>
   <msup>
    <mo stretchy="false">)</mo>
    <mn>2</mn>
   </msup>
  </math>
 </math-renderer>
</p>
```

---

## ブロック式の書き方

### Markdown

```markdown
**コーシー・シュワルツの不等式**\
$$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$
```

### 解析されたHTML

```html
<p>
  <strong>コーシー・シュワルツの不等式</strong><br>
  <math-renderer>
    <math xmlns="http://www.w3.org/1998/Math/MathML">
      <msup>
        <mrow>
          <mo>(</mo>
          <munderover>
            <mo>∑</mo>
            <mrow>
              <mi>k</mi>
              <mo>=</mo>
              <mn>1</mn>
            </mrow>
            <mi>n</mi>
          </munderover>
          <msub>
            <mi>a</mi>
            <mi>k</mi>
          </msub>
          <msub>
            <mi>b</mi>
            <mi>k</mi>
          </msub>
          <mo>)</mo>
        </mrow>
        <mn>2</mn>
      </msup>
      <mo>≤</mo>
      <mrow>
        <mo>(</mo>
        <munderover>
          <mo>∑</mo>
          <mrow>
            <mi>k</mi>
            <mo>=</mo>
            <mn>1</mn>
          </mrow>
          <mi>n</mi>
        </munderover>
        <msubsup>
          <mi>a</mi>
          <mi>k</mi>
          <mn>2</mn>
        </msubsup>
        <mo>)</mo>
      </mrow>
      <mrow>
        <mo>(</mo>
        <munderover>
          <mo>∑</mo>
          <mrow>
            <mi>k</mi>
            <mo>=</mo>
            <mn>1</mn>
          </mrow>
          <mi>n</mi>
        </munderover>
        <msubsup>
          <mi>b</mi>
          <mi>k</mi>
          <mn>2</mn>
        </msubsup>
        <mo>)</mo>
      </mrow>
    </math>
  </math-renderer>
</p>
```

### Markdown

```markdown
**コーシー・シュワルツの不等式**

    ```math
    \left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)
    ```
```

### 解析されたHTML

```html
<p><strong>コーシー・シュワルツの不等式</strong></p>

<math-renderer>
  <math xmlns="http://www.w3.org/1998/Math/MathML">
    <msup>
      <mrow>
        <mo>(</mo>
        <munderover>
          <mo>∑</mo>
          <mrow>
            <mi>k</mi>
            <mo>=</mo>
            <mn>1</mn>
          </mrow>
          <mi>n</mi>
        </munderover>
        <msub>
          <mi>a</mi>
          <mi>k</mi>
        </msub>
        <msub>
          <mi>b</mi>
          <mi>k</mi>
        </msub>
        <mo>)</mo>
      </mrow>
      <mn>2</mn>
    </msup>
    <mo>≤</mo>
    <mrow>
      <mo>(</mo>
      <munderover>
        <mo>∑</mo>
        <mrow>
          <mi>k</mi>
          <mo>=</mo>
          <mn>1</mn>
        </mrow>
        <mi>n</mi>
      </munderover>
      <msubsup>
        <mi>a</mi>
        <mi>k</mi>
        <mn>2</mn>
      </msubsup>
      <mo>)</mo>
    </mrow>
    <mrow>
      <mo>(</mo>
      <munderover>
        <mo>∑</mo>
        <mrow>
          <mi>k</mi>
          <mo>=</mo>
          <mn>1</mn>
        </mrow>
        <mi>n</mi>
      </munderover>
      <msubsup>
        <mi>b</mi>
        <mi>k</mi>
        <mn>2</mn>
      </msubsup>
      <mo>)</mo>
    </mrow>
  </math>
</math-renderer>
```

### Markdown

```markdown
方程式 $a^2 + b^2 = c^2$ はピタゴラスの定理です。
```

### 解析されたHTML

```html
<p>方程式
 <math-renderer><math xmlns="http://www.w3.org/1998/Math/MathML">
  <msup>
    <mi>a</mi>
    <mn>2</mn>
  </msup>
  <mo>+</mo>
  <msup>
    <mi>b</mi>
    <mn>2</mn>
  </msup>
  <mo>=</mo>
  <msup>
    <mi>c</mi>
    <mn>2</mn>
  </msup>
 </math></math-renderer> はピタゴラスの定理です。
</p>
```

### Markdown

```
$$
\int_0^\infty e^{-x} dx = 1
$$
```

### 解析されたHTML

```html
<p><math-renderer><math xmlns="http://www.w3.org/1998/Math/MathML">
  <msubsup>
    <mo>∫</mo>
    <mn>0</mn>
    <mi>∞</mi>
  </msubsup>
  <msup>
    <mi>e</mi>
    <mrow>
      <mo>−</mo>
      <mi>x</mi>
    </mrow>
  </msup>
  <mi>d</mi>
  <mi>x</mi>
  <mo>=</mo>
  <mn>1</mn>
</math></math-renderer></p>
```

---

## 数学式内のドル記号のインライン表示

### Markdown

```markdown
この式は `\$` を使ってドル記号を表示しています: $`\sqrt{\$4}`$
```

### 解析されたHTML

```html
<p>この式は
 <code>\$</code> を使ってドル記号を表示しています:
 <math-renderer>
  <math xmlns="http://www.w3.org/1998/Math/MathML">
   <msqrt>
    <mi>$</mi>
    <mn>4</mn>
   </msqrt>
  </math>
 </math-renderer>
</p>
```

### Markdown

```markdown
100ドルを半分に分けるには、$100/2$ を計算します
```

### 解析されたHTML

```html
<p>100ドルを半分に分けるには、
 <span>$</span>100 を計算します
 <math-renderer>
  <math xmlns="http://www.w3.org/1998/Math/MathML">
   <mn>100</mn>
   <mrow data-mjx-texclass="ORD">
    <mo>/</mo>
   </mrow>
   <mn>2</mn>
  </math>
 </math-renderer>
</p>
```
