# 🛡️ RAG ChatBot IP制限認証システム

## 📋 概要

このシステムは、Streamlit CloudにホストされたRAG ChatBotアプリへのアクセスを、IPアドレスベースで制限するプロキシ認証機能を提供します。設定情報は環境変数で管理され、コードに機密情報が含まれません。

## 🏗️ アーキテクチャ

```
ユーザー → AWS Amplify (IP認証) → Streamlit Cloud (RAG ChatBot)
```

### 動作フロー
1. ユーザーがAmplify URLにアクセス
2. JavaScriptでユーザーのIPアドレスを取得
3. 環境変数で設定された許可IP範囲と照合
4. ✅ 許可 → Streamlit Cloudアプリにリダイレクト
5. ❌ 拒否 → アクセスブロック画面を表示

## 🚀 デプロイ手順

### Step 1: 新しいAmplifyアプリ作成

1. **AWS Amplify Console**にアクセス
2. **"Create new app"** → **"Host web app"**
3. **"GitHub"** を選択し、リポジトリを接続

### Step 2: 環境変数設定

**Amplify Console > App settings > Environment variables** で以下を設定:

```
STREAMLIT_URL = https://your-actual-streamlit-app.streamlit.app
ALLOWED_IPS = 192.168.28.0/24,203.0.113.0/24,10.0.0.0/8
REDIRECT_DELAY = 3
```

#### 設定例

| 環境変数名 | 設定値 | 説明 |
|-----------|--------|------|
| `STREAMLIT_URL` | `https://rag-chatbot-ui.streamlit.app` | Streamlit CloudアプリのURL |
| `ALLOWED_IPS` | `192.168.28.0/24,10.0.0.0/8` | 許可IP範囲（カンマ区切り） |
| `REDIRECT_DELAY` | `3` | リダイレクト待機時間（秒） |

### Step 3: アプリ設定

```
App name: rag-chatbot-ip-auth
Environment: production
Build settings: amplify.yml (自動検出)
```

### Step 4: デプロイ実行

環境変数設定後、**"Redeploy this version"** をクリックしてビルドを実行します。

## 🔧 環境変数詳細

### STREAMLIT_URL

Streamlit CloudアプリのフルURLを指定します。

```bash
# 正しい例
STREAMLIT_URL=https://rag-chatbot-ui.streamlit.app

# 間違った例
STREAMLIT_URL=rag-chatbot-ui.streamlit.app  # httpス://が必要
```

### ALLOWED_IPS

許可するIP範囲をカンマ区切りで指定します。CIDR記法と単一IPの両方をサポートします。

```bash
# 複数の範囲を指定する例
ALLOWED_IPS=192.168.28.0/24,203.0.113.0/24,10.0.0.0/8,198.51.100.50

# 単一の範囲のみ
ALLOWED_IPS=192.168.28.0/24

# 特定のIPアドレスのみ
ALLOWED_IPS=203.0.113.50,203.0.113.51,203.0.113.52
```

#### IP範囲設定例

| ネットワーク種別 | CIDR表記例 | 説明 |
|---------------|------------|------|
| 単一IP | `203.0.113.50` | 特定の1つのIPアドレス |
| 小規模オフィス | `192.168.28.0/24` | 254個のIPアドレス |
| VPN | `10.0.0.0/8` | 大規模プライベートネットワーク |
| 支社 | `203.0.113.0/24` | 中規模オフィス |

### REDIRECT_DELAY

認証成功後のリダイレクト待機時間（秒）を指定します。

```bash
# デフォルト値
REDIRECT_DELAY=3

# 即座にリダイレクト
REDIRECT_DELAY=0

# 長めの待機時間
REDIRECT_DELAY=5
```

## 📊 セキュリティ機能

### ✅ 実装済み機能

- **環境変数管理**: 機密情報のコード外管理
- **IPアドレス自動検出**: 複数のサービスを使用
- **CIDR範囲チェック**: ネットワーク範囲でのアクセス制御
- **WebRTC フォールバック**: 外部サービス失敗時の代替手段
- **自動リダイレクト**: 認証成功時の自動転送
- **アクセス拒否画面**: 分かりやすいエラーメッセージ
- **セッション管理**: ブラウザセッションでの認証状態保持

### 🛡️ セキュリティ対策

- **開発者ツール検知**: F12キー監視
- **コンソール警告**: 不正操作の抑制
- **エラーハンドリング**: 詳細なエラー情報の非表示
- **ビルド時検証**: 必須環境変数の存在チェック

## 🧪 テスト手順

### 基本テスト

1. **許可IPからのアクセス**
   - 社内ネットワークからアクセス
   - "アクセス許可" メッセージ確認
   - 自動リダイレクト確認

2. **拒否IPからのアクセス**  
   - 外部ネットワーク（モバイル等）からアクセス
   - "アクセス拒否" メッセージ確認
   - Streamlitアプリへの直接アクセス不可確認

### 環境変数テスト

ビルドログで環境変数が正しく適用されているか確認:

```bash
# Amplify Console > Build logs で確認
Environment variables check:
STREAMLIT_URL=https://your-app.streamlit.app
ALLOWED_IPS=192.168.28.0/24,10.0.0.0/8
REDIRECT_DELAY=3
✅ Environment variables substituted successfully
```

## 🔄 更新・メンテナンス

### IP範囲の変更

1. **Amplify Console** > **Environment variables**
2. `ALLOWED_IPS` を編集
3. **"Redeploy this version"** をクリック

### Streamlit URL変更

1. **Amplify Console** > **Environment variables**
2. `STREAMLIT_URL` を編集
3. **"Redeploy this version"** をクリック

### 設定変更の例

```bash
# 例: 新しいオフィスのIP範囲を追加
# 変更前
ALLOWED_IPS=192.168.28.0/24

# 変更後
ALLOWED_IPS=192.168.28.0/24,203.0.113.0/24
```

## ⚠️ 制限事項

### 既知の制限

- **NATed環境**: 社内NAT環境では外部IPが表示される
- **プロキシ経由**: プロキシサーバーのIPが検出される
- **IPアドレス変動**: 動的IPの場合は定期的な更新が必要
- **JavaScript必須**: JavaScriptが無効な環境では動作しない

### 回避策

- **VPN利用**: 固定IPのVPNサーバー経由でのアクセス
- **IP範囲拡大**: より広範囲のCIDRブロック設定
- **手動更新**: IPアドレス変更時の環境変数更新

## 📞 トラブルシューティング

### よくある問題

| 問題 | 原因 | 解決策 |
|------|------|--------|
| ビルドエラー「環境変数未設定」 | 必須環境変数の未設定 | Amplify環境変数を確認・設定 |
| IP検出失敗 | 外部サービスブロック | VPN無効化、別ネットワーク試行 |
| 認証後リダイレクト失敗 | STREAMLIT_URL間違い | 環境変数のURL確認・修正 |
| 社内からアクセス拒否 | ALLOWED_IPS設定ミス | CIDR表記と実際のIPを確認 |

### デバッグ手順

1. **ビルドログ確認**
   ```bash
   # Amplify Console > Build details > Build logs
   # 環境変数置換の成功/失敗を確認
   ```

2. **ブラウザコンソール確認**
   ```javascript
   // F12 > Console で以下を確認
   console.log('検出されたIP:', userIP);
   console.log('許可状態:', isAllowed);
   console.log('設定:', CONFIG);
   ```

3. **IP検出テスト**
   ```bash
   # 現在のIPアドレスを確認
   curl -s https://api.ipify.org
   ```

## 🎯 運用開始チェックリスト

### デプロイ前確認

- [ ] `STREAMLIT_URL` 環境変数設定完了
- [ ] `ALLOWED_IPS` 環境変数設定完了
- [ ] `REDIRECT_DELAY` 環境変数設定完了
- [ ] リポジトリのindex.html、amplify.ymlファイル配置完了

### デプロイ後確認

- [ ] Amplifyビルド成功確認
- [ ] 環境変数置換成功確認（ビルドログ）
- [ ] 社内IPからの認証成功確認
- [ ] 外部IPからのアクセス拒否確認
- [ ] Streamlitアプリへのリダイレクト確認
- [ ] エラーハンドリング動作確認

## 💡 運用のベストプラクティス

### セキュリティ

```bash
# 定期的なIPアドレス監視
# 月次でアクセスログをレビュー
# 不審なアクセス試行の検知
```

### メンテナンス

```bash
# 四半期ごとのIP範囲見直し
# Streamlit URLの有効性確認
# バックアップ設定の確認
```

### 監視

```bash
# CloudWatch Logs でアクセスパターン監視
# 異常なアクセス増加の検知
# サービス可用性の監視
```

これで完全に環境変数ベースの設定管理が実現できます！
