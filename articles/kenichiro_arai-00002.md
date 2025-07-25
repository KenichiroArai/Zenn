---
title: "DevContainerでReactアプリケーションをホットリロードさせる方法2025年7月時点"
emoji: "🚀"
type: "tech" # または "idea"
topics: ["DevContainer", "React", "ホットリロード"]
published: true
---

## はじめに

DevContainerを使用してReactアプリケーションを開発している際に、ホットリロードが動作しない問題に遭遇したことはありませんか？この記事では、その問題を解決する方法を紹介します。

## 記事の流れ

この記事では、DevContainerでReactアプリケーションのホットリロード問題を解決するために、以下の流れで説明します：

1. **問題の特定**: DevContainerでホットリロードが動作しない原因を説明
2. **解決策の提示**: 環境変数設定とpackage.jsonの修正方法を紹介
3. **実践的なセットアップ**: 既存プロジェクト使用と新規作成の2つの方法を詳しく解説
4. **使用方法**: 開発サーバー起動とホットリロード確認の手順
5. **詳細設定**: DevContainer設定の詳細と重要な注意点
6. **トラブルシューティング**: よくある問題とその解決方法
7. **まとめ**: 設定のポイントと参考リソース

## 問題の概要

DevContainer内でReactアプリケーションを開発する際、以下のような問題が発生することがあります：

- ファイルを変更してもブラウザが自動更新されない
- ホットリロードが動作しない
- 手動でブラウザをリフレッシュする必要がある

## 解決方法

### 1. 環境変数の設定

まず、DevContainerの設定ファイル（`.devcontainer/devcontainer.json`）に以下の環境変数を追加します：

```json
{
  "name": "Node.js",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:1-22-bookworm",
  "forwardPorts": [3000],
  "postCreateCommand": "cd my-app && npm install",
  "remoteUser": "node",
  "mounts": [
    "source=${localWorkspaceFolderBasename}-my-app-node-modules,target=/workspaces/${localWorkspaceFolderBasename}/my-app/node_modules,type=volume"
  ]
}
```

「my-app」はReactのプロジェクトです。作成するプロジェクト合わせて設定してください。セットアップ手順の方法2を参照ください。
DevContainerのウィザードで作成する場合は、主に「mounts」を追加する必要があります。

### 2. package.jsonのスクリプト修正

`package.json`のstartスクリプトを以下のように修正します：

```json
{
  "scripts": {
    "start": "CHOKIDAR_USEPOLLING=true WATCHPACK_POLLING=true react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  }
}
```

**重要な変更点**:

- `CHOKIDAR_USEPOLLING=true`: Chokidar（ファイル監視ライブラリ）にポーリングモードを強制
- `WATCHPACK_POLLING=true`: Webpackのファイル監視にポーリングモードを強制

### 3. 変更理由

DevContainer環境では、ファイルシステムの監視が通常のinotifyベースの監視では正しく動作しない場合があります。特にWindowsホストからLinuxコンテナへのマウント時に、ファイル変更イベントが適切に伝播されないことがあります。

ポーリングモードを有効にすることで、定期的にファイルの変更をチェックし、確実にホットリロードが動作するようになります。

## セットアップ手順

### 前提条件

- Docker Desktop
- Visual Studio Code
- VS Code拡張機能: "Dev Containers"

### 手順

#### 方法1: 既存のReactプロジェクトを使用する場合

1. このリポジトリをクローン

    ```bash
    git clone https://github.com/KenichiroArai/sample-devcontainer-hotreload.git
    cd sample-devcontainer-hotreload
    ```

2. VS Codeでプロジェクトを開く

3. DevContainerで開く

   - VS Codeでプロジェクトを開いた後、右下に表示される「Reopen in Container」をクリック
   - または、コマンドパレット（Ctrl+Shift+P）で「Dev Containers: Reopen in Container」を実行

4. コンテナ内でアプリケーションを起動

    ```bash
    cd my-app
    npm start
    ```

**注意**: 依存関係のインストールは`postCreateCommand`により自動で実行されるため、手動で`npm install`を実行する必要はありません。

#### 方法2: 新しくReactプロジェクトを作成する場合

1. 空のディレクトリを作成し、VS Codeで開く

2. DevContainer設定ファイルを作成（`.devcontainer/devcontainer.json`）

3. **重要**: 最初はmounts設定をコメントアウトまたは削除する

    ```json
    // "mounts": [
    //     "source=${localWorkspaceFolderBasename}-my-app-node-modules,target=/workspaces/${localWorkspaceFolderBasename}/my-app/node_modules,type=volume"
    // ]
    ```

4. DevContainerで開く

5. コンテナ内でReactプロジェクトを作成

    ```bash
    npx create-react-app my-app
    ```

6. コンテナを終了し、`.devcontainer/devcontainer.json`のmounts設定を有効にする

7. DevContainerを再作成（Rebuild Container）

8. アプリケーションを起動

    ```bash
    cd my-app
    npm start
    ```

## 使用方法

### 開発サーバーの起動

DevContainer内で以下のコマンドを実行：

```bash
cd my-app
npm start
```

アプリケーションは `http://localhost:3000` で起動し、ホットリロード機能が有効になります。

### ホットリロードの確認

1. `my-app/src/App.js` を編集
2. 保存すると自動でブラウザが更新される
3. 変更が即座に反映されることを確認

## DevContainer設定詳細

### 基本設定

- **ベースイメージ**: `mcr.microsoft.com/devcontainers/javascript-node:1-22-bookworm`
- **ユーザー**: `node`（非rootユーザー）
- **ポート転送**: 3000番ポート

### 自動化

- **postCreateCommand**: コンテナ作成時に`cd my-app && npm install`を自動実行
- **Named volume**: node_modulesをボリュームで管理し、パフォーマンスを向上

### 重要な設定項目

#### Reactプロジェクト名の設定

このプロジェクトでは、Reactアプリケーションのディレクトリ名を`my-app`として設定しています。この名前は以下の設定で使用されています：

- **postCreateCommand**: `cd my-app && npm install`
- **mounts設定**: `target=/workspaces/${localWorkspaceFolderBasename}/my-app/node_modules`

もし異なる名前でReactプロジェクトを作成する場合は、これらの設定を適切に変更する必要があります。

#### mounts設定の注意点

```json
"mounts": [
    "source=${localWorkspaceFolderBasename}-my-app-node-modules,target=/workspaces/${localWorkspaceFolderBasename}/my-app/node_modules,type=volume"
]
```

この設定により、`node_modules`ディレクトリがNamed volumeとして管理され、コンテナ再作成時も依存関係が保持されます。ただし、**Reactプロジェクト作成前にこの設定を有効にすると、プロジェクト作成が妨げられる可能性がある**ため、上記の手順に従って設定してください。

## トラブルシューティング

### DevContainerが起動しない場合

1. Docker Desktopが起動していることを確認
2. VS CodeのDev Containers拡張機能がインストールされていることを確認
3. コンテナのビルドログを確認

### ホットリロードが動作しない場合

1. ポート3000が正しく公開されていることを確認
2. ブラウザのキャッシュをクリア
3. 開発サーバーを再起動

### 依存関係のインストールエラー

1. コンテナを再作成（Dev Containers: Rebuild Container）
2. 手動で`npm install`を実行

### Reactプロジェクトが作成できない場合

1. mounts設定が有効になっていないか確認
2. 必要に応じてmounts設定を一時的にコメントアウト
3. Reactプロジェクト作成後にmounts設定を有効化
4. コンテナを再作成

### node_modulesが正しくマウントされない場合

1. Named volumeが正しく作成されているか確認
2. コンテナを再作成（Dev Containers: Rebuild Container）
3. 必要に応じてDocker volumeを手動で削除して再作成

## まとめ

DevContainerでReactアプリケーションのホットリロードを有効にするには、以下の設定が重要です：

1. **環境変数の設定**: `WATCHPACK_POLLING`と`CHOKIDAR_USEPOLLING`を`true`に設定
2. **package.jsonの修正**: `start`スクリプトにポーリング環境変数を追加
3. **DevContainer設定**: Named volumeとpostCreateCommandの適切な設定

これらの設定により、DevContainer内でもローカル開発と同様の快適な開発体験を得ることができます。

## 参考リポジトリ

実際の動作例は以下のリポジトリで確認できます：

- [sample-devcontainer-hotreload](https://github.com/KenichiroArai/sample-devcontainer-hotreload)
