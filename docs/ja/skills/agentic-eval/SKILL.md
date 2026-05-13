---
name: agentic-eval
description: |
  AIエージェントの出力を評価・改善するためのパターンと手法。このスキルは次のような場合に使用します:
  - 自己批評とリフレクションのループを実装する
  - 品質重視の生成のために evaluator-optimizer パイプラインを構築する
  - テスト駆動のコード改善ワークフローを作成する
  - ルーブリックベースまたは LLM-as-judge の評価システムを設計する
  - エージェント出力（コード、レポート、分析）に反復的な改善を追加する
  - エージェント応答の品質を測定・改善する
---

# Agentic Evaluation Patterns

反復的な評価と改良によって自己改善するためのパターン。

## Overview

評価パターンにより、エージェントは自身の出力を評価・改善でき、単発の生成を超えて反復的な改良ループへ進めます。

```
Generate → Evaluate → Critique → Refine → Output
    ↑                              │
    └──────────────────────────────┘
```

## When to Use

- **品質が重要な生成**: 高い正確性が求められるコード、レポート、分析
- **明確な評価基準があるタスク**: 成功指標が定義されている
- **特定の基準への準拠が必要なコンテンツ**: スタイルガイド、コンプライアンス、フォーマット

---

## Pattern 1: Basic Reflection

自己批評を通じて、エージェントが自身の出力を評価・改善します。

```python
def reflect_and_refine(task: str, criteria: list[str], max_iterations: int = 3) -> str:
    """Generate with reflection loop."""
    output = llm(f"Complete this task:\n{task}")
    
    for i in range(max_iterations):
        # Self-critique
        critique = llm(f"""
        Evaluate this output against criteria: {criteria}
        Output: {output}
        Rate each: PASS/FAIL with feedback as JSON.
        """)
        
        critique_data = json.loads(critique)
        all_pass = all(c["status"] == "PASS" for c in critique_data.values())
        if all_pass:
            return output
        
        # Refine based on critique
        failed = {k: v["feedback"] for k, v in critique_data.items() if v["status"] == "FAIL"}
        output = llm(f"Improve to address: {failed}\nOriginal: {output}")
    
    return output
```

**重要な洞察**: 批評結果を確実にパースするため、構造化された JSON 出力を使います。

---

## Pattern 2: Evaluator-Optimizer

責務を明確にするため、生成と評価を別々のコンポーネントに分離します。

```python
class EvaluatorOptimizer:
    def __init__(self, score_threshold: float = 0.8):
        self.score_threshold = score_threshold
    
    def generate(self, task: str) -> str:
        return llm(f"Complete: {task}")
    
    def evaluate(self, output: str, task: str) -> dict:
        return json.loads(llm(f"""
        Evaluate output for task: {task}
        Output: {output}
        Return JSON: {{"overall_score": 0-1, "dimensions": {{"accuracy": ..., "clarity": ...}}}}
        """))
    
    def optimize(self, output: str, feedback: dict) -> str:
        return llm(f"Improve based on feedback: {feedback}\nOutput: {output}")
    
    def run(self, task: str, max_iterations: int = 3) -> str:
        output = self.generate(task)
        for _ in range(max_iterations):
            evaluation = self.evaluate(output, task)
            if evaluation["overall_score"] >= self.score_threshold:
                break
            output = self.optimize(output, evaluation)
        return output
```

---

## Pattern 3: Code-Specific Reflection

コード生成向けのテスト駆動リファインメントループです。

```python
class CodeReflector:
    def reflect_and_fix(self, spec: str, max_iterations: int = 3) -> str:
        code = llm(f"Write Python code for: {spec}")
        tests = llm(f"Generate pytest tests for: {spec}\nCode: {code}")
        
        for _ in range(max_iterations):
            result = run_tests(code, tests)
            if result["success"]:
                return code
            code = llm(f"Fix error: {result['error']}\nCode: {code}")
        return code
```

---

## Evaluation Strategies

### Outcome-Based
出力が期待される結果を達成しているかを評価します。

```python
def evaluate_outcome(task: str, output: str, expected: str) -> str:
    return llm(f"Does output achieve expected outcome? Task: {task}, Expected: {expected}, Output: {output}")
```

### LLM-as-Judge
LLM を使って出力を比較し、順位付けします。

```python
def llm_judge(output_a: str, output_b: str, criteria: str) -> str:
    return llm(f"Compare outputs A and B for {criteria}. Which is better and why?")
```

### Rubric-Based
重み付きの評価軸に基づいて出力を採点します。

```python
RUBRIC = {
    "accuracy": {"weight": 0.4},
    "clarity": {"weight": 0.3},
    "completeness": {"weight": 0.3}
}

def evaluate_with_rubric(output: str, rubric: dict) -> float:
    scores = json.loads(llm(f"Rate 1-5 for each dimension: {list(rubric.keys())}\nOutput: {output}"))
    return sum(scores[d] * rubric[d]["weight"] for d in rubric) / 5
```

---

## Best Practices

| Practice | Rationale |
|----------|-----------|
| **明確な基準** | 具体的で測定可能な評価基準を最初に定義する |
| **反復回数の上限** | 無限ループ防止のため最大反復回数（3〜5）を設定する |
| **収束チェック** | 反復間で出力スコアが改善しない場合は停止する |
| **履歴の記録** | デバッグと分析のために全経路を保持する |
| **構造化出力** | 評価結果を確実にパースするため JSON を使う |

---

## Quick Start Checklist

```markdown
## Evaluation Implementation Checklist

### Setup
- [ ] 評価基準/ルーブリックを定義する
- [ ] 「十分に良い」とみなすスコア閾値を設定する
- [ ] 最大反復回数を設定する（デフォルト: 3）

### Implementation
- [ ] generate() 関数を実装する
- [ ] 構造化出力付きで evaluate() 関数を実装する
- [ ] optimize() 関数を実装する
- [ ] リファインメントループを接続する

### Safety
- [ ] 収束検知を追加する
- [ ] デバッグ用にすべての反復を記録する
- [ ] 評価結果のパース失敗を適切に処理する
```

