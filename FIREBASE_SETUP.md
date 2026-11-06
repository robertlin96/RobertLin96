# Firebase Google 登入與雲端同步設定

目前 index.html 已放入 Firebase Authentication Google Popup 登入流程，以及 Firestore 的每位使用者日程／目標即時同步程式。要真正啟用，請使用自己的 Firebase 專案完成以下設定；Firebase 專案的 Web App 設定值不在目前 repo，因此頁面預設保留未設定狀態。

## 1. 建立 Firebase Web App

1. 在 Firebase Console 建立或選取專案，新增 Web App。
2. 從該 Web App 的設定複製 apiKey、authDomain、projectId、appId。
3. 編輯 index.html 內 firebaseConfig，取代 YOUR_... 佔位值。這些是 Firebase 網頁設定識別值，不是服務帳戶金鑰；絕不可放入 Admin SDK 私密金鑰或服務帳戶憑證。

## 2. 啟用 Google Authentication

Firebase Console → Authentication → Sign-in method → Google → Enable 並儲存。再到 Authentication → Settings → Authorized domains，確認包含 robertlin96.github.io；若從自訂網域使用，也要加入該網域。Firebase 的授權網域設定無法由此靜態網站自動完成。

## 3. 建立 Firestore 與安全規則

在 Firebase Console 建立 Firestore Database，接著到 Rules 貼上以下規則並發布。這些規則讓已登入使用者只能讀寫自己 UID 底下的文件；未登入者不能存取雲端日程。

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

不要使用允許所有人讀寫的測試規則。

## 4. 部署及測試

提交並推送更新後，開啟 GitHub Pages，按「使用 Google 登入」。首次登入時，若該 Google 帳戶的 Firestore 使用者目錄還是空的，頁面會把這個瀏覽器目前的本機日程和目標遷移至該帳戶。之後新增、勾選或刪除內容會同步到登入帳戶。登出後雲端資料仍保留在該帳戶，訪客模式則繼續使用本機資料。

> Firebase 設定值會隨靜態前端公開；安全邊界是 Authentication 與 Firestore Rules。不要把 Service Account JSON、Admin SDK 私鑰、OAuth client secret 或其他伺服器憑證放進前端或 GitHub。
