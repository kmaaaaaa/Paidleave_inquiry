# 有給残日数問い合わせ Slackbot (oyasumi_inquiry)

Slackで `@oyasumi_inquiry` をメンションすると、呼び出したユーザーの有給残日数をスレッドに返信するbotです。

## 機能

- Slackメンションによる有給残日数の照会
- ユーザー名付きの返信メッセージ
- 重複リアクション防止（Slackリトライ対策）
- スレッド返信による通知
- Workflow Builder 等アプリ経由のメンションにも対応（案 B: `event.user` が存在する場合のみ応答）

## システム構成

```mermaid
sequenceDiagram
    participant U as User
    participant S as Slack
    participant G as GAS Web App
    participant SS as Google Sheets

    U->>S: @oyasumi_inquiry メンション
    S->>G: Event API (app_mention / message.channels)
    G->>G: 重複チェック (CacheService)
    G->>SS: C列検索（User ID）
    SS-->>G: A列（氏名）+ Y列（残日数）
    G->>S: chat.postMessage（スレッド返信）
    S-->>U: "○○さんの有給休暇残日数は○○日です！"
```

## リソース情報

| リソース | ID/URL |
|---------|--------|
| Apps Script | `1wz0kPm561eJnCoNDFRayKhtYOXMwJM7cVQSNOdtc2vX53R1O-GBXo_07` |
| Apps Script Editor | [開く](https://script.google.com/d/1wz0kPm561eJnCoNDFRayKhtYOXMwJM7cVQSNOdtc2vX53R1O-GBXo_07/edit) |
| 参照スプレッドシート | `1HpeLLzMprhRCKX1Xc_9FjjAOEODEYl4-ISSQh3pHGWs` |
| 参照シート名 | `有給管理表` |

## ファイル構成（Apps Script）

| ファイル | 説明 |
|----------|------|
| `Code.gs` | メインエントリポイント（doPost/doGet）、重複防止ロジック |
| `SlackHandler.gs` | Slack処理（handleAppMention/postSlackMessage） |
| `SheetHandler.gs` | スプレッドシート検索（findPaidLeaveBalance） |
| `Config.gs` | 設定管理（getConfig/setConfig） |
| `Test.gs` | テスト関数 |

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
| `channels:history` | パブリックチャンネルのメッセージ受信（アプリ経由メンション対応） |

### 3. GAS Webアプリのデプロイ

1. [Apps Script Editor](https://script.google.com/d/1wz0kPm561eJnCoNDFRayKhtYOXMwJM7cVQSNOdtc2vX53R1O-GBXo_07/edit) を開く
2. 右上の「デプロイ」→「新しいデプロイ」をクリック
3. 「種類を選択」→ ⚙ アイコン →「ウェブアプリ」を選択
4. 以下を設定:
   - 説明: `Initial deployment`
   - 次のユーザーとして実行: `自分`
   - アクセスできるユーザー: `全員`
5. 「デプロイ」をクリック
6. **表示されるWebアプリURLをコピー**（後で使用）

### 4. Script Propertiesの設定

1. Apps Script Editorで「プロジェクトの設定」（⚙アイコン）を開く
2. 「スクリプト プロパティ」セクションで以下を追加:

| プロパティ名 | 値 |
|-------------|-----|
| `SLACK_BOT_TOKEN` | Slack AppのBot User OAuth Token（`xoxb-...`） |
| `SLACK_SIGNING_SECRET` | Slack AppのSigning Secret |
| `SPREADSHEET_ID` | `1HpeLLzMprhRCKX1Xc_9FjjAOEODEYl4-ISSQh3pHGWs` |
| `SHEET_NAME` | `有給管理表` |
| `BOT_USER_ID` | Bot の User ID（`U...`）— `auth.test` API の `user_id` または `@oyasumi_inquiry` のMember ID |
| `OUR_BOT_ID` | Bot ID（`B...`）— `auth.test` API の `bot_id` |
| `OUR_APP_ID` | App ID（`A...`）— Slack App管理画面「Basic Information」→ App Credentials |

**Bot Token の取得方法:**
- Slack App管理画面 → **OAuth & Permissions** → **Bot User OAuth Token**

**Signing Secret の取得方法:**
- Slack App管理画面 → **Basic Information** → **App Credentials** → **Signing Secret**

### 5. Slack Event Subscriptionsの設定

1. Slack App管理画面 → **Event Subscriptions** を開く
2. 「Enable Events」を **On** に切り替え
3. **Request URL** に手順3でコピーしたWebアプリURLを貼り付け
   - 検証が成功すると「Verified」と表示される
4. **Subscribe to bot events** で「Add Bot User Event」をクリック
5. `app_mention` を追加
6. 同様に `message.channels` を追加
7. 「Save Changes」をクリック

### 6. Slack App のワークスペース再インストール

1. **OAuth & Permissions** → **Install to Workspace** をクリック
2. 権限を確認して「許可する」

**⚠️ 注意**: スコープ追加後（手順 2）は必ず再インストール（**Install to Workspace**）を再実行すること。スコープが反映されないと新機能が動作しません。

### 7. 動作確認（アプリメンション対応後の注意）

**⚠️ 重要**: 以下の 3 点が正しく設定されていることが必須です:
- `BOT_USER_ID`, `OUR_BOT_ID`, `OUR_APP_ID` が Script Properties に登録されている
- Slack App 管理画面で `message.channels` が Event Subscriptions に追加されている
- Workflow Builder を利用する場合、そのステップで人間ユーザー情報（`event.user`）が Bot に渡されている（案 B の制約）

実行手順

1. Slack Appを使用するチャンネルに `/invite @oyasumi_inquiry` で招待
2. チャンネルで `@oyasumi_inquiry` をメンション
3. スレッドに有給残日数が返信されることを確認

## データ構造

### 参照スプレッドシート（有給管理表シート）

| 列 | 内容 | 例 |
|----|------|-----|
| A列 | 氏名 | `神田 宏樹` |
| C列 | Slack User ID | `U02JLA69B1N` |
| Y列 | Bot照会用（有給残日数） | `49.0` |

## メッセージ仕様

| 条件 | メッセージ |
|------|-----------|
| ユーザー発見時 | `○○さんの有給休暇残日数は○○日です！` |
| ユーザー未登録時 | `あなたは有給休暇管理の対象外です。有給休暇管理の対象である場合は設定が完了していないので管理者へお問い合わせください。` |
| システムエラー時 | `システムエラーが発生しました。しばらくしてから再度お試しください。` |

## テスト関数

Apps Script Editorから以下の関数を実行してテスト可能:

| 関数 | 説明 |
|------|------|
| `testConfig()` | 設定確認 |
| `testFindPaidLeaveBalance()` | スプレッドシート検索テスト |
| `testDuplicatePrevention()` | 重複防止機能テスト |
| `testFullFlow()` | 全体フローテスト（Slack投稿なし） |

## トラブルシューティング

### 「あなたは有給休暇管理の対象外です」と表示される

- スプレッドシートのC列にSlack User IDが正しく登録されているか確認
- Slack User IDは `U` で始まる英数字（例: `U02JLA69B1N`）

### 返信が来ない

- Event SubscriptionsのRequest URLが正しく設定されているか確認
- Apps Script Editorの「実行数」で実行ログを確認
- Bot TokenとSigning Secretが正しく設定されているか確認

### アプリ経由メンションで返信が来ない

- `BOT_USER_ID`, `OUR_BOT_ID`, `OUR_APP_ID` が Script Properties に正しく設定されているか確認
- `BOT_USER_ID` は `auth.test` API で確認できます（`curl -X POST https://slack.com/api/auth.test -H "Authorization: Bearer xoxb-..."` を実行）
- Slack App 管理画面で `message.channels` イベントが Event Subscriptions に追加されているか確認
- スコープ追加後に「Install to Workspace」を再実行したか確認

### 「Verified」にならない

- WebアプリのURLが正しいか確認（`/exec`で終わる）
- デプロイが完了しているか確認

### 複数回返信が来る

- 通常は重複防止機能で対策済み
- 問題が続く場合はApps Scriptの実行ログを確認

## 既知の制約（Known Limitations）

本リリースでは以下の項目がスコープ外となっています。今後の改修で対応予定です：

### OOS-1: Slack 署名検証（HMAC）の未実装

- 現在、`SLACK_SIGNING_SECRET` を保持していますが、Slack からのリクエストに対する HMAC 検証処理がありません
- `message.channels` 購読により GAS エンドポイントへのアクセス量が増加するため、セキュリティ強化が今後必須です
- 別 Issue で対応予定

### OOS-2: 既存の `X-Slack-Retry-Num` 検出ロジックの実効性欠如

- `Code.gs` の既存ロジックは GAS `doPost(e)` の設計上、HTTP ヘッダに直接アクセスできないため機能していません
- 現状の重複防止は実質的に `event_id` キャッシュのみで機能しており、新たに追加された `channel + ts` 補助キャッシュで十分に担保されています
- 削除または別手法への置換は別 Issue で管理します

## 更新手順

コードを修正した場合:

1. Apps Script Editorでコードを編集
2. 「デプロイ」→「デプロイを管理」→「新しいバージョン」
3. 「デプロイ」をクリック

または、MCPツールを使用:
```
smart_update_webapp(script_id="1wz0kPm561eJnCoNDFRayKhtYOXMwJM7cVQSNOdtc2vX53R1O-GBXo_07")
```
