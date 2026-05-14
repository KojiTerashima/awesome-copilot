# エバリュエーターの検証 (Python)

人間がラベルを付けた例に対して LLM 評価者を検証します。 TPR/TNR/精度 >80% を目標にします。

## メトリクスを計算する「」パイソン
sklearn.metrics からのインポート、classification_report、confusion_matrix

print(classification_report(human_labels, evaluator_predictions))

cm = 混乱行列(人間ラベル、評価者予測)
tn、fp、fn、tp = cm.ravel()
tpr = tp / (tp + fn)
tnr = tn / (tn + fp)
print(f"TPR: {tpr:.2f}, TNR: {tnr:.2f}")
「」## 正しい生産見積り「」パイソン
デフォルトの正しい推定値(観測値、tpr、tnr):
    """既知の TPR/TNR を使用して、観察された合格率を調整します。"""
    return (観測値 - (1 - tnr)) / (tpr - (1 - tnr))
「」## 誤って分類されたものを検索「」パイソン
# 誤検知: 評価者は合格、人間は不合格
fp_mask = (evaluator_predictions == 1) & (human_labels == 0)
false_positives = データセット[fp_mask]

# False Negative: 評価者は失敗、人間は合格
fn_mask = (evaluator_predictions == 0) & (human_labels == 1)
false_negatives = データセット[fn_mask]
「」## 赤旗

- TPR または TNR < 70%
- TPRとTNRの間に大きなギャップがある
- カッパ < 0.6