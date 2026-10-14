# AI-trend 引き継ぎ書（別のAI・開発者向け）

このリポジトリで作っている「AI・医療レポート」ダッシュボードの作り方・決まりごと・経緯をまとめた文書です。**作業を始める前に最後まで読んでください。** 詳細な画面仕様とデータ仕様は [DASHBOARD.md](DASHBOARD.md)、収集botへの依頼内容は [DATA_BOT_REQUEST.md](DATA_BOT_REQUEST.md) にあります。

---

## 1. 何を作っているか

- **目的**: 別の収集botが定期収集する X の AI 関連・医療／介護関連の投稿を、カテゴリ別に見やすく閲覧する Web ページ。AI の動向把握と投稿ネタ探しに使う。あわせて **医療・介護全般**の投稿も集め、右上の［AI｜医療］で切り替えて見る（オーナーは訪問看護まわりの仕事をしている）。
- **元になったもの**: [X-trend](https://github.com/nichijowork-rgb/X-trend)（夜職版の同じダッシュボード）を 2026-10-14 に複製し、カテゴリ・配色・文言を AI 向けに変えた。**データ形式・画面の動きは X-trend と同じ。**
- **公開URL**: https://nichijowork-rgb.github.io/AI-trend/ （GitHub Pages。独自ドメインは未設定）
- **リポジトリ**: `nichijowork-rgb/AI-trend`（公開リポジトリ）。ブランチは `main`（Pages の配信元）
- **利用者**: 非エンジニアのオーナー1名。説明は日本語・専門用語少なめで。

## 2. 全体構成（役割分担）

```
[収集bot（別担当・X-trend とは別のbot）]
   1日1回の集計 → data/*.json を push
        │
        ▼
[このリポジトリ]  ← GitHub Pages が push のたびに自動配信
   index.html + assets/  … ダッシュボード（読むだけ）
   data/                 … botだけが書く（フロントは絶対に触らない・コミットしない）
        │
        ▼
[閲覧者のブラウザ]  JSON を1回だけ取得して、絞り込み・並べ替えはすべて画面側で処理
```

- **サーバー・API・DB・ビルドは無し**。HTML / CSS / ES modules の素の JavaScript だけ。外部ライブラリも無し（グラフは自前 SVG）。
- 収集botの担当者とはオーナー経由でやり取りする。botへの依頼は「依頼文」を書いてオーナーに渡す（初回の依頼は [DATA_BOT_REQUEST.md](DATA_BOT_REQUEST.md)）。

## 3. ファイル構成

| パス | 役割 |
|---|---|
| `index.html` | ページ本体。CSP（通信先の制限）と `noindex` を設定 |
| `assets/dashboard.js` | エントリ。読み込み・状態管理・描画・イベント |
| `assets/js/core.js` | **DOM 非依存の純粋関数**（書式、フィルタ、並べ替え、集計、URL 状態、伸びペース、reports 正規化、カテゴリ色の割り当てなど）。単体テストの対象 |
| `assets/js/config.js` | **運用で変わる値と文言の置き場所**（更新スケジュール、レポート文言、カテゴリのグループ定義、表示名） |
| `assets/js/charts.js` | グラフ（横棒・日別積み上げ）。自前 SVG |
| `assets/js/embed.js` | X公式埋め込み（widgets.js の遅延読み込み） |
| `assets/js/media.js` | 写真・動画・GIF とライトボックス |
| `assets/js/theme-init.js` | テーマ初期化（OS設定に自動追従のみ） |
| `assets/dashboard.css` | デザイン。色はすべて `:root` のトークン |
| `sample/*.json` | サンプルデータ（`data/` が無い・壊れているとき表示） |
| `tools/make-sample.mjs` | サンプルの決定的生成（`npm run sample`） |
| `test/dashboard.test.js` | 単体テスト（node:test） |
| `test/e2e/dashboard-e2e.mjs` | E2E（Playwright。リポジトリに依存は入れない） |
| `data/` | 収集botの管轄。読むだけ（2026-10-14 時点ではまだ無い） |

## 4. カテゴリ

`assets/js/config.js` の `AI_CATEGORIES`・`MEDICAL_CATEGORIES`・`CATEGORY_GROUPS` で定義。並び順がそのままチップの並びになる。右上の［AI｜医療］でモードを切り替え、チップはそのモードのカテゴリ（＋未知カテゴリ）だけを出す。

| モード | カテゴリ | 色 |
|---|---|---|
| AI | 新モデル・発表 | パープル |
| AI | ツール・活用 | イエロー |
| AI | 研究・論文 | アクア |
| AI | ビジネス・業界 | オレンジ |
| AI | AIその他 | ブルー |
| **両方** | 医療・介護×AI（重点） | ピンク |
| 医療 | 訪問看護 | イエロー |
| 医療 | 医療ニュース | ブルー |
| 医療 | 介護・福祉 | アクア |
| 医療 | 制度・報酬改定 | オレンジ |
| — | 監視アカウント | グリーン（話題ではなく「見張り対象の人」の印） |

- 色は `assets/js/core.js` の `KNOWN_CATEGORY_SLOTS` → CSS の `--cat-N`。AI と医療は同時にチップが並ばないので色を使い回している（同じモード内では重ならない）。未知カテゴリは `--cat-8` / `--cat-9`。
- URL は `group=ai` / `group=medical`（既定は ai で URL に書かない）。X-trend にあった「一般のバズ」（`その他`）は 2026-10-14 に廃止。
- カテゴリ名を変えるときは **bot と同時に**（文字列の完全一致で判定している）。

## 5. 画面仕様の要点

X-trend で決まったことを引き継いでいる。変更するときは崩さないこと。

- **見た目**: ネイビー×シアン（アクセント ダーク `#3ee6f2` / ライト `#0b7f8a`）。ダーク/ライトは**OS設定に自動追従**（切替ボタンは無し）。PC・スマホ両対応（360px で横スクロールなし、タップ領域 44px 以上）。
- **ヘッダー右上**: ［AI｜医療］切替。
- **初期状態**: **24時間・AIすべて・インプレッションが多い順**。
- **期間**: 24時間 / 3日 / 7日 / 30日。**基準時刻は `generated_at`**（現在時刻ではない）。
- **並び替え**: 新しい順 / インプレッション / いいね / ブックマーク / 伸びペース。
- **「追跡アカウントだけ」ボタン**: 追跡（監視）アカウントの投稿だけに絞る。並び替えはそのまま使える（URL `watched=1`）。画面上の呼び方は「追跡」、データ上は `is_watched`・`監視アカウント` のまま。
- **「伸びている」ボタン**: reports.json の `rising` と `rising_watch`。**AI・医療関連のみ**（話題カテゴリの無い投稿は除外）。AI／医療の切替とは連動しない。しきい値は画面に決め打ちせず `condition_label` を表示。
- **「詳細」ボタン**: 検索・アカウント・監視のみ、サマリー、答え合わせ・的中率・週間、グラフ、上位アカウント。
- **不明な数値**は「不明」。0 や推測で埋めない。
- **性能**: JSON は最初の1回だけ取得。投稿は20件ずつ表示。画像は小サイズ＋遅延読み込み。

## 6. 守るべきルール

1. **`data/` を変更・コミットしない**（`git add -A ':!data'` を使う）。botと競合する。
2. **安全**: 本文などは必ず `textContent` / DOM 生成で入れる（`innerHTML` 禁止）。リンク化は http(s) のみ、`rel="noopener noreferrer"`。メディアは `https://pbs.twimg.com` / `video.twimg.com` のみ。
3. **決め打ち禁止**: botが変える可能性がある値（しきい値、条件名、カテゴリ一覧）は画面に固定しない。運用値・文言は `config.js` に集約。
4. **URL 状態**: フィルタはハッシュに保存（既定値と違うものだけ）。localStorage は try/catch。
5. **テスト**: 変更したら `npm test`（単体）と E2E を全件通す。見た目の変更は 1280px と 375px、ダーク/ライトで目視。
6. **コミット**: `main` に直接で可。bot が push を始めたら、push 前に `git pull --no-rebase`。
7. **ドキュメント**: 仕様を変えたら [DASHBOARD.md](DASHBOARD.md) も更新。

## 7. 開発・確認の手順

```bash
npm start      # = python3 -m http.server 8000 → http://localhost:8000
npm test       # 単体テスト（node 22+）
npm run sample # サンプル再生成（sample/ に出力。data/ には出さない）

# E2E（Playwright は別ディレクトリに入れる）
mkdir -p /tmp/qa && cd /tmp/qa && npm init -y && npm i playwright && npx playwright install chromium
PW_DIR=/tmp/qa/node_modules node test/e2e/dashboard-e2e.mjs
```

## 8. 経緯

1. 2026-10-14: X-trend をもとに新規リポジトリとして作成。カテゴリを AI 6 種（医療・介護×AI を重点）に、配色をネイビー×シアンに変更。サンプルデータを AI の話題で作り直し。独自ドメイン（CNAME）は持ち込んでいない。
2. 2026-10-14: 右上の切替を［AI｜一般のバズ］から［AI｜医療］に変更。医療側に 訪問看護／医療ニュース／介護・福祉／制度・報酬改定 を追加（医療・介護×AI は両方）。一般のバズは廃止。タイトルを「AI・医療レポート」に。
3. 2026-10-14: ツールバーに「追跡アカウントだけ」ボタンを追加（画面上の「監視」を「追跡」に言い換え）。

## 9. 未解決・今後の候補

- **収集botの用意**（担当者に [DATA_BOT_REQUEST.md](DATA_BOT_REQUEST.md) を渡す → push 権限の付与 → 初回データの確認）。
- 更新時刻が X-trend（毎日 23:59・週間は土曜 12:00）と違う場合は `config.js` の `UPDATE_SCHEDULE` を直す。
- 答え合わせの目標ライン（X-trend は 50万）を bot が変えたら、`config.js` の `REPORT_TEXT.crossedColumn` などの文言を直す。
- E2E の「レポート（サンプル）」の 1 件（週の切替 `#rweek` の待ちでタイムアウト）は X-trend 側でも失敗している既存の問題。画面上は週の切替は動いている。
