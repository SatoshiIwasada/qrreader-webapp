# QRコードリーダー

ブラウザで動作するシンプルなQRコードリーダーアプリケーションです。カメラを使用したリアルタイムスキャンと画像ファイルからのQRコード読み取りに対応しています。

## 機能

- カメラを使用したQRコードのリアルタイムスキャン
- 画像ファイルからのQRコード読み取り
- ドラッグ＆ドロップによる画像ファイルの読み込み
- JWTトークンの自動検出と解析
- レスポンシブデザイン（モバイル対応）
- ダークモード対応

## デモ

[GitHub Pages](https://satoshiiwasada.github.io/qrreader-webapp/)で動作デモを確認できます。

## 開発環境のセットアップ

```bash
# リポジトリのクローン
git clone https://github.com/SatoshiIwasada/qrreader-webapp.git
cd qrreader-webapp

# 依存関係のインストール
npm install

# 開発サーバーの起動
npm run dev
```

## ビルド

```bash
# 本番用にビルド
npm run build

# ビルド結果のプレビュー
npm run preview
```

## デプロイ

このプロジェクトはGitHub Actionsを使用して、GitHub Pagesに自動的にデプロイされるように設定されています。

1. `main`ブランチにコードをプッシュすると、GitHub Actionsが自動的に実行されます
2. ビルドが成功すると、`gh-pages`ブランチに生成されたファイルがデプロイされます
3. GitHub Pagesの設定で、`gh-pages`ブランチが公開ソースとして設定されていることを確認してください

手動でデプロイする場合は、以下のコマンドを実行します：

```bash
# ビルド
npm run build

# gh-pagesブランチにデプロイ（gh-pagesパッケージが必要）
npm run deploy
```

## 技術スタック

- [Vue 3](https://v3.vuejs.org/)
- [TypeScript](https://www.typescriptlang.org/)
- [Vite](https://vitejs.dev/)
- [jsQR](https://github.com/cozmo/jsQR) - QRコード検出ライブラリ

## ライセンス

[MIT](LICENSE)

## 注意事項

- カメラを利用した読み込みを行う場合はカメラへのアクセス許可が必要です
- HTTPSでの実行が必要です（ローカル開発環境を除く）
