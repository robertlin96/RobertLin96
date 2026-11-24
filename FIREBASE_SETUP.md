# Firebase Google 登入與雲端同步設定

目前 index.html 已設定 Firebase 專案 `project-2675303080804209126` 的 Daymark Web App，並包含 Firebase Authentication Google Popup 登入流程，以及 Firestore 的每位使用者日程／目標即時同步程式。已在 Firebase Console 啟用 Google 登入，並建立位於台灣 asia-east1、以正式版安全模式啟動的 Firestore 資料庫；安全規則只允許登入者讀寫自己的 UID 路徑。此 Firebase 專案目前為 Spark 方案，避免啟用需升級付費方案的服務。

## 1. Firebase Web App（已建立）

已在 Firebase Console 為 Daymark 註冊 Web App。
Firebase Web SDK 設定已填入 index.html。
前端 Firebase 設定是公開識別值，不是服務帳戶金鑰；絕不可放入 Admin SDK 私密金鑰或服務帳戶憑證。

## 2. Google Authentication（已啟用）

Firebase Console 已顯示 Google 提供者「已啟用」。目前授權網域清單仍需加入 GitHub Pages 使用的網域 `robertlin96.github.io`；若使用自訂網域，也需加入該網域。

## 3. Firestore 與安全規則（已設定）

資料庫使用 Standard 正式版模式，位置為 asia-east1 (Taiwan)。以下規則已發布；已登入使用者只能讀寫自己 UID 底下的文件，未登入者不能存取雲端日程。

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

程式推送後，開啟 GitHub Pages，按「使用 Google 登入」。首次登入時，若該 Google 帳戶的 Firestore 使用者目錄還是空的，頁面會把這個瀏覽器目前的本機日程和目標遷移至該帳戶。之後新增、勾選或刪除內容會同步到登入帳戶。登出後雲端資料仍保留在該帳戶，訪客模式則繼續使用本機資料。

> Firebase 設定值會隨靜態前端公開；安全邊界是 Authentication 與 Firestore Rules。不要把 Service Account JSON、Admin SDK 私鑰、OAuth client secret 或其他伺服器憑證放進前端或 GitHub。


## LINE 登入（尚未啟用；需付費後端）

目前網站使用 Firebase Authentication 與 Firestore。若需安全接入 LINE Login 並同步至同一 Firebase 帳戶，需先設定 LINE Developers Login channel、HTTPS callback URL，並部署驗證 LINE 回傳資料及簽發 Firebase custom token 的可信任後端（例如 Cloud Functions）。不可將 LINE channel secret、服務帳戶私鑰或 token 簽發邏輯放在 index.html。完成後再將 LINE identity 安全連結到 Firebase 使用者，避免同一人因 Google／LINE 使用不同 UID 而資料分離。


## 費用限制

本設定保留 Firebase Spark 免費方案，沒有綁定帳單。LINE Login 要安全接入 Firebase，需可信任後端驗證 LINE 回應並簽發 Firebase custom token；Cloud Functions 部署需要升級計費方案。因此依照「不要花錢」的要求，沒有升級 Blaze、建立帳單帳戶或部署付費後端，LINE 登入按鈕保持停用。Firestore Spark 免費配額有限，超出配額時服務可能受限；請勿啟用需計費的功能。
