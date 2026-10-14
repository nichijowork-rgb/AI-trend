# AI-trend：AIレポート ダッシュボード

別の収集botが毎日集めている X の **AI 関連の投稿**（新モデル・ツール・研究・ビジネス・**医療・介護×AI**）を、カテゴリ別に見やすく閲覧する静的 Web ページです。GitHub Pages で公開します（**https://nichijowork-rgb.github.io/AI-trend/**）。

[X-trend](https://github.com/nichijowork-rgb/X-trend)（夜職版）と同じ仕組み・同じデータ形式で、カテゴリと見た目だけを AI 向けにしています。

- サーバー・API・DB・ビルドは無し（HTML / CSS / 素の JavaScript）
- `data/*.json` は収集botだけが書く。画面は読むだけ
- `data/` が届くまでは `sample/` のサンプルデータを表示（画面上部に黄色い帯）
- ネイビー×シアン、ダーク/ライトは OS 設定に自動追従。PC・スマホ対応

## 使い方（開発）

```bash
npm start      # http://localhost:8000 （file:// では動きません）
npm test       # 単体テスト（Node 22 以上）
npm run sample # サンプルデータを作り直す（sample/ に出力）
```

## ドキュメント

- [docs/HANDOFF.md](docs/HANDOFF.md)：引き継ぎ書（**作業前に必読**）
- [docs/DATA_BOT_REQUEST.md](docs/DATA_BOT_REQUEST.md)：収集botの担当者に渡す依頼書
- [docs/DASHBOARD.md](docs/DASHBOARD.md)：画面仕様・データ仕様
- [docs/DASHBOARD_REQUIREMENTS.md](docs/DASHBOARD_REQUIREMENTS.md)：元の要件（X-trend の要件を AI 版に読み替えたもの）
