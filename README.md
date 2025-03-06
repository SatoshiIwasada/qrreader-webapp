# QRコードリーダーWebアプリ

Vue 3、TypeScript、Viteを使用したQRコードリーダーWebアプリケーションです。QRコードをスキャンし、その内容を表示します。

## 機能

- カメラもしくはファイル選択を用いたQRコードのスキャン
- スキャン結果の表示
- URLの場合は直接リンクを開くオプション
- モバイルデバイス対応のレスポンシブデザイン
- JWTの自動判定と、JWT内のデコード実施(平文部分のみ)

## 技術スタック

- Vue 3
- TypeScript
- Vite
- jsQR (QRコード読み取りライブラリ)

## 開発方法

### 必要条件

- Node.js 16.0以上
- npm 7.0以上

### インストール

```bash
# リポジトリのクローン
git clone https://github.com/SatoshiIwasada/qrreader-webapp.git
cd qrreader-webapp

# 依存関係のインストール
npm install
```

### 開発サーバーの起動

```bash
npm run dev
```

### ビルド

```bash
npm run build
```

### プレビュー

```bash
npm run preview
```

## デプロイ

このプロジェクトはGitHub Pagesを使用して自動的にデプロイされます。`main`ブランチにプッシュすると、GitHub Actionsによって自動的にビルドとデプロイが行われます。

## ライセンス

MIT

## 注意事項

- カメラを利用した読み込みを行う場合はカメラへのアクセス許可が必要です
- HTTPSでの実行が必要です（ローカル開発環境を除く）
