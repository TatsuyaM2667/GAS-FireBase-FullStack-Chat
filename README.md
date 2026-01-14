# GAS-FireBase-FullStack-Chat
---

## プロジェクト概要

このプロジェクトは、**Google Apps Script（GAS）をフロントエンド・バックエンド・フレームワーク**に、**Firebase Realtime Database** を使ってチャットメッセージを保存・同期し、**Cloudinary** を使って画像のアップロード・表示を行う、**フルスタックのチャットアプリ**。

---

##  使用したもの

| 技術             | 用途                                       |
|------------------|-------------------------------------------|
| Google App Script              | フロントエンド・バックエンド・フレームワーク  |
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
- 
以下はセキュリティーノート
 ```plaintext
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null",

    "users": {
      "$uid": {
        ".read": "auth !== null && auth.uid === $uid",
        ".write": "auth !== null && auth.uid === $uid",

        "username": { ".validate": "newData.isString() && newData.val().length > 0" },
        "username_lower": { ".validate": "newData.isString() && newData.val().length > 0" },
        "color": { ".validate": "newData.isString()" },
        "bio": { ".validate": "newData.isString()" },
        "email": { ".validate": "newData.isString() && newData.val().length > 0" },
        "photoURL": { ".validate": "newData.isString() && newData.val().contains('https://res.cloudinary.com')" },

        "location": { ".validate": "newData.isString()" },
        "phone": { ".validate": "newData.isString()" },
        "website": { ".validate": "newData.isString()" },
        "profileBgImage": { ".validate": "newData.isString() && newData.val().contains('https://res.cloudinary.com')" },

        "blockedUsers": {
          ".read": "auth.uid === $uid",
          ".write": "auth.uid === $uid",
          "$blockedUid": { ".validate": "newData.isBoolean()" }
        }
      },
      ".read": "auth != null"
    },

    "rooms": {
      "$roomId": {
        ".read": "data.child('members/' + auth.uid).val() === true && (data.child('type').val() === 'group' || (data.child('type').val() === 'dm' && !root.child('users/' + auth.uid + '/blockedUsers').hasChild($roomId.replace('dm_','').replace(auth.uid,'').replace('_','')) && !root.child('users/' + $roomId.replace('dm_','').replace(auth.uid,'').replace('_','') + '/blockedUsers').hasChild(auth.uid)))",
        ".write": "auth != null && (data.child('members/' + auth.uid).val() === true || (!data.exists() && newData.child('members/' + auth.uid).val() === true))",

        "members": {
          ".read": "data.parent().child('members/' + auth.uid).val() === true",
          ".write": "data.parent().child('members/' + auth.uid).val() === true",
          "$memberId": { ".validate": "newData.isBoolean()" }
        },

        "messages": {
          ".read": "data.parent().child('members/' + auth.uid).val() === true",

          "$messageId": {
            ".write": "data.parent().parent().child('members/' + auth.uid).val() === true && (data.parent().parent().child('type').val() === 'group' || (data.parent().parent().child('type').val() === 'dm' && !root.child('users/' + auth.uid + '/blockedUsers').hasChild($roomId.replace('dm_','').replace(auth.uid,'').replace('_','')) && !root.child('users/' + $roomId.replace('dm_','').replace(auth.uid,'').replace('_','') + '/blockedUsers').hasChild(auth.uid)))",
            ".validate": "newData.hasChildren(['username','uid','timestamp','type']) && newData.child('uid').val() === auth.uid",

            "username": { ".validate": "newData.isString()" },
            "uid": { ".validate": "newData.isString() && newData.val() === auth.uid" },
            "timestamp": { ".validate": "newData.isNumber()" },
            "message": { ".validate": "newData.isString()" },
            "type": { ".validate": "newData.isString() && (newData.val()==='text' || newData.val()==='image' || newData.val()==='video')" },
            "fileUrl": { ".validate": "(newData.parent().child('type').val()==='image' || newData.parent().child('type').val()==='video') && newData.isString() && newData.val().contains('https://res.cloudinary.com')" }
          }
        },

        "lastRead": {
          ".read": "data.parent().child('members/' + auth.uid).val() === true",
          ".write": "data.parent().child('members/' + auth.uid).val() === true",
          "$uid": { ".validate": "$uid === auth.uid" }
        },

        "deletionRequest": {
          ".read": "data.parent().child('members/' + auth.uid).val() === true",
          ".write": "data.parent().child('members/' + auth.uid).val() === true",
          ".validate": "newData.hasChildren(['requestedBy','requestedAt','votes'])",

          "votes": {
            "$voterUid": { ".validate": "newData.isBoolean() && $voterUid === auth.uid" }
          }
        },

        "callOffers": {
          "$uid": {
            ".read": "data.parent().parent().child('members').hasChild(auth.uid)",
            ".write": "data.parent().parent().child('members').hasChild(auth.uid)",
            ".validate": "newData.hasChildren(['type','sdp']) && newData.child('type').val()==='offer'"
          }
        },

        "callAnswers": {
          "$uid": {
            ".read": "data.parent().parent().child('members').hasChild(auth.uid)",
            ".write": "data.parent().parent().child('members').hasChild(auth.uid)",
            ".validate": "newData.hasChildren(['type','sdp']) && newData.child('type').val()==='answer'"
          }
        },

        "iceCandidates": {
          "$uid": {
            ".read": "data.parent().parent().child('members').hasChild(auth.uid)",
            ".write": "data.parent().parent().child('members').hasChild(auth.uid)"
          }
        },

        "type": { ".validate": "newData.isString() && (newData.val()==='dm' || newData.val()==='group')" },
        "name": { ".validate": "newData.isString()" },
        "createdAt": { ".validate": "newData.isNumber()" },
        "createdBy": { ".validate": "newData.isString()" },
        "iconType": { ".validate": "newData.isString() && (newData.val()==='initial' || newData.val()==='emoji')" },
        "iconCode": { ".validate": "newData.isString()" },
        "iconColor": { ".validate": "newData.isString()" }
      }
    }
  }
}
```


---

##  Cloudinaryによる画像管理

- ユーザーが画像を選択すると、Cloudinary API経由でアップロード。
- Cloudinaryから返される画像URLを Realtime Database に保存。
- チャット画面では画像URLを使って `<img>` タグで表示。

---
## 👨‍💻 作者
- [TatsuyaM2667](https://github.com/TatsuyaM2667)

