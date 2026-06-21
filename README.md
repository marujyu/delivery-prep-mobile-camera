# GitHub Pagesカメラ読み取り版 導入手順

この成果物は、スマホのカメラ読み取りをGitHub Pagesで行い、読み取り後の入力・取消処理を既存GAS Webアプリで実行します。既存の `scanOrder()` と `cancelScanOrder()` は変更していません。更新対象はP列「準備済」とQ列「準備完了日」だけで、O列「納品」は更新しません。

## 1. GAS側ファイルの追加・更新手順

1. 対象スプレッドシートから「拡張機能」→「Apps Script」を開きます。
2. 念のため現在のApps Scriptプロジェクトをバックアップします。
3. 既存のスクリプトファイルの内容を `gas/Code.gs` の内容で更新します。ファイル名が `コード.gs` でも問題ありません。
4. Apps ScriptエディタでHTMLファイルを追加し、ファイル名を拡張子なしで `ApiScanResult` とします。
5. `gas/ApiScanResult.html` の内容を追加したHTMLファイルへ貼り付けます。
6. すべて保存します。

`Code.gs` の追加点は、`doGet(e)` の `mode=apiScan` 分岐と `handleApiScan(e)` です。既存の `scanOrder()` / `cancelScanOrder()`、列番号、シート構成は維持されています。

## 2. GAS Webアプリのデプロイ更新手順

1. Apps Scriptエディタ右上の「デプロイ」→「デプロイを管理」を開きます。
2. 現在使用中のWebアプリの編集を選びます。
3. バージョンを「新バージョン」に変更します。
4. 実行ユーザーとアクセス権を既存Webアプリと同じ設定にして「デプロイ」を押します。
5. 表示された `/exec` で終わるWebアプリURLを控えます。

既存デプロイを更新すれば通常URLは変わりません。新しいデプロイを作った場合は、新しいURLをGitHub Pages側に設定してください。

### GAS側だけの先行テスト

ブラウザで次のURLを開きます。`GAS_WEB_APP_URL` は実際の `/exec` URLに置き換えてください。

入力テスト:

```text
GAS_WEB_APP_URL?mode=apiScan&action=input&orderNo=M003482986
```

P列が `TRUE`、Q列に日時が入り、O列が変化しないことを確認します。

取消テスト:

```text
GAS_WEB_APP_URL?mode=apiScan&action=cancel&orderNo=M003482986
```

P列が `FALSE`、Q列が空欄になり、O列が変化しないことを確認します。テストには実在する注文番号を使用してください。注文番号は `M` + 数字9桁との完全一致だけが処理されます。

## 3. GAS_WEB_APP_URLの差し替え手順

`github-pages/index.html` の冒頭付近にある次の1行だけを変更します。

```javascript
const GAS_WEB_APP_URL = 'ここにGASのWebアプリURLを入れる';
```

例:

```javascript
const GAS_WEB_APP_URL = 'https://script.google.com/macros/s/デプロイID/exec';
```

URL末尾に `?mode=...` は付けません。

## 4. GitHub Pages公開手順

1. GitHubで公開用リポジトリを作成します。業務情報や秘密情報は登録しないでください。
2. `github-pages` フォルダ内の `index.html` をリポジトリ直下へ配置してコミットします。
3. リポジトリの「Settings」→「Pages」を開きます。
4. 「Build and deployment」のSourceで「Deploy from a branch」を選びます。
5. 公開するブランチ（通常 `main`）とフォルダ `/ (root)` を選び、「Save」を押します。
6. 発行された `https://ユーザー名.github.io/リポジトリ名/` をスマホで開きます。

カメラ利用にはHTTPSが必要です。GitHub PagesのURLはHTTPSで配信されます。

## 5. スマホでの動作確認手順

1. スマホでGitHub Pages URLを開きます。
2. 「読取」を押し、ブラウザのカメラ利用を許可します。
3. カメラが起動することを確認します。
4. `M` + 数字9桁のQRコードを読み取ります。
5. GASの結果画面へ遷移し、注文番号、メッセージ、処理種別、結果コード、日時が表示されることを確認します。
6. 「次のQR作業へ戻る」でGitHub Pagesへ戻ることを確認します。
7. 「取消」でも同じ流れを確認します。
8. 「進捗確認」でGASの進捗画面へ移動することを確認します。

形式不正のQRはGitHub Pages上でエラーになり、GASへ送信されません。空白除去、全角英数字の半角化、大文字化以外の補正や推測は行いません。
