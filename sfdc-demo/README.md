# TSUBASA-Legacy 統合基幹業務システム(Devinデモ用アプリ)

お客様にDevinの機能一式(計画 → 実装 → PR作成 → CI → コードレビュー対応 → UIテスト)を
デモするための、**意図的にレガシーな見た目・作りにした**業務システム風Webアプリです。

## テーマ

共通テーマは「レガシーモダナイゼーション」。1タブ=1業種で、お客様の業種に合わせて
どのタブを使ってデモするか選べます。

| タブ | 架空企業 | 業務 |
|------|----------|------|
| 航空 | つばさ航空(JAL/ANA風) | 便運航状況の照会・予約(PNR)管理 |
| 金融 | みらい信用銀行 | 口座元帳照会・振込処理 |
| 小売 | さくらマート | 在庫マスタ照会・売上登録(POS連携) |

UIは1990年代の汎用機端末・Windows 95風(灰色パネル、青いタイトルバー、密なテーブル)に
しており、Devinがモダナイズすると**変化が一目でわかる**ようになっています。

## 起動方法

```bash
cd sfdc-demo
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

ブラウザで http://localhost:8000 を開く。

## お客様に公開する(cloudflared トンネル)

デモ当日だけ一時的に公開URLを発行する方法。無料でアカウント不要、コードを書き換えても
再デプロイ不要で即反映されるため、モダナイゼーションのデモに向いています。

```bash
# 1. cloudflared を取得(初回のみ)
curl -sL -o cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
chmod +x cloudflared
# macOS の場合: brew install cloudflared

# 2. アプリを起動(別ターミナル)
cd sfdc-demo && uvicorn main:app --port 8000

# 3. トンネルを開く
./cloudflared tunnel --url http://localhost:8000
```

出力に表示される `https://<ランダム名>.trycloudflare.com` をお客様に共有します。
URLはトンネルを停止するまで有効(停止すると失効し、次回は別のURLになります)。

## 構成

- `main.py` — FastAPIバックエンド。インメモリのシードデータとJSON API
  (`/api/flights`, `/api/reservations`, `/api/accounts`, `/api/transfer`,
  `/api/inventory`, `/api/sales` など)。
- `static/` — フレームワーク不使用の素朴なHTML/CSS/JSフロントエンド。

## デモシナリオ例(Devinへの依頼文サンプル)

UIの変化がわかりやすい順に並べています。各依頼でDevinが計画 → 実装 → PR作成 →
CI確認 → ブラウザでのUIテスト(録画付き)まで行う流れを見せられます。

1. **見た目のモダナイズ(効果最大)**
   「sfdc-demoのUIをモダンにしてください。ダークモード対応のカードベースの
   レイアウト、モダンなフォント、レスポンシブ対応にしてください。」
2. **可視化の追加**
   「便ごとの搭乗率を棒グラフで表示するダッシュボードを航空タブに追加してください。」
3. **機能追加**
   「予約のキャンセル機能を追加してください(一覧に取消ボタン、APIも追加)。」
4. **バリデーション/UX改善**
   「振込フォームに確認ダイアログと入力チェック(全角数字の自動変換)を追加してください。」
5. **バックエンド刷新**
   「インメモリデータをSQLiteに永続化してください。」
6. **国際化**
   「UIを日英切替できるようにしてください。」

各シナリオは独立しているので、デモのたびにこのブランチ状態へ戻せば繰り返し使えます。
