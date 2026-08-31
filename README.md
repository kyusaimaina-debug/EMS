# EMS Staff Portal v1.0

Lait Divinとは完全に別の、EMS専用Cloudflare版です。

## 入っている機能
- ログイン
- 新規従業員の初回パスワード設定
- Discordステータス一覧 / 自分のステータス変更
- 管理者: 従業員追加・編集・停止
- 管理者: 従業員ごとの権限管理
- PC / スマホ対応
- 認証・セッション・データをCloudflare Workers + D1だけで処理
- Supabase不要

## 1. D1を作る
npx.cmd wrangler d1 create ems-staff

表示された database_id を wrangler.toml の
PUT_YOUR_D1_DATABASE_ID_HERE
と置き換えてください。

## 2. DB作成
npx.cmd wrangler d1 execute ems-staff --remote --file=./d1/schema.sql

## 3. 初期管理者作成用キーを登録
npx.cmd wrangler secret put BOOTSTRAP_KEY

好きな長い秘密キーを入力してください。

## 4. デプロイ
node --check ./public/app.js
node --check ./worker/index.js
npx.cmd wrangler deploy

## 5. 最初の管理者を1回だけ作る
デプロイURLの /api/bootstrap に POST します。
PowerShell例:

$body = @{
  bootstrap_key = "手順3で設定したキー"
  employee_id = "EMS001"
  name = "管理者名"
  role = "Chief"
  password = "初期管理者パスワード"
} | ConvertTo-Json

Invoke-RestMethod `
  -Method Post `
  -Uri "https://あなたのWorkers URL/api/bootstrap" `
  -ContentType "application/json" `
  -Body $body

成功後、通常ログイン画面から EMS001 でログインできます。
bootstrapは管理者が1人作成された時点で二度と使えません。

## 従業員追加の流れ
1. 管理者ログイン
2. 「従業員管理」→「従業員追加」
3. 従業員ID・名前・役職・Discord名を登録
4. 本人は「新規従業員ログイン」から従業員IDを入力
5. 本人が自分のパスワードを設定してログイン

## Discordについて
v1.0の「Discordステータス」はEMSサイト内のスタッフステータスです。
Discord Bot/Webhookへ実際に自動同期する機能はまだ接続していません。
必要なら次版でCloudflare WorkerからDiscord側へ同期できます。
