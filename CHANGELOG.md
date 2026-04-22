# Changelog

このプロジェクトのすべての変更を記録します。

## [1.1.0] - 2026-04-22

### Added
- アプリ経由メンション対応（Slack Workflow Builder 等）: `message.channels` イベント受信
- `handleChannelMessage` / `isOwnBotMessage` / `extractMentionedBotUserId` 関数の新設
- 二段重複防止キャッシュ（event_id + channel+ts 補助キー）
- Script Properties に `BOT_USER_ID`, `OUR_BOT_ID`, `OUR_APP_ID` を追加
- Test.gs にアプリ発メンション関連のテスト 6 本を追加

### Docs
- README.md: Bot Token Scopes に `channels:history` 追加、Script Properties テーブル拡張、再インストール手順を明示

### Deferred（別 Issue 管理）
- OOS-1: Slack 署名検証（HMAC）の追加 — `message.channels` 購読で攻撃面積拡大のため要対応
- OOS-2: 既存の `X-Slack-Retry-Num` 検出ロジックは GAS `e.parameter` では無効のため削除 or 別手法に置換

### Specification
- 採用仕様: 案 B（Workflow Builder 等で `event.user` が入る場合のみ応答、純粋な Bot 発メッセージは無応答）

## [1.0.0] - 2026-01-08

### Added
- 初回リリース
- Slackメンション（@oyasumi_inquiry）による有給残日数照会機能
- ユーザー名（A列）付きの返信メッセージ
- 重複リアクション防止機能（CacheService + Slackリトライ検出）
- スレッド返信機能
- 以下のメッセージパターン:
  - ユーザー発見時: `○○さんの有給休暇残日数は○○日です！`
  - ユーザー未登録時: `あなたは有給休暇管理の対象外です。有給休暇管理の対象である場合は設定が完了していないので管理者へお問い合わせください。`
  - システムエラー時: `システムエラーが発生しました。しばらくしてから再度お試しください。`

### Technical Details
- Google Apps Script (GAS) Webアプリとして実装
- Slack Event API (app_mention) を使用
- Google Sheets API でスプレッドシートからデータ取得
- Script Properties で設定管理
