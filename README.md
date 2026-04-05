# switchbot-battery-monitor

SwitchBot デバイスのバッテリー残量を監視し、閾値を下回ったら Discord に通知する Cloudflare Worker。

## 概要

- Cloudflare Workers の Cron トリガーで毎日定時に SwitchBot API をポーリング
- 指定したデバイスのバッテリーが閾値以下になったら Discord Webhook で通知
- `/trigger` エンドポイントで手動実行も可能

## セットアップ

### 1. 依存インストール

```bash
npm install
```

### 2. SwitchBot API キーの取得

SwitchBot アプリ → プロフィール → 一番下「開発者向けオプション」からトークンとシークレットを取得。

### 3. 監視対象デバイス ID の確認

SwitchBot アプリのデバイス詳細画面、または API のデバイス一覧から確認できる。

### 4. Discord Webhook URL の取得

通知したいチャンネルの設定 → 連携サービス → ウェブフック → 新しいウェブフック。

### 5. シークレット登録

```bash
wrangler secret put SWITCHBOT_TOKEN
wrangler secret put SWITCHBOT_SECRET
wrangler secret put DISCORD_WEBHOOK_URL
wrangler secret put MONITORED_DEVICE_IDS  # カンマ区切りで複数指定可
wrangler secret put BATTERY_THRESHOLD     # 省略時は 20 (%) がデフォルト
```

`MONITORED_DEVICE_IDS` の例：

`:` なし👇️

```
************,************
```

### 6. デプロイ

```bash
npm run deploy
```

### 7. 動作確認

```bash
# 手動トリガー
curl https://<your-worker>.workers.dev/trigger

# ログ確認
wrangler tail
```

## Cron スケジュール

`wrangler.toml` の `crons` で変更可能。デフォルトは毎日 9:00 JST。

```toml
[triggers]
crons = ["0 0 * * *"]  # 0:00 UTC = 9:00 JST
```

## 通知例

```
🔋 SwitchBot バッテリー低下 🔴
デバイス: ロックUltra 5B
残量: 15%
閾値: 20% 以下
早めに充電してください！
```

## 環境変数

| 変数名 | 説明 | デフォルト |
|---|---|---|
| `SWITCHBOT_TOKEN` | SwitchBot API トークン | 必須 |
| `SWITCHBOT_SECRET` | SwitchBot API シークレット | 必須 |
| `DISCORD_WEBHOOK_URL` | Discord Webhook URL | 必須 |
| `MONITORED_DEVICE_IDS` | 監視対象のデバイス ID（カンマ区切り） | 必須 |
| `BATTERY_THRESHOLD` | 通知する残量の閾値 (%) | `20` |
