# AWS Signature Version 4

## 流程

+ 客戶端生成請求： 客戶端首先準備請求，這可以是任意的 HTTP 請求，如 GET、POST 等。

+ 生成 Canonical Request，Canonical Request 應包含:

    + HTTP 請求方法（如 GET、POST 等）

    + URI 路徑
    
    + 查詢字符串（按字典順序排列）
    
    + 以字典順序排列的 HTTP 請求頭
   
    + 需要簽名的請求頭列表（SignedHeaders）
    
    + 請求體的 SHA256 哈希值

+ 生成 stringToSignin

    客戶端將 Canonical Request 的 SHA256 哈希值與請求的元數據（如時間戳、日期、服務名稱等）組合，生成一個 “String to Sign”

+ 生成簽名密鑰： 客戶端使用自己的 Secret Access Key 和當前請求日期生成一個簽名密鑰。這個過程涉及多次 HMAC-SHA256 的加密操作。

    這些步驟依次使用密鑰進行 HMAC-SHA256 加密：
    + kSecret = "AWS4" + SecretAccessKey
    + kDate = HMAC-SHA256(kSecret, Date)
    + kRegion = HMAC-SHA256(kDate, Region)
    + kService = HMAC-SHA256(kRegion, Service)
    + kSigning = HMAC-SHA256(kService, "aws4_request")

+ 用生成的簽名密鑰對 String to Sign 進行 HMAC-SHA256 簽名，得到最終的簽名。

+ Authorization 標頭，其中包含簽名、已簽名的請求頭、使用的加密算法和憑證（Access Key ID）。
```
AWS4-HMAC-SHA256 Credential=ACCESS_KEY_ID/DATE/REGION/SERVICE/aws4_request, SignedHeader
```

+ 請求發送至伺服器： 客戶端將已簽名的請求發送到 AWS 服務。請求中通常包括：Authorization, X-Amz-Date, X-Amz-Content-Sha256


## reference

1. https://docs.aws.amazon.com/IAM/latest/UserGuide/create-signed-request.html

2. http://aws-signature.com.s3-website-us-west-2.amazonaws.com/
