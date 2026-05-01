# pharmacy-quote

病院薬局向け 価格見積もり管理システム

---

## 概要

採用追加品目について全卸へ一括で価格見積もりを依頼し、回答を比較・採用・CSV出力できるWebアプリ。

---

## 構成

| 項目 | 内容 |
|------|------|
| GitHub | `fukuchik05-maker/pharmacy-quote` |
| Vercel | `pharmacy-quote.vercel.app` |
| Supabase | 独立プロジェクト（pharmacy-inventoryとは分離） |

---

## ファイル構成

| ファイル | 用途 |
|----------|------|
| `quote_m.html` | 薬剤部管理画面 |
| `quote.html` | 卸向け入力フォーム |
| `vercel.json` | Vercelルーティング設定 |

---

## DBテーブル（Supabase）

| テーブル | 内容 |
|----------|------|
| `master_medicines` | orderepiマスター（YJコード・品名・薬価・GS1・JAN・包装数など） |
| `pq_sessions` | 見積もりセッション（タイトル・期限） |
| `pq_items` | セッションの対象品目（マスタのスナップショット） |
| `pq_quotes` | 卸の回答（価格・在庫区分・備考） |

---

## 運用フロー

```
① マスタ取り込み
   orderepi CSVをアップロード → master_medicinesにupsert

② セッション作成
   タイトル・期限を設定 → 品目を選択 → 卸5社のURLを発行・送付

③ 卸が回答（quote.html）
   品目ごとに価格・在庫区分・備考を入力 → 送信

④ 比較・採用（quote_m.html）
   品目×卸の価格マトリクスを確認 → 採用卸を選択 → CSV出力
```

---

## quote_m.html（薬剤部側）の機能

- orderepi CSVアップロード → `master_medicines` にupsert
- セッション作成（タイトル・期限・品目選択）
- 卸5社（アトル・スズケン・ダイコー・東邦・琉薬）のURL発行・コピー
- 回答比較画面（品目×卸の価格マトリクス・最安値ハイライト・薬価率表示）
- 採用卸の選択 → 採用CSV出力

---

## quote.html（卸側）の機能

- URLパラメータでセッション・卸名を受け取り
- 品目一覧表示（品名・規格・包装薬価・包装数・GS1コード）
- 価格入力 → 薬価率リアルタイム計算
- 在庫区分選択（常備・受発注・取扱なし）・備考入力
- 再送信可能（前回回答を自動読み込み・上書き）
- 期限切れ警告バナー表示
- 送信前CSV出力（確認用）
- 送信後CSV出力（控え）

---

## 薬価率計算式

```
薬価率 = 提示価格 ÷ 包装薬価 × 100（%）
```

| 薬価率 | 表示色 | 意味 |
|--------|--------|------|
| 90.9%以下 | 🟢 緑 | 税込み逆ザヤにならない |
| 90.9%超 | 🔴 赤 | 税込み逆ザヤになる |

> 消費税10%を考慮すると、薬価の90.9%（= 1 ÷ 1.1）を超える価格での仕入れは逆ザヤになる。

---

## 卸向けURLの形式

```
https://pharmacy-quote.vercel.app/quote.html?session={SESSION_ID}&wholesaler={卸名}
```

例：
```
https://pharmacy-quote.vercel.app/quote.html?session=xxxx-xxxx&wholesaler=アトル
```

---

## 将来対応（未実装）

- GitHub Actionsによる自動ping（Supabase停止防止）
- 採用価格のpharmacy-inventoryへのCSV取込連携
