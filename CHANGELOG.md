# Changelog

このプロジェクトのすべての変更を記録します。

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
