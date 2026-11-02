# Driveレコード棚 セットアップ手順

Google Driveに置いた音楽を、アルバム（フォルダ）ごとに聴くためのPWAです。
最初に一度だけ、次の3つを行います。

1. GitHub Pagesにこのフォルダを公開する（アプリのURLを決める）
2. Google CloudでOAuthクライアントIDを発行する
3. iPhoneのSafariで開き、ホーム画面に追加する

---

## 0. Driveの準備

```
マイドライブ
└─ Music            ← アプリで「音楽フォルダ」として選ぶ
   ├─ アーティスト名 - アルバム名   ← 1フォルダ = 1アルバム
   │   ├─ 01 曲名.mp3
   │   ├─ 02 曲名.flac
   │   └─ cover.jpg       ← あればジャケットとして表示
   └─ 別のアルバム
       ├─ Disc 1/         ← 1段下のフォルダはディスクとして結合
       └─ Disc 2/
```

- フォルダ名を「アーティスト名 - アルバム名」（半角スペース・ハイフン・半角スペース）にすると、アーティスト名とアルバム名を分けて表示します。
- 曲順はファイル名順です。先頭の番号（`01 ` `01.` `01-`）は表示から外します。
- ジャケットは `cover` / `folder` / `front` / `jacket` で始まる画像を優先し、なければフォルダ内の最初の画像を使います。

## 1. GitHub Pagesで公開する

1. GitHubで新しいリポジトリを作る（例：`drive-music`）。**Public**にします（無料プランでPagesを使うため）。
   公開されるのはアプリのプログラムだけで、曲やGoogleのログイン情報は含まれません。
2. このフォルダの中身（`index.html`、`manifest.webmanifest`、`sw.js`、`icons/`）をそのままアップロードする。
   GitHubの画面で「Add file」→「Upload files」にドラッグすれば十分です。
3. リポジトリの「Settings」→「Pages」で、Sourceを「Deploy from a branch」、Branchを `main` と `/ (root)` にして保存する。
4. 1〜2分後に `https://<GitHubのユーザー名>.github.io/drive-music/` で開けるようになります。これがアプリのURLです。

参考：https://docs.github.com/ja/pages/getting-started-with-github-pages/creating-a-github-pages-site

## 2. Google CloudでクライアントIDを作る

Google Cloud Console（https://console.cloud.google.com/）で操作します。画面の名前は時期によって少し変わります。

1. 新しいプロジェクトを作る（例：`drive-music`）。
2. 「APIとサービス」→「ライブラリ」で **Google Drive API** を検索して「有効にする」。
3. 「Google Auth Platform」（旧「OAuth同意画面」）を開いて設定する。
   - アプリ名：`Driveレコード棚` など。サポートメールは自分のアドレス。
   - 対象（ユーザーの種類）：**外部**
   - **テストユーザー**に、自分のGoogleアカウント（音楽を置いたDriveのアカウント）を追加する。
   - 公開ステータスは「テスト」のままで構いません。
4. 「データアクセス」（スコープ）で `.../auth/drive.readonly` を追加する。
5. 「クライアント」→「クライアントを作成」で、種類を **ウェブ アプリケーション** にする。
   - 承認済みのJavaScript生成元：`https://<GitHubのユーザー名>.github.io`
   - 承認済みのリダイレクトURI：`https://<GitHubのユーザー名>.github.io/drive-music/`
     （アプリの最初の画面に、登録すべきURLがそのまま表示されます。末尾の `/` まで一致させてください）
6. 作成された **クライアントID**（`〜.apps.googleusercontent.com`）をコピーする。
   クライアントシークレットは使いません。

参考：https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow

## 3. iPhoneで使い始める

1. SafariでアプリのURLを開く。
2. 共有ボタン →「ホーム画面に追加」。以後はホーム画面のアイコンから開きます。
3. クライアントIDを貼り付けて保存 →「Googleでログイン」。
   - 「このアプリはGoogleで確認されていません」と出たら、「詳細」→「（安全ではないページに）移動」で進みます。自分で作った個人用アプリなので出る表示です。
4. 音楽フォルダ（例：`Music`）を選ぶ。

## 使い方のメモ

- **端末に保存**：アルバム画面のボタンで、アルバム全曲をiPhoneに保存します。保存済みの曲は通信なし・ログインなしで再生できます。曲名の右の点が保存済みの印です。
- 再生を始めると、アルバムの残りの曲も自動で順に保存します。
- **ログインの有効期限**は約1時間です。切れると上部に「再接続」が出ます。保存済みの曲はそのまま聴けます。
- 曲を追加したら、ライブラリ右上の更新ボタンを押してください。
- ロック画面とコントロールセンターから、再生・一時停止・曲送りができます。
- 保存した曲が増えたら「設定」→「保存データを削除」で空けられます。

## 対応形式

mp3 / m4a（AAC・ALAC）/ wav / aiff / flac。
FLACはiOS 11以降のSafariで再生できます（https://caniuse.com/flac）。
ogg / opus はiPhoneでは再生できないことがあります。

## うまくいかないとき

| 症状 | 確認すること |
|---|---|
| ログイン後に `redirect_uri_mismatch` | リダイレクトURIが1文字違わず登録されているか（末尾の `/`、`https`） |
| `access_denied` | テストユーザーに自分のアカウントを追加したか |
| フォルダが出ない | Drive APIを有効にしたか、`drive.readonly` を追加したか |
| 画面をロックすると止まる | iOSの仕様でホーム画面アプリの裏再生が不安定な場合があります。Safariのタブで開いて試してください |
