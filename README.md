# 有給残日数問い合わせ Slackbot (oyasumi_inquiry)

Slackで `@oyasumi_inquiry` をメンションすると、呼び出したユーザーの有給残日数をスレッドに返信するbotです。

## システム構成

```mermaid
sequenceDiagram
    participant U as User
    participant S as Slack
    participant G as GAS Web App
    participant SS as Google Sheets

    U->>S: @oyasumi_inquiry メンション
    S->>G: Event API (app_mention)
    G->>SS: C列検索（User ID）
    SS-->>G: Y列の値（残日数）
    G->>S: chat.postMessage（スレッド返信）
    S-->>U: "有給休暇の残日数は**日となります！"
```

## リソース情報

| リソース | ID/URL |
|---------|--------|
| Apps Script | `1wz0kPm561eJnCoNDFRayKhtYOXMwJM7cVQSNOdtc2vX53R1O-GBXo_07` |
| 参照スプレッドシート | `1HpeLLzMprhRCKX1Xc_9FjjAOEODEYl4-ISSQh3pHGWs` |
| 参照シート名 | `有給管理表` |

## セットアップ手順

### 1. Slack Appの作成

1. [Slack API](https://api.slack.com/apps) にアクセス
2. 「Create New App」→「From scratch」を選択
3. App名: `oyasumi_inquiry`、Workspaceを選択して作成

### 2. Bot Token Scopesの設定

**OAuth & Permissions** で以下のスコープを追加:

| スコープ | 説明 |
|---------|------|
| `app_mentions:read` | メンション検知 |
| `chat:write` | メッセージ投稿 |

### 3. GAS Webアプリのデプロイ

1. [Apps Script Editor](https://script.google.com/d/1wz0kPm561eJnCoNDFRayKhtYOXMwJM7cVQSNOdtc2vX53R1O-GBXo_07/edit) を開く
2. 右上の「デプロイ」→「新しいデプロイ」をクリック
3. 「種類を選択」→ ⚙️ アイコン →「ウェブアプリ」を選択
4. 以下を設定:
   - 説明: `Initial deployment`
   - 次のユーザーとして実行: `自分`
   - アクセスできるユーザー: `全員`
5. 「デプロイ」をクリック
6. **表示されるWebアプリURLをコピー**（後で使用）

### 4. Script Propertiesの設定

1. Apps Script Editorで「プロジェクトの設定」（⚙️アイコン）を開く
2. 「スクリプト プロパティ」セクションで以下を追加:

| プロパティ名 | 値 |
|-------------|-----|
| `SLACK_BOT_TOKEN` | Slack AppのBot User OAuth Token（`xoxb-...`） |
| `SLACK_SIGNING_SECRET` | Slack AppのSigning Secret |
| `SPREADSHEET_ID` | `1HpeLLzMprhRCKX1Xc_9FjjAOEODEYl4-ISSQh3pHGWs` |
| `SHEET_NAME` | `有給管理表` |

**Bot Token の取得方法:**
- Slack App管理画面 → **OAuth & Permissions** → **Bot User OAuth Token**

**Signing Secret の取得方法:**
- Slack App管理画面 → **Basic Information** → **App Credentials** → **Signing Secret**

### 5. Slack Event Subscriptionsの設定

1. Slack App管理画面 → **Event Subscriptions** を開く
2. 「Enable Events」を **On** に切り替え
3. **Request URL** に手順3でコピーしたWebアプリURLを貼り付け
   - 検証が成功すると「Verified ✓」と表示される
4. **Subscribe to bot events** で「Add Bot User Event」をクリック
5. `app_mention` を追加
6. 「Save Changes」をクリック

### 6. Slack Appのインストール

1. **OAuth & Permissions** → **Install to Workspace** をクリック
2. 権限を確認して「許可する」

### 7. 動作確認

1. Slack Appを使用するチャンネルに `/invite @oyasumi_inquiry` で招待
2. チャンネルで `@oyasumi_inquiry` をメンション
3. スレッドに有給残日数が返信されることを確認

## データ構造

### 参照スプレッドシート（有給管理表シート）

| 列 | 内容 | 例 |
|----|------|-----|
| C列 | Slack User ID | `U02JLA69B1N` |
| Y列 | Bot照会用（有給残日数） | `49.0` |

## テスト関数

Apps Script Editorから以下の関数を実行してテスト可能:

- `testConfig()` - 設定確認
- `testFindPaidLeaveBalance()` - スプレッドシート検索テスト
- `testFullFlow()` - 全体フローテスト（Slack投稿なし）

## トラブルシューティング

### 「ユーザー情報が見つかりませんでした」と表示される

- スプレッドシートのC列にSlack User IDが正しく登録されているか確認
- Slack User IDは `U` で始まる英数字（例: `U02JLA69B1N`）

### 返信が来ない

- Event SubscriptionsのRequest URLが正しく設定されているか確認
- Apps Script Editorの「実行数」で実行ログを確認
- Bot TokenとSigning Secretが正しく設定されているか確認

### 「Verified」にならない

- WebアプリのURLが正しいか確認（`/exec`で終わる）
- デプロイが完了しているか確認

## 更新手順

コードを修正した場合:

1. Apps Script Editorでコードを編集
2. 「デプロイ」→「デプロイを管理」→「新しいバージョン」
3. 「デプロイ」をクリック

または、MCPツールを使用:
```
smart_update_webapp(script_id="1wz0kPm561eJnCoNDFRayKhtYOXMwJM7cVQSNOdtc2vX53R1O-GBXo_07")
```
