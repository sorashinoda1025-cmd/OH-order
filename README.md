# 文化祭オーダーシステム（受付・調理連動）

このWebアプリはFirebase Realtime Databaseを使い、受付画面と調理管理画面の注文をリアルタイム同期します。

## できること
- 商品を選んで注文を登録、注文番号を発行
- 調理画面で「受付済み → 調理中 → 提供完了」を更新
- 商品メニューを追加・販売停止
- 注文数、注文総額、商品別販売数を集計
- 受付用・調理用の複数端末で同じ店舗コードを使って共有

## 最初の設定（Firebase）
1. https://console.firebase.google.com/ を開き、学校用または担当者管理のFirebaseプロジェクトを作成します。
2. プロジェクト設定（歯車）→「アプリを追加」→ Webアプリを登録し、`apiKey`、`authDomain`、`projectId`、`appId` を控えます。
3. Firebase Console → Build → **Authentication** → Sign-in method で **匿名 (Anonymous)** ログインを有効にします。
4. Build → **Realtime Database** でデータベースを作成し、表示されたURL（`https://...firebasedatabase.app` または `https://...firebaseio.com`）を控えます。
5. Realtime Database → Rules に以下を設定して公開します（匿名ログイン済みの利用者だけが読み書き可能）。「公開」ボタンで保存します。

```json
{
  "rules": {
    "festivalOrders": {
      "$stallCode": {
        ".read": "auth != null",
        ".write": "auth != null"
      }
    }
  }
}
```

このルールでは、匿名認証できた利用者は店舗コードを知っていればその店舗の注文を読み書きできます。文化祭用の一時運用を想定しています。個人情報・機密情報は入力しないでください。運用終了後はルールを閉じるか、プロジェクトを停止してください。より厳密な運用では認証とアクセス制御を追加してください。

## 公開方法（例：Firebase Hosting）
このフォルダの `index.html` をHTTPSで配信してください。簡単な方法はFirebase Hostingです。

1. Node.jsをインストールし、ターミナルで次を実行します。
   ```bash
   npm install -g firebase-tools
   firebase login
   ```
2. このフォルダで実行します。
   ```bash
   firebase init hosting
   ```
   - 既存プロジェクトを選択
   - 公開ディレクトリは `public`（または現在のフォルダを指定するなら設定に合わせてファイルを置く）
   - Single-page app rewrite は `No`
   - GitHub連携は任意
3. `public` フォルダを選んだ場合は `index.html` を `public/index.html` に置き、`firebase.json` の公開ディレクトリが `public` になっていることを確認してから：
   ```bash
   firebase deploy
   ```
4. 表示された `https://....web.app` のURLを受付端末と調理端末の両方で開きます。
5. 各端末で「接続設定」を開き、同じFirebase設定値と同じ店舗コードを入力して保存します。役割は上部タブで切り替えられます。

## テスト手順
1. 2台の端末で同じURLを開いて同じ設定・店舗コードで接続。
2. 受付端末で「焼きそば」などを注文登録。
3. もう一台の「調理管理」に注文が表示されるか確認。
4. 調理画面で「調理開始」「提供完了」を押し、受付側・調理側で状態が同期されるか確認。

## 大事な注意
- `file://` でHTMLを直接開く方法は使わず、HTTPSで公開してください。Firebase SDKの読み込みや認証に影響する場合があります。
- FirebaseのAPIキーはWebアプリ設定の一部で、単独ではデータベースへのアクセス権限を与える秘密鍵ではありません。ただし、アクセス制御は必ずFirebase Security Rulesで行ってください。
- これは文化祭用の基本システムです。注文の削除・返金・支払い済み管理、印刷、在庫自動減算、オフライン時の確実な注文受付は未実装です。
- まず少人数で複数端末テストを行い、混雑時・通信切断時の運用手順も決めてください。注文確定ボタンを連打しないようスタッフに共有してください。
