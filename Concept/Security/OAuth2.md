## OAuth 2.0 認證流程

1. 角色說明

    + 資源擁有者（Resource Owner）: 通常是用戶，擁有受保護資源（如 Gmail 或 Google Drive 資料）。

    + 客戶端（Client）: 想要存取使用者資源的第三方應用程式（如某個應用程式或網站）。

    + 授權伺服器（Authorization Server）: Google 提供的伺服器，用於認證使用者並頒發存取權杖（Access Token）。

    + 資源伺服器（Resource Server）: 儲存受保護資源的伺服器（如 Google API）。

2. 過程

    1. 註冊應用程式：Client 在 Authorization Server 註冊你的應用程序，並取得你的客戶端 ID（client ID）和客戶端金鑰（client secret）。

    2. 用戶授權：Resource Owner 訪問你的應用程序，應用程式將重定向用戶到 Authorization Server 的認證頁面，請求用戶授權。

    3. 取得授權碼： Resource Owner 在認證頁面上登入並授權你的應用程式存取其資料。認證頁面將重定向用戶回到 Client 的應用程序(並呼叫Clien API 的 googleCallback)，並附帶一個授權碼。

    4. 取得訪問令牌： Client 的應用程式使用授權碼向 Authorization Server 授權伺服器發送請求，請求取得存取權杖。在請求中，應用程式需要提供客戶端 ID、客戶端金鑰、授權碼和重定向 URI。

    5. 驗證授權碼：Authorization Server 的授權伺服器驗證授權碼和用戶端憑證的有效性，如果驗證通過，則頒發存取權杖給Client的應用程式。

    6. 請求存取資源： 應用程式使用存取權杖向 Resource Server 的資源伺服器發送請求，請求存取受保護的資源。在請求中，應用程式需要在請求頭中提供存取權杖作為身份驗證憑證。

    7. 存取資源：Resource Server 驗證存取權杖的有效性，並根據存取權杖授權或拒絕對資源的存取。如果存取權杖有效，則資源伺服器傳回受保護資源的回應給應用程式。

3. reference 

    + https://juejin.cn/post/7010636081305485319
