##. Cookie

 以key-value的方式儲存在瀏覽器內。

+ 儲存在Client
Server會發送一些資料到Client儲存，下次Client發送請求的時候會送給Server。
因為儲存在Client，資料可能會被用戶任意竄改，因此在使用前可以先進行檢查。

+ 不可跨域
Cookie會綁定同一個網域，無法在別的網域獲取使用。

## Session

紀錄Server與Cient狀態的一種機制，Session基於Cookie產生出Session與SessionId，Session儲存在Server，SessionId則儲存在Client。

+ 流程
1.	Server根據用戶提供的資料創建Session
2.	請求回到Client收到回傳的SessionId
3.	將返回的SessionId紀錄在Cookie與Domain
4.	當Client第二次訪問Server的時候，會將Cookie傳送到Server拿取SessionId，在查找到對應的Session驗證使用者已經登入了。

## Token
+ Acesss Token: 一種訪問時所需要的憑證。

+ 組成: UId + Time stamp + signature

+ 流程
	1.	Client要求登入
	2.	Server收到請求驗證帳號密碼
	3.	驗證成功會發送Token到Client
	4.	Client收到請求會儲存在Cookie或localstorage
	5.	Client每次請求都會帶Token
	6.	Server每次收到請求都會驗證Token

+ 特點
	+ 	Server不需儲存狀態，因此擴展性較高
	+	較高的安全性
	+ 	可跨域使用
	
+ 重點
	+ 每次攜帶Token都會放到Http的header
	+ 由於是Server無狀態的驗證方式，因次Server不需儲存Token，改以解析Token的時間換取儲存空間來減少Server的負擔，不需再頻繁的查詢資料庫。
	+ Token不受限於同源問題