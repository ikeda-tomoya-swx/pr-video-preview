---
name: agent-browser
description: ブラウザ操作の自動化。Webページのテスト、フォーム入力、スクリーンショット取得、データ抽出、動画録画に使用する。ユーザーが「ブラウザで確認して」「Webページを開いて」「スクリーンショットを撮って」「フォームを入力して」「操作を録画して」などと依頼した場合に使用する。
allowed-tools: Bash(agent-browser:*)
---

# agent-browserによるブラウザ自動化

AIエージェント向けに最適化されたブラウザ自動化ツール。コンテキスト消費を抑えつつ効率的にブラウザ操作を実行できる。

## 前提条件

agent-browserがインストールされていること:

```bash
npm install -g agent-browser
agent-browser install  # Chromiumのダウンロード
```

## 基本ワークフロー

1. **ページを開く**: `agent-browser open <url>`
2. **要素を取得**: `agent-browser snapshot -i`（インタラクティブ要素のみ、ref付き）
3. **操作を実行**: snapshotで得たrefを使って操作（`@e1`, `@e2`など）
4. **必要に応じて再snapshot**: DOM変更後は再度snapshotを取得
5. **終了**: `agent-browser close`

## コマンドリファレンス

### ナビゲーション

```bash
agent-browser open <url>      # URLに移動
agent-browser back            # 戻る
agent-browser forward         # 進む
agent-browser reload          # リロード
agent-browser close           # ブラウザを閉じる
```

### スナップショット（ページ解析）

```bash
agent-browser snapshot        # 完全なアクセシビリティツリー
agent-browser snapshot -i     # インタラクティブ要素のみ（推奨）
agent-browser snapshot -c     # コンパクト出力
agent-browser snapshot -d 3   # 深さを3に制限
```

### インタラクション（snapshotの@refを使用）

```bash
agent-browser click @e1           # クリック
agent-browser dblclick @e1        # ダブルクリック
agent-browser fill @e2 "テキスト"  # クリアして入力
agent-browser type @e2 "テキスト"  # クリアせず入力
agent-browser press Enter         # キー押下
agent-browser press Control+a     # キーコンビネーション
agent-browser hover @e1           # ホバー
agent-browser check @e1           # チェックボックスをオン
agent-browser uncheck @e1         # チェックボックスをオフ
agent-browser select @e1 "値"     # ドロップダウン選択
agent-browser scroll down 500     # ページスクロール
agent-browser scrollintoview @e1  # 要素までスクロール
```

### 情報取得

```bash
agent-browser get text @e1        # 要素のテキスト取得
agent-browser get value @e1       # 入力値を取得
agent-browser get title           # ページタイトル取得
agent-browser get url             # 現在のURL取得
```

### スクリーンショット

```bash
agent-browser screenshot          # 標準出力に出力
agent-browser screenshot path.png # ファイルに保存
agent-browser screenshot --full   # フルページ
```

### 動画録画

Playwrightのネイティブ録画機能を使用してブラウザセッションをWebM形式で録画できる。

```bash
agent-browser record start ./demo.webm  # 録画開始（出力ファイルを指定）
agent-browser record stop               # 録画停止・保存
```

#### 録画の注意点

- 録画開始時に新しいコンテキストが作成されるが、cookies/storageは保持される
- URLを指定しない場合、自動的に現在のページに戻る
- スムーズなデモを作るには、先に操作を確認してから録画を開始するのがおすすめ

#### 録画の使用例

```bash
# ログインフローを録画
agent-browser open https://example.com/login
agent-browser snapshot -i
# 先に操作を確認...

# 準備ができたら録画開始
agent-browser record start ./login-demo.webm
agent-browser open https://example.com/login
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3
agent-browser wait --load networkidle
agent-browser record stop
```

### トレース記録

デバッグ用にトレースを記録できる。

```bash
agent-browser trace start [path]   # トレース記録開始
agent-browser trace stop [path]    # トレース停止・保存
```

### 待機

```bash
agent-browser wait @e1                     # 要素の出現を待機
agent-browser wait 2000                    # ミリ秒待機
agent-browser wait --text "成功"           # テキストの出現を待機
agent-browser wait --load networkidle      # ネットワークアイドルを待機
```

### セマンティックロケーター（refの代替）

```bash
agent-browser find role button click --name "送信"
agent-browser find text "ログイン" click
agent-browser find label "メールアドレス" fill "user@example.com"
```

## 使用例

### フォーム送信

```bash
agent-browser open https://example.com/form
agent-browser snapshot -i
# 出力例: textbox "Email" [ref=e1], textbox "Password" [ref=e2], button "Submit" [ref=e3]

agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3
agent-browser wait --load networkidle
agent-browser snapshot -i  # 結果を確認
```

### 認証状態の保存・復元

```bash
# 初回ログイン
agent-browser open https://app.example.com/login
agent-browser snapshot -i
agent-browser fill @e1 "username"
agent-browser fill @e2 "password"
agent-browser click @e3
agent-browser wait --url "**/dashboard"
agent-browser state save auth.json

# 以降のセッション: 保存した状態を読み込み
agent-browser state load auth.json
agent-browser open https://app.example.com/dashboard
```

### 複数セッション（並列ブラウザ）

```bash
agent-browser --session test1 open site-a.com
agent-browser --session test2 open site-b.com
agent-browser session list
```

### 操作を録画してデモ動画を作成

```bash
# 事前準備：操作手順を確認
agent-browser open https://example.com
agent-browser snapshot -i

# 録画開始
agent-browser record start ./demo.webm
agent-browser open https://example.com
agent-browser click @e1
agent-browser fill @e2 "テスト入力"
agent-browser click @e3
agent-browser wait --load networkidle
agent-browser record stop
# -> demo.webm が生成される
```

## デバッグ

```bash
agent-browser open example.com --headed  # ブラウザウィンドウを表示
agent-browser console                    # コンソールメッセージを表示
agent-browser errors                     # ページエラーを表示
```

## JSON出力（パース用）

```bash
agent-browser snapshot -i --json
agent-browser get text @e1 --json
```

## 注意事項

- snapshotは操作前に必ず取得し、最新のrefを使用すること
- ページ遷移やDOM変更後は再度snapshotを取得すること
- `--headed`オプションでブラウザを可視化してデバッグ可能
- セッション機能で複数ブラウザを並列操作可能
- 録画は新しいコンテキストを作成するため、録画開始後は再度ページを開くこと
