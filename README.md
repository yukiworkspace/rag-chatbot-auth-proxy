# 🛡️ RAG ChatBot IP制限認証システム

## 📋 概要

このシステムは、Streamlit CloudにホストされたRAG ChatBotアプリへのアクセスを、IPアドレスベースで制限するプロキシ認証機能を提供します。

## 🏗️ アーキテクチャ

```
ユーザー → AWS Amplify (IP認証) → Streamlit Cloud (RAG ChatBot)
```

### 動作フロー
1. ユーザーがAmplify URLにアクセス
2. JavaScriptでユーザーのIPアドレスを取得
3. 許可されたIP範囲と照合
4. ✅ 許可 → Streamlit Cloudアプリにリダイレクト
5. ❌ 拒否 → アクセスブロック画面を表示

## 🚀 デプロイ手順

### Step 1: 新しいAmplifyアプリ作成

1. **AWS Amplify Console**にアクセス
2. **"Create new app"** → **"Host web app"**
3. **"Deploy without Git provider"** を選択

### Step 2: ファイルアップロード

1. **index.html** をダウンロード
2. **amplify.yml** をダウンロード  
3. ZIPファイルを作成: `ip-auth-proxy.zip`
4. AmplifyにZIPファイルをアップロード

### Step 3: アプリ設定

```
App name: rag-chatbot-ip-auth
Environment: production
```

### Step 4: IP範囲設定

**index.html** の以下の部分を編集:

```javascript
const CONFIG = {
    STREAMLIT_URL: 'https://your-streamlit-app.streamlit.app', // 実際のURL
    ALLOWED_IPS: [
        '203.0.113.0/24',      // 本社オフィス
        '198.51.100.0/24',     // 支社オフィス
        '10.0.0.0/8',          // VPN範囲
        '192.168.0.0/16',      // ローカルネットワーク
        // 必要に応じて追加
    ],
    REDIRECT_DELAY: 3
};
```

## 🔧 設定項目

### IP範囲設定例

| ネットワーク種別 | CIDR表記例 | 説明 |
|---------------|------------|------|
| 単一IP | `203.0.113.50` | 特定の1つのIPアドレス |
| オフィス | `203.0.113.0/24` | 254個のIPアドレス |
| VPN | `10.0.0.0/8` | 大規模プライベートネットワーク |
| 支社 | `192.168.1.0/24` | 小規模オフィス |

### Streamlit Cloud URL

Streamlit Cloudアプリの実際のURLに変更:

```javascript
STREAMLIT_URL: 'https://rag-chatbot-ui-actual-url.streamlit.app'
```

## 📊 セキュリティ機能

### ✅ 実装済み機能

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
- **タイムアウト処理**: 長時間の処理を防止

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

### デバッグ方法

ブラウザの開発者ツールで以下を確認:

```javascript
// IPアドレス確認
console.log('検出されたIP:', userIP);

// 許可状態確認  
console.log('アクセス許可:', isAllowed);

// 設定確認
console.log('許可IP範囲:', CONFIG.ALLOWED_IPS);
```

## 🔄 更新・メンテナンス

### IP範囲の変更

1. **index.html** を編集
2. `ALLOWED_IPS` 配列を更新
3. Amplifyに再アップロード

### Streamlit URL変更

1. **index.html** を編集
2. `STREAMLIT_URL` を更新
3. Amplifyに再アップロード

## ⚠️ 制限事項

### 既知の制限

- **NATed環境**: 社内NAT環境では外部IPが表示される
- **プロキシ経由**: プロキシサーバーのIPが検出される
- **IPアドレス変動**: 動的IPの場合は定期的な更新が必要
- **JavaScript必須**: JavaScriptが無効な環境では動作しない

### 回避策

- **VPN利用**: 固定IPのVPNサーバー経由でのアクセス
- **IP範囲拡大**: より広範囲のCIDRブロック設定
- **手動更新**: IPアドレス変更時の設定更新

## 📞 トラブルシューティング

### よくある問題

| 問題 | 原因 | 解決策 |
|------|------|--------|
| IP検出失敗 | 外部サービスブロック | VPN無効化、別ネットワーク試行 |
| 認証後リダイレクト失敗 | Streamlit URL間違い | URL確認・修正 |
| 社内からアクセス拒否 | IP範囲設定ミス | CIDR表記確認 |

### ログ確認

ブラウザコンソールで詳細ログを確認:

```javascript
// エラーログ確認
console.log('エラー情報をここで確認');
```

## 🎯 運用開始チェックリスト

### デプロイ前確認

- [ ] IP範囲設定完了
- [ ] Streamlit URL設定完了  
- [ ] amplify.yml設定完了
- [ ] テスト環境での動作確認完了

### デプロイ後確認

- [ ] Amplify URLアクセス確認
- [ ] 社内IPからの認証成功確認
- [ ] 外部IPからのアクセス拒否確認
- [ ] Streamlitアプリへのリダイレクト確認
- [ ] エラーハンドリング動作確認

## 💡 カスタマイズ例

### デザインのカスタマイズ

```css
/* 企業カラーに変更 */
.logo {
    background: linear-gradient(135deg, #your-color1, #your-color2);
}

/* メッセージのカスタマイズ */
.subtitle {
    content: "あなたの会社名 セキュアシステム";
}
```

### 機能拡張

- **ログ記録**: アクセス試行の記録機能
- **通知機能**: 不正アクセス試行時のアラート
- **時間制限**: 特定時間帯のみアクセス許可
- **多要素認証**: IPアドレス + パスワード認証

これで完全なIP制限認証システムが完成します！
