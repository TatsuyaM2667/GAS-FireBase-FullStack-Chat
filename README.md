# GAS-FireBase-FullStack-Chat
---

## プロジェクト概要

このプロジェクトは、**Google Apps Script（GAS）をフロントエンド・バックエンド・フレームワーク**に、**Firebase Realtime Database** を使ってチャットメッセージを保存・同期し、**Cloudinary** を使って画像のアップロード・表示を行う、**フルスタックのチャットアプリ**です。

---

##  使用技術

| 技術             | 用途                                       |
|------------------|-------------------------------------------|
| GAS              | フロントエンド・バックエンド・フレームワーク  |
| Firebase Realtime Database | メッセージの保存・リアルタイム同期 |
| Firebase Authentication | Googleログインによるユーザー認証     |
| Cloudinary       | 画像のアップロード・ホスティング             |
| HTML/CSS(Style属性として使用)| UI構築とスタイリング                       |
| JavaScript       | クライアント側の動的処理                   |

---

## ファイルの内容

```plaintext
GAS-FireBase-FullStack-Chat/
├── Code.gs
└── index.html
```

### 🔹 Code.gs
- GASのメインスクリプト。
- `doGet()` によるWebアプリのエントリーポイント。
- Firebase Realtime Databaseへの読み書き処理。
- Cloudinary APIを呼び出して画像URLを取得・保存。

### 🔹 index.html
- チャット画面のUI。
- メッセージ表示、画像表示、入力フォーム、送信ボタンなどを含む。
- JavaScriptでFirebaseとCloudinaryを管理。
---

##  Realtime Databaseの活用

- メッセージは `push()` メソッドで Realtime Database に保存。
- `on("child_added")` イベントで新しいメッセージをリアルタイムに取得。
- データは JSON 形式で保存され、すべてのクライアントに即時反映。

---

##  Cloudinaryによる画像管理

- ユーザーが画像を選択すると、Cloudinary API経由でアップロード。
- Cloudinaryから返される画像URLを Realtime Database に保存。
- チャット画面では画像URLを使って `<img>` タグで表示。

---

