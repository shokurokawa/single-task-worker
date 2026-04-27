# Single Task Worker

今やることひとつだけに集中するための、自分専用タスク管理アプリ。

- 1番目のタスク = 白文字ボールドで主役
- 2,3番目 = グレーで控えめ
- 4番目以降 = 濃グレーで沈ませる
- 完了はワンクリックでアーカイブ、ワンクリックで復元
- ラップトップ Chrome では Floating（Document Picture-in-Picture）ウィンドウで別タブにいる間も今のタスクを表示
- iPhone Safari と同期（Firebase Firestore）

## ファイル構成

```
app/
├── index.html             アプリ本体（UI + Firebase + PiP）
├── manifest.webmanifest   PWA メタ
└── README.md              このファイル
```

`index.html` 内の `firebaseConfig` を書き換えるだけで動きます。

---

## セットアップ手順

### 1. Firebase プロジェクトを作る

1. [Firebase Console](https://console.firebase.google.com) にアクセス（Google アカウントでログイン）
2. **「プロジェクトを追加」** → 名前 `single-task-worker` → Google アナリティクスは **無効** で作成
3. 左メニュー **「Authentication」** → 「始める」 → Sign-in method タブ → **Google** を有効化
   - サポートメールに自分のメールアドレスを設定
4. 左メニュー **「Firestore Database」** → 「データベースを作成」
   - **本番環境モード** を選択
   - リージョン `asia-northeast1`（東京）を選択
5. Firestore の **「ルール」** タブを開き、以下に置き換えて「公開」

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{uid}/tasks/{taskId} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```

6. プロジェクト設定（左上の歯車アイコン）→ 「全般」タブ → **「マイアプリ」** で `</>`（Web）アイコンをクリック
   - アプリのニックネーム `single-task-worker-web` などを入力 → 「アプリを登録」
   - 表示される `firebaseConfig` の中身（apiKey から appId まで）をコピー

### 2. index.html に Firebase 設定を貼る

`index.html` を開いて、以下のブロックを Firebase Console からコピーした値で書き換えます。

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

> Firebase の Web API キーは公開しても問題ありません（識別子であり、認可は Firestore のルール側で守ります）。次の手順の Authorized domains 設定が重要。

### 3. GitHub リポジトリを作って push

1. GitHub で新規 Public リポジトリ作成（例: `single-task-worker`）
2. ローカルで `app/` ディレクトリに移動して以下を実行（実行する際は実際のコマンドを別途案内します）

   ```bash
   cd "/Users/shokurokawa/Library/CloudStorage/GoogleDrive-sho@shokurokawa.net/マイドライブ/3_Apps/260427_Single_Task_Worker/app"
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<USERNAME>/single-task-worker.git
   git push -u origin main
   ```

### 4. GitHub Pages を有効化

1. リポジトリの **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: **main** / `/ (root)` → **Save**
4. 数分待つと `https://<USERNAME>.github.io/single-task-worker/` で公開される

### 5. Authorized domains を Firebase に追加

1. Firebase Console → **Authentication** → **Settings** タブ → **承認済みドメイン**
2. **「ドメインを追加」** → `<USERNAME>.github.io` を追加

これで Google サインインが Pages のドメインで動くようになります。

### 6. 動作確認

- ラップトップ Chrome で `https://<USERNAME>.github.io/single-task-worker/` を開く
- 「Google でサインイン」 → 自分のアカウント選択
- タスクをいくつか追加（緊急/高/低 を混ぜる）→ 並び順を確認
- 「Floating」ボタンを押す → 小さい独立ウィンドウが開く
- 別タブに切り替える → Floating ウィンドウが最前面に残る

### 7. iPhone でホーム画面に追加

1. iPhone Safari で同じ URL を開く
2. Google サインイン（リダイレクト方式で動く）
3. 共有メニュー → **「ホーム画面に追加」** → アイコン「集中」で追加される

---

## 使い方

| 操作 | 場所 |
|---|---|
| タスク追加 | 上部の入力欄 + 優先順位セレクタ + Add |
| 完了（アーカイブ） | 各タスク右の ✓ ボタン |
| アーカイブ閲覧 | ヘッダの **Archive** ボタン |
| 復元 | Archive モーダル内の **Restore** ボタン |
| Floating 起動 | ヘッダの **Floating** ボタン（Chrome デスクトップのみ） |
| サインアウト | ヘッダの **Sign out** |

初回サインイン後、Floating の使用を促すバナーが一度だけ表示されます。「試す」を選ぶと次回以降は自動で開きます（要：最初のクリック操作）。

---

## トラブルシューティング

- **「サインインに失敗しました」** → Authorized domains に GitHub Pages のドメインを追加したか確認
- **Floating ボタンが押せない** → Chrome 最新版で開いているか確認（Safari/Firefox は非対応）
- **iPhone でサインインしても戻ってこない** → Authorized domains の設定漏れ、もしくは iCloud Private Relay などをオフに
- **タスクが同期しない** → 一度サインアウトして再サインイン、ネット接続確認、Firestore のルール公開済みか確認

---

## 技術メモ

- Firebase JS SDK v10（CDN、ビルド工程なし）
- Firestore のオフライン永続化（`persistentLocalCache` + `persistentMultipleTabManager`）有効
- Google サインイン: デスクトップは popup、iOS/PWA standalone は redirect
- favicon は SVG を data URL で埋め込み（漢字「集中」白文字）
- 階層表示は CSS の `:nth-child` セレクタで実装
